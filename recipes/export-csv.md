---
title: Export to CSV
excerpt: Log aircraft sightings to CSV files for analysis and record-keeping.
hidden: false
recipe:
  color: '#10B981'
  icon: 📊
difficulty: beginner
tags: [data, export, logging, csv, analytics]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Python: `pip install sseclient-py requests`
- Node.js: `npm install eventsource`
- Go: No external dependencies required

## What You'll Build

A logging system that records every aircraft sighting to a CSV file. This is useful for:

- **Historical analysis** - Track patterns over time
- **Statistics** - Count aircraft by type, altitude, etc.
- **Verification** - Prove an aircraft was overhead
- **Data science** - Feed into Jupyter notebooks or pandas

```shell Shell
pip install sseclient-py requests
python csv_logger.py
# Output: aircraft_2024-01-15.csv
```

```go Go
package main

import (
    "bufio"
    "encoding/csv"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "strings"
    "time"
)

func main() {
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    // Create dated filename
    filename := fmt.Sprintf("aircraft_%s.csv", time.Now().Format("2006-01-02"))
    file, err := os.Create(filename)
    if err != nil {
        fmt.Printf("Error creating file: %v\n", err)
        os.Exit(1)
    }
    defer file.Close()

    writer := csv.NewWriter(file)
    headers := []string{"timestamp", "hex", "flight", "type", "alt", "gs", "track", "lat", "lon", "distance", "military", "squawk"}
    writer.Write(headers)

    fmt.Printf("Logging aircraft to %s\n", filename)

    seen := make(map[string]time.Time)

    for {
        resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
        if err != nil {
            fmt.Printf("Connection failed: %v, retrying in 5s\n", err)
            time.Sleep(5 * time.Second)
            continue
        }

        scanner := bufio.NewScanner(resp.Body)
        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data map[string]interface{}
                if err := json.Unmarshal([]byte(line[5:]), &data); err != nil {
                    continue
                }
                if aircraft, ok := data["aircraft"].([]interface{}); ok {
                    for _, a := range aircraft {
                        ac := a.(map[string]interface{})
                        hex := fmt.Sprint(ac["hex"])

                        // Only log once per minute per aircraft
                        if lastSeen, exists := seen[hex]; exists && time.Since(lastSeen) < time.Minute {
                            continue
                        }
                        seen[hex] = time.Now()

                        row := []string{
                            time.Now().Format(time.RFC3339),
                            hex,
                            fmt.Sprint(ac["flight"]),
                            fmt.Sprint(ac["type"]),
                            fmt.Sprint(ac["alt"]),
                            fmt.Sprint(ac["gs"]),
                            fmt.Sprint(ac["track"]),
                            fmt.Sprint(ac["lat"]),
                            fmt.Sprint(ac["lon"]),
                            fmt.Sprint(ac["distance"]),
                            fmt.Sprint(ac["military"]),
                            fmt.Sprint(ac["squawk"]),
                        }
                        writer.Write(row)
                        writer.Flush()
                    }
                }
            }
        }
        resp.Body.Close()
    }
}
```

```python Python
import csv
import json
import os
import sys
from datetime import datetime, date
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
OUTPUT_DIR = os.getenv("OUTPUT_DIR", ".")
LOG_INTERVAL = int(os.getenv("LOG_INTERVAL", "60"))  # Seconds between logs per aircraft

FIELDS = [
    "timestamp", "hex", "flight", "type", "alt", "gs",
    "track", "lat", "lon", "distance", "military", "squawk", "category"
]


def get_daily_filename():
    """Generate dated filename for daily rotation."""
    return os.path.join(OUTPUT_DIR, f"aircraft_{date.today().isoformat()}.csv")


def main():
    seen = {}  # Track last log time per aircraft
    current_date = date.today()

    filename = get_daily_filename()
    file_exists = os.path.exists(filename)

    f = open(filename, "a", newline="")
    writer = csv.DictWriter(f, fieldnames=FIELDS)

    if not file_exists:
        writer.writeheader()
        print(f"Created new log file: {filename}")
    else:
        print(f"Appending to existing log: {filename}")

    try:
        while True:
            try:
                response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
                response.raise_for_status()
                client = sseclient.SSEClient(response)

                print(f"Connected to {SKYSPY_URL}, logging aircraft...")

                for event in client.events():
                    # Check for date rollover
                    if date.today() != current_date:
                        f.close()
                        current_date = date.today()
                        filename = get_daily_filename()
                        f = open(filename, "w", newline="")
                        writer = csv.DictWriter(f, fieldnames=FIELDS)
                        writer.writeheader()
                        print(f"Rolled over to new file: {filename}")

                    if event.event in ["aircraft_update", "aircraft_new"]:
                        try:
                            data = json.loads(event.data)
                        except json.JSONDecodeError:
                            continue

                        for aircraft in data.get("aircraft", []):
                            hex_code = aircraft.get("hex")
                            now = datetime.now()

                            # Rate limit logging per aircraft
                            if hex_code in seen:
                                if (now - seen[hex_code]).total_seconds() < LOG_INTERVAL:
                                    continue

                            seen[hex_code] = now

                            writer.writerow({
                                "timestamp": now.isoformat(),
                                "hex": hex_code,
                                "flight": aircraft.get("flight", ""),
                                "type": aircraft.get("type", ""),
                                "alt": aircraft.get("alt", ""),
                                "gs": aircraft.get("gs", ""),
                                "track": aircraft.get("track", ""),
                                "lat": aircraft.get("lat", ""),
                                "lon": aircraft.get("lon", ""),
                                "distance": aircraft.get("distance", ""),
                                "military": aircraft.get("military", False),
                                "squawk": aircraft.get("squawk", ""),
                                "category": aircraft.get("category", ""),
                            })
                            f.flush()

                            print(f"Logged: {aircraft.get('flight', hex_code)} ({aircraft.get('type', 'Unknown')})")

            except requests.RequestException as e:
                print(f"Connection error: {e}, reconnecting in 5s...")
                import time
                time.sleep(5)

    except KeyboardInterrupt:
        print("\nStopping logger...")
    finally:
        f.close()


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const fs = require('fs');
const path = require('path');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const OUTPUT_DIR = process.env.OUTPUT_DIR || '.';
const LOG_INTERVAL = parseInt(process.env.LOG_INTERVAL || '60') * 1000; // ms

const fields = ['timestamp', 'hex', 'flight', 'type', 'alt', 'gs', 'track', 'lat', 'lon', 'distance', 'military', 'squawk'];
const seen = new Map();
let currentDate = new Date().toISOString().split('T')[0];
let stream;

function getFilename() {
  return path.join(OUTPUT_DIR, `aircraft_${currentDate}.csv`);
}

function openStream() {
  const filename = getFilename();
  const exists = fs.existsSync(filename);
  stream = fs.createWriteStream(filename, { flags: 'a' });

  if (!exists) {
    stream.write(fields.join(',') + '\n');
    console.log(`Created new log file: ${filename}`);
  } else {
    console.log(`Appending to existing log: ${filename}`);
  }
}

function checkDateRollover() {
  const today = new Date().toISOString().split('T')[0];
  if (today !== currentDate) {
    stream.end();
    currentDate = today;
    openStream();
    console.log(`Rolled over to new file: ${getFilename()}`);
  }
}

openStream();

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.onerror = (err) => {
  console.error('SSE connection error, will retry...');
};

es.onopen = () => {
  console.log(`Connected to ${SKYSPY_URL}, logging aircraft...`);
};

es.addEventListener('aircraft_update', (e) => {
  checkDateRollover();

  let data;
  try {
    data = JSON.parse(e.data);
  } catch (err) {
    return;
  }

  const now = Date.now();

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;

    // Rate limit logging per aircraft
    if (seen.has(hex) && now - seen.get(hex) < LOG_INTERVAL) {
      continue;
    }
    seen.set(hex, now);

    const row = [
      new Date().toISOString(),
      aircraft.hex || '',
      aircraft.flight || '',
      aircraft.type || '',
      aircraft.alt || '',
      aircraft.gs || '',
      aircraft.track || '',
      aircraft.lat || '',
      aircraft.lon || '',
      aircraft.distance || '',
      aircraft.military || false,
      aircraft.squawk || ''
    ];
    stream.write(row.join(',') + '\n');
    console.log(`Logged: ${aircraft.flight || hex} (${aircraft.type || 'Unknown'})`);
  }
});

// Cleanup old entries from seen map every 5 minutes
setInterval(() => {
  const cutoff = Date.now() - (LOG_INTERVAL * 2);
  for (const [hex, time] of seen) {
    if (time < cutoff) seen.delete(hex);
  }
}, 300000);

process.on('SIGINT', () => {
  console.log('\nStopping logger...');
  stream.end();
  process.exit(0);
});
```

```json Response Example
{"timestamp": "2024-01-15T14:32:18Z", "hex": "A12345", "flight": "UAL123", "type": "B738", "alt": 35000, "gs": 450, "track": 270, "lat": 40.7128, "lon": -74.0060, "distance": 12.4, "military": false, "squawk": "1200"}
```

## Create CSV File & Headers

<!-- shell@1-3 -->
<!-- go@15-28 -->
<!-- python@18-32 -->
<!-- javascript@12-26 -->

Create a dated CSV file with appropriate headers. The filename includes the date for automatic daily rotation.

## Connect to SSE Stream

<!-- go@34-45 -->
<!-- python@35-44 -->
<!-- javascript@31-38 -->

Connect to SkySpy's Server-Sent Events endpoint to receive real-time aircraft updates. The connection automatically reconnects if it drops.

## Write Aircraft Data with Rate Limiting

<!-- go@47-72 -->
<!-- python@46-79 -->
<!-- javascript@40-76 -->

Log each aircraft with rate limiting to avoid duplicate entries. By default, each aircraft is logged at most once per minute to keep file sizes manageable.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `OUTPUT_DIR` | Directory for CSV files | Current directory |
| `LOG_INTERVAL` | Seconds between logs per aircraft | `60` |

### Customizing Fields

Add or remove fields from the CSV:

```python
# Add registration/owner info if available
FIELDS = [..., "registration", "owner"]

# Log only essential fields
FIELDS = ["timestamp", "hex", "flight", "alt", "lat", "lon"]
```

### Filtering What Gets Logged

Only log specific aircraft:

```python
# Only log military aircraft
if not aircraft.get("military"):
    continue

# Only log aircraft below 10,000 feet
if aircraft.get("alt", 99999) > 10000:
    continue

# Only log aircraft within 25nm
if aircraft.get("distance", 999) > 25:
    continue
```

## File Rotation

The scripts automatically rotate to a new file at midnight. Files are named:

- `aircraft_2024-01-15.csv`
- `aircraft_2024-01-16.csv`
- etc.

### Manual Rotation

Force rotation based on file size:

```python
import os

MAX_SIZE = 100 * 1024 * 1024  # 100 MB

if os.path.getsize(filename) > MAX_SIZE:
    # Create new file with sequence number
    filename = f"aircraft_{date.today().isoformat()}_{sequence}.csv"
    sequence += 1
```

## Testing & Verification

1. **Start the logger** using any of the code examples
2. **Check file creation** - A new CSV file should appear in the output directory
3. **Verify entries** - Open the CSV and verify aircraft are being logged
4. **Test rollover** - Wait for midnight or manually adjust the date check

### Sample Output

```csv
timestamp,hex,flight,type,alt,gs,track,lat,lon,distance,military,squawk
2024-01-15T14:32:18Z,A12345,UAL123,B738,35000,450,270,40.7128,-74.0060,12.4,False,1200
2024-01-15T14:33:45Z,AE1234,RCH419,C17,28000,380,180,40.8000,-73.9500,8.2,True,
```

## Analyzing the Data

### With pandas (Python)

```python
import pandas as pd

df = pd.read_csv("aircraft_2024-01-15.csv")

# Count by aircraft type
print(df.groupby("type").size().sort_values(ascending=False))

# Find all military aircraft
military = df[df["military"] == True]

# Average altitude by type
print(df.groupby("type")["alt"].mean())
```

### With csvkit (Command Line)

```shell
# View summary
csvstat aircraft_2024-01-15.csv

# Filter military
csvgrep -c military -m True aircraft_2024-01-15.csv

# Sort by altitude
csvsort -c alt -r aircraft_2024-01-15.csv | head
```

## Troubleshooting

### File Not Created

**Problem:** No CSV file appears
**Solution:**
- Check write permissions in the output directory
- Verify the script has access to create files
- Check for errors in the console output

### Missing Data

**Problem:** Some aircraft aren't being logged
**Solution:**
- The rate limiting might be filtering them out
- Reduce `LOG_INTERVAL` for more frequent logging
- Check if filters are excluding the aircraft

### Large File Sizes

**Problem:** CSV files are too large
**Solution:**
- Increase `LOG_INTERVAL` to reduce entries
- Add filters to log only specific aircraft
- Implement file size-based rotation

### CSV Corruption

**Problem:** File has malformed rows
**Solution:**
- Ensure proper flushing after each write
- Check for special characters in callsigns
- Use CSV library instead of string concatenation

## Related Recipes

- [JSON Lines Logging](/docs/jsonl-logging) - Alternative format
- [SQLite Local DB](/docs/sqlite-db) - Queryable storage
- [InfluxDB Logging](/docs/influxdb-logging) - Time-series database
- [Parquet Export](/docs/parquet-export) - Columnar format for analytics
