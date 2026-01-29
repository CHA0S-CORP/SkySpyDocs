---
title: SQLite Local Database
excerpt: Store aircraft history in a lightweight local database.
hidden: false
recipe:
  color: '#003B57'
  icon: 🗄️
difficulty: beginner
tags: [data, database, sqlite, storage]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Python 3.7+ (sqlite3 is built-in)

## What You'll Build

A local SQLite database that stores all aircraft sightings for querying and analysis. Perfect for building historical records without external database infrastructure.

```shell Shell
pip install sseclient-py requests
python sqlite_logger.py
# Creates: skyspy.db
```

```go Go
package main

import (
    "bufio"
    "database/sql"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "strings"
    "time"

    _ "github.com/mattn/go-sqlite3"
)

func main() {
    dbPath := os.Getenv("DB_PATH")
    if dbPath == "" {
        dbPath = "skyspy.db"
    }
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    db, err := sql.Open("sqlite3", dbPath)
    if err != nil {
        fmt.Printf("Failed to open database: %v\n", err)
        os.Exit(1)
    }
    defer db.Close()

    // Create tables
    db.Exec(`CREATE TABLE IF NOT EXISTS aircraft (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
        hex TEXT NOT NULL,
        flight TEXT,
        type TEXT,
        altitude INTEGER,
        speed INTEGER,
        track REAL,
        lat REAL,
        lon REAL,
        distance REAL,
        military BOOLEAN,
        squawk TEXT
    )`)
    db.Exec(`CREATE INDEX IF NOT EXISTS idx_hex ON aircraft(hex)`)
    db.Exec(`CREATE INDEX IF NOT EXISTS idx_timestamp ON aircraft(timestamp)`)

    stmt, _ := db.Prepare(`INSERT INTO aircraft
        (hex, flight, type, altitude, speed, track, lat, lon, distance, military, squawk)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)`)
    defer stmt.Close()

    seen := make(map[string]time.Time)

    for {
        resp, _ := http.Get(skyspyURL + "/api/v1/map/sse")
        scanner := bufio.NewScanner(resp.Body)

        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data map[string]interface{}
                json.Unmarshal([]byte(line[5:]), &data)

                if aircraft, ok := data["aircraft"].([]interface{}); ok {
                    for _, a := range aircraft {
                        ac := a.(map[string]interface{})
                        hex := fmt.Sprint(ac["hex"])

                        if last, exists := seen[hex]; exists && time.Since(last) < time.Minute {
                            continue
                        }
                        seen[hex] = time.Now()

                        stmt.Exec(
                            hex,
                            ac["flight"],
                            ac["type"],
                            ac["alt"],
                            ac["gs"],
                            ac["track"],
                            ac["lat"],
                            ac["lon"],
                            ac["distance"],
                            ac["military"],
                            ac["squawk"],
                        )
                    }
                }
            }
        }
        resp.Body.Close()
    }
}
```

```python Python
import json
import os
import sqlite3
import sys
from datetime import datetime, timedelta
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
DB_PATH = os.getenv("DB_PATH", "skyspy.db")
LOG_INTERVAL = int(os.getenv("LOG_INTERVAL", "60"))  # Seconds

# Initialize database
conn = sqlite3.connect(DB_PATH)
cursor = conn.cursor()

# Create tables
cursor.execute("""
CREATE TABLE IF NOT EXISTS aircraft (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    hex TEXT NOT NULL,
    flight TEXT,
    type TEXT,
    altitude INTEGER,
    speed INTEGER,
    track REAL,
    lat REAL,
    lon REAL,
    distance REAL,
    military BOOLEAN,
    squawk TEXT,
    category TEXT
)
""")

cursor.execute("CREATE INDEX IF NOT EXISTS idx_hex ON aircraft(hex)")
cursor.execute("CREATE INDEX IF NOT EXISTS idx_timestamp ON aircraft(timestamp)")
cursor.execute("CREATE INDEX IF NOT EXISTS idx_military ON aircraft(military)")
cursor.execute("CREATE INDEX IF NOT EXISTS idx_type ON aircraft(type)")

conn.commit()

seen = {}


def log_aircraft(aircraft):
    """Log aircraft to database."""
    hex_code = aircraft.get("hex")

    # Rate limit per aircraft
    if hex_code in seen:
        if (datetime.now() - seen[hex_code]).total_seconds() < LOG_INTERVAL:
            return
    seen[hex_code] = datetime.now()

    cursor.execute("""
        INSERT INTO aircraft (hex, flight, type, altitude, speed, track, lat, lon, distance, military, squawk, category)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
    """, (
        hex_code,
        aircraft.get("flight"),
        aircraft.get("type"),
        aircraft.get("alt"),
        aircraft.get("gs"),
        aircraft.get("track"),
        aircraft.get("lat"),
        aircraft.get("lon"),
        aircraft.get("distance"),
        aircraft.get("military", False),
        aircraft.get("squawk"),
        aircraft.get("category")
    ))
    conn.commit()

    print(f"Logged: {aircraft.get('flight', hex_code)}")


def main():
    print(f"SQLite logger connected to {SKYSPY_URL}...")
    print(f"Database: {DB_PATH}")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    for aircraft in data.get("aircraft", []):
                        log_aircraft(aircraft)

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            import time
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const Database = require('better-sqlite3');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const DB_PATH = process.env.DB_PATH || 'skyspy.db';
const LOG_INTERVAL = parseInt(process.env.LOG_INTERVAL || '60') * 1000;

const db = new Database(DB_PATH);

db.exec(`
  CREATE TABLE IF NOT EXISTS aircraft (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    hex TEXT NOT NULL,
    flight TEXT,
    type TEXT,
    altitude INTEGER,
    speed INTEGER,
    track REAL,
    lat REAL,
    lon REAL,
    distance REAL,
    military BOOLEAN,
    squawk TEXT
  );
  CREATE INDEX IF NOT EXISTS idx_hex ON aircraft(hex);
  CREATE INDEX IF NOT EXISTS idx_timestamp ON aircraft(timestamp);
`);

const insert = db.prepare(`
  INSERT INTO aircraft (hex, flight, type, altitude, speed, track, lat, lon, distance, military, squawk)
  VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
`);

const seen = new Map();

console.log(`SQLite logger: ${DB_PATH}`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;

    if (seen.has(hex) && Date.now() - seen.get(hex) < LOG_INTERVAL) continue;
    seen.set(hex, Date.now());

    insert.run(
      hex,
      aircraft.flight || null,
      aircraft.type || null,
      aircraft.alt || null,
      aircraft.gs || null,
      aircraft.track || null,
      aircraft.lat || null,
      aircraft.lon || null,
      aircraft.distance || null,
      aircraft.military ? 1 : 0,
      aircraft.squawk || null
    );

    console.log(`Logged: ${aircraft.flight || hex}`);
  }
});
```

```sql Sample Queries
-- Total aircraft seen today
SELECT COUNT(DISTINCT hex) FROM aircraft
WHERE date(timestamp) = date('now');

-- Most common aircraft types
SELECT type, COUNT(*) as count FROM aircraft
WHERE type IS NOT NULL
GROUP BY type ORDER BY count DESC LIMIT 10;

-- All military aircraft
SELECT DISTINCT hex, flight, type, MAX(timestamp) as last_seen
FROM aircraft WHERE military = 1
GROUP BY hex ORDER BY last_seen DESC;

-- Aircraft seen in the last hour
SELECT * FROM aircraft
WHERE timestamp > datetime('now', '-1 hour')
ORDER BY timestamp DESC;

-- Average altitude by type
SELECT type, ROUND(AVG(altitude)) as avg_alt
FROM aircraft WHERE altitude > 0
GROUP BY type ORDER BY avg_alt DESC;
```

## Create Database Schema

<!-- python@17-35 -->

The schema includes indexes on frequently queried columns for fast lookups.

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER | Auto-incrementing primary key |
| `timestamp` | DATETIME | When the aircraft was logged |
| `hex` | TEXT | ICAO hex code |
| `flight` | TEXT | Callsign |
| `type` | TEXT | Aircraft type |
| `altitude` | INTEGER | Altitude in feet |
| `speed` | INTEGER | Ground speed in knots |
| `military` | BOOLEAN | Military aircraft flag |

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `DB_PATH` | Path to SQLite database | `skyspy.db` |
| `LOG_INTERVAL` | Seconds between logs per aircraft | `60` |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Query from Command Line

```shell
# Open database
sqlite3 skyspy.db

# Quick stats
sqlite3 skyspy.db "SELECT COUNT(*) FROM aircraft"

# Export to CSV
sqlite3 -header -csv skyspy.db "SELECT * FROM aircraft" > export.csv
```

### Python Query Helper

```python
import sqlite3

def query_aircraft(db_path="skyspy.db"):
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row

    # Today's military aircraft
    cursor = conn.execute("""
        SELECT DISTINCT hex, flight, type, MAX(timestamp) as last_seen
        FROM aircraft
        WHERE military = 1 AND date(timestamp) = date('now')
        GROUP BY hex
        ORDER BY last_seen DESC
    """)

    for row in cursor:
        print(f"{row['flight']} ({row['type']}) - {row['last_seen']}")

    conn.close()
```

### Retention Policy

Automatically delete old records:

```python
def cleanup_old_records(days=30):
    cursor.execute("""
        DELETE FROM aircraft
        WHERE timestamp < datetime('now', ?)
    """, (f'-{days} days',))
    conn.commit()
    print(f"Deleted records older than {days} days")

# Run daily
cleanup_old_records(30)
```

## Testing & Verification

1. **Start the logger** and verify database is created
2. **Check file size** grows over time:
   ```shell
   ls -lh skyspy.db
   ```
3. **Query the database**:
   ```shell
   sqlite3 skyspy.db "SELECT COUNT(*) FROM aircraft"
   ```
4. **Verify indexes** are working:
   ```sql
   EXPLAIN QUERY PLAN SELECT * FROM aircraft WHERE hex = 'A12345';
   ```

## Troubleshooting

### Database Locked

**Problem:** `database is locked` error
**Solution:**
- Only one writer at a time
- Use WAL mode for better concurrency:
  ```python
  cursor.execute("PRAGMA journal_mode=WAL")
  ```

### File Size Growing Too Fast

**Problem:** Database becomes very large
**Solution:**
- Increase `LOG_INTERVAL`
- Implement retention policy
- Run `VACUUM` periodically:
  ```sql
  VACUUM;
  ```

### Slow Queries

**Problem:** Queries taking too long
**Solution:**
- Add indexes for query patterns
- Use `EXPLAIN QUERY PLAN` to verify index usage
- Consider partitioning by date

## Related Recipes

- [Export to CSV](/docs/export-csv) - Simple file logging
- [PostgreSQL Archive](/docs/postgresql-archive) - Production database
- [InfluxDB Logging](/docs/influxdb-logging) - Time-series database
- [JSON Lines Logging](/docs/jsonl-logging) - Append-only logs
