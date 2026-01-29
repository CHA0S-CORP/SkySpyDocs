---
title: JSON Lines Logging
excerpt: Append-only JSON logs for easy processing.
hidden: false
recipe:
  color: '#10B981'
  icon: 📝
difficulty: beginner
tags: [data, logging, json, analytics]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Python: `pip install sseclient-py requests`

## What You'll Build

An append-only log file in JSON Lines format (one JSON object per line). This format is ideal for log processing tools, streaming analytics, and data pipelines.

```shell Shell
pip install sseclient-py requests
python jsonl_logger.py
# Creates: aircraft_2024-01-15.jsonl
```

```python Python
import json
import os
import sys
import gzip
from datetime import datetime, date
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
OUTPUT_DIR = os.getenv("OUTPUT_DIR", ".")
LOG_INTERVAL = int(os.getenv("LOG_INTERVAL", "30"))
COMPRESS = os.getenv("COMPRESS", "false").lower() == "true"

seen = {}


def get_filename():
    """Generate dated filename."""
    ext = ".jsonl.gz" if COMPRESS else ".jsonl"
    return os.path.join(OUTPUT_DIR, f"aircraft_{date.today().isoformat()}{ext}")


def open_log(filename):
    """Open log file, optionally compressed."""
    if COMPRESS:
        return gzip.open(filename, "at", encoding="utf-8")
    return open(filename, "a", encoding="utf-8")


def log_aircraft(f, aircraft):
    """Log a single aircraft to JSONL file."""
    hex_code = aircraft.get("hex")
    now = datetime.now()

    # Rate limit per aircraft
    if hex_code in seen:
        if (now - seen[hex_code]).total_seconds() < LOG_INTERVAL:
            return False
    seen[hex_code] = now

    record = {
        "timestamp": now.isoformat() + "Z",
        "hex": hex_code,
        "flight": aircraft.get("flight"),
        "type": aircraft.get("type"),
        "altitude": aircraft.get("alt"),
        "speed": aircraft.get("gs"),
        "track": aircraft.get("track"),
        "lat": aircraft.get("lat"),
        "lon": aircraft.get("lon"),
        "distance": aircraft.get("distance"),
        "military": aircraft.get("military", False),
        "squawk": aircraft.get("squawk"),
        "category": aircraft.get("category"),
    }

    # Write as single line
    f.write(json.dumps(record, separators=(",", ":")) + "\n")
    f.flush()
    return True


def main():
    print(f"JSONL logger connected to {SKYSPY_URL}...")

    current_date = date.today()
    filename = get_filename()
    f = open_log(filename)
    print(f"Logging to {filename}")

    try:
        while True:
            try:
                response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
                client = sseclient.SSEClient(response)

                for event in client.events():
                    # Handle date rollover
                    if date.today() != current_date:
                        f.close()
                        current_date = date.today()
                        filename = get_filename()
                        f = open_log(filename)
                        print(f"Rolled to {filename}")

                    if event.event in ["aircraft_update", "aircraft_new"]:
                        data = json.loads(event.data)
                        for aircraft in data.get("aircraft", []):
                            if log_aircraft(f, aircraft):
                                flight = aircraft.get("flight", aircraft.get("hex"))
                                print(f"Logged: {flight}")

            except requests.RequestException as e:
                print(f"Error: {e}, reconnecting...")
                import time
                time.sleep(5)

    finally:
        f.close()


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const fs = require('fs');
const zlib = require('zlib');
const path = require('path');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const OUTPUT_DIR = process.env.OUTPUT_DIR || '.';
const LOG_INTERVAL = parseInt(process.env.LOG_INTERVAL || '30') * 1000;
const COMPRESS = process.env.COMPRESS === 'true';

const seen = new Map();
let currentDate = new Date().toISOString().split('T')[0];
let stream;

function getFilename() {
  const ext = COMPRESS ? '.jsonl.gz' : '.jsonl';
  return path.join(OUTPUT_DIR, `aircraft_${currentDate}${ext}`);
}

function openStream() {
  const filename = getFilename();
  if (COMPRESS) {
    const gzip = zlib.createGzip();
    gzip.pipe(fs.createWriteStream(filename, { flags: 'a' }));
    stream = gzip;
  } else {
    stream = fs.createWriteStream(filename, { flags: 'a' });
  }
  console.log(`Logging to ${filename}`);
}

openStream();

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  // Date rollover
  const today = new Date().toISOString().split('T')[0];
  if (today !== currentDate) {
    stream.end();
    currentDate = today;
    openStream();
  }

  const data = JSON.parse(e.data);
  const now = Date.now();

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;

    if (seen.has(hex) && now - seen.get(hex) < LOG_INTERVAL) continue;
    seen.set(hex, now);

    const record = {
      timestamp: new Date().toISOString(),
      hex,
      flight: aircraft.flight,
      type: aircraft.type,
      altitude: aircraft.alt,
      speed: aircraft.gs,
      lat: aircraft.lat,
      lon: aircraft.lon,
      distance: aircraft.distance,
      military: aircraft.military || false
    };

    stream.write(JSON.stringify(record) + '\n');
  }
});
```

```jsonl Sample Output
{"timestamp":"2024-01-15T14:32:18.123Z","hex":"A12345","flight":"UAL123","type":"B738","altitude":35000,"speed":450,"lat":40.7128,"lon":-74.006,"distance":12.4,"military":false}
{"timestamp":"2024-01-15T14:32:48.456Z","hex":"AE1234","flight":"RCH419","type":"C17","altitude":28000,"speed":380,"lat":40.75,"lon":-73.95,"distance":8.2,"military":true}
{"timestamp":"2024-01-15T14:33:18.789Z","hex":"A54321","flight":"DAL456","type":"A321","altitude":38000,"speed":485,"lat":40.8,"lon":-74.1,"distance":15.7,"military":false}
```

## Why JSON Lines?

- **Append-only**: No need to read entire file
- **Streamable**: Process line by line
- **Tool-friendly**: Works with jq, grep, awk
- **Compressible**: Gzip reduces size significantly

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `OUTPUT_DIR` | Output directory | Current directory |
| `LOG_INTERVAL` | Seconds between logs per aircraft | `30` |
| `COMPRESS` | Gzip compress files | `false` |

## Processing with jq

```shell
# Count records
wc -l aircraft_2024-01-15.jsonl

# Extract military aircraft
jq -c 'select(.military == true)' aircraft.jsonl

# Get unique aircraft types
jq -r '.type' aircraft.jsonl | sort | uniq -c | sort -rn

# Find closest approach
jq -s 'min_by(.distance)' aircraft.jsonl

# Filter by altitude
jq -c 'select(.altitude < 5000)' aircraft.jsonl

# Time range filter
jq -c 'select(.timestamp > "2024-01-15T12:00:00")' aircraft.jsonl
```

## Load into pandas

```python
import pandas as pd

# Read JSONL
df = pd.read_json("aircraft.jsonl", lines=True)

# Parse timestamps
df["timestamp"] = pd.to_datetime(df["timestamp"])

# Analysis
print(df.groupby("type").size().sort_values(ascending=False))
print(df[df["military"]]["flight"].unique())
```

## Streaming with Python

```python
# Process as stream
with open("aircraft.jsonl") as f:
    for line in f:
        record = json.loads(line)
        if record.get("military"):
            print(f"Military: {record['flight']}")
```

## Testing & Verification

1. **Start the logger**
2. **Check file creation**:
   ```shell
   ls -la *.jsonl
   ```
3. **Validate JSON**:
   ```shell
   head -1 aircraft.jsonl | jq .
   ```
4. **Test rollover** by waiting until midnight or adjusting date check

## Troubleshooting

### Invalid JSON

**Problem:** Line parsing fails
**Solution:**
- Ensure flush after each write
- Check for interrupted writes
- Validate with `jq -c '.' file.jsonl`

### Large File Sizes

**Problem:** Files growing too large
**Solution:**
- Enable compression: `COMPRESS=true`
- Increase `LOG_INTERVAL`
- Add filters for specific aircraft

## Related Recipes

- [Export to CSV](/docs/export-csv) - Traditional format
- [SQLite Local DB](/docs/sqlite-db) - Queryable storage
- [InfluxDB Logging](/docs/influxdb-logging) - Time-series
- [Parquet Export](/docs/parquet-export) - Analytics format
