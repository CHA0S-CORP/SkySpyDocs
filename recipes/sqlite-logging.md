---
title: SQLite Logging
description: Log aircraft sightings to a local SQLite database.
hidden: false
recipe:
  color: '#003B57'
  icon: 💾
---
```shell Shell
sqlite3 aircraft.db "CREATE TABLE IF NOT EXISTS sightings (
  id INTEGER PRIMARY KEY,
  hex TEXT,
  flight TEXT,
  type TEXT,
  alt INTEGER,
  lat REAL,
  lon REAL,
  seen_at DATETIME DEFAULT CURRENT_TIMESTAMP
);"
```

```go Go
package main

import (
	"database/sql"
	"time"

	_ "github.com/mattn/go-sqlite3"
)

type Aircraft struct {
	Hex, Flight, Type string
	Alt               int
	Lat, Lon          float64
}

func main() {
	db, _ := sql.Open("sqlite3", "./aircraft.db")
	defer db.Close()

	db.Exec(`CREATE TABLE IF NOT EXISTS sightings (
		id INTEGER PRIMARY KEY,
		hex TEXT, flight TEXT, type TEXT,
		alt INTEGER, lat REAL, lon REAL,
		seen_at DATETIME DEFAULT CURRENT_TIMESTAMP
	)`)

	ac := Aircraft{"A12345", "RCH419", "C-17", 35000, 34.05, -118.25}

	db.Exec(`INSERT INTO sightings (hex, flight, type, alt, lat, lon, seen_at)
		VALUES (?, ?, ?, ?, ?, ?, ?)`,
		ac.Hex, ac.Flight, ac.Type, ac.Alt, ac.Lat, ac.Lon, time.Now())
}
```

```python Python
import sqlite3
from datetime import datetime

conn = sqlite3.connect("aircraft.db")
cursor = conn.cursor()

cursor.execute("""
    CREATE TABLE IF NOT EXISTS sightings (
        id INTEGER PRIMARY KEY,
        hex TEXT,
        flight TEXT,
        type TEXT,
        alt INTEGER,
        lat REAL,
        lon REAL,
        seen_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
""")

def log_aircraft(aircraft):
    cursor.execute("""
        INSERT INTO sightings (hex, flight, type, alt, lat, lon, seen_at)
        VALUES (?, ?, ?, ?, ?, ?, ?)
    """, (
        aircraft["hex"], aircraft.get("flight"), aircraft.get("type"),
        aircraft.get("alt"), aircraft.get("lat"), aircraft.get("lon"),
        datetime.now()
    ))
    conn.commit()

# Example usage
log_aircraft({"hex": "A12345", "flight": "RCH419", "type": "C-17", "alt": 35000, "lat": 34.05, "lon": -118.25})
```

```javascript JavaScript
const Database = require('better-sqlite3');

const db = new Database('aircraft.db');

db.exec(`
    CREATE TABLE IF NOT EXISTS sightings (
        id INTEGER PRIMARY KEY,
        hex TEXT,
        flight TEXT,
        type TEXT,
        alt INTEGER,
        lat REAL,
        lon REAL,
        seen_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
`);

const insert = db.prepare(`
    INSERT INTO sightings (hex, flight, type, alt, lat, lon, seen_at)
    VALUES (?, ?, ?, ?, ?, ?, datetime('now'))
`);

function logAircraft(aircraft) {
    insert.run(aircraft.hex, aircraft.flight, aircraft.type, aircraft.alt, aircraft.lat, aircraft.lon);
}

// Example usage
logAircraft({ hex: 'A12345', flight: 'RCH419', type: 'C-17', alt: 35000, lat: 34.05, lon: -118.25 });
```

```json Response Example
{"lastInsertRowid": 1, "changes": 1}
```

# Create Database Schema

<!-- shell@1-9 -->
<!-- go@1-24 -->
<!-- python@1-18 -->
<!-- javascript@1-16 -->

Create a sightings table with columns for hex code, callsign, aircraft type, altitude, position, and timestamp.

# Insert Aircraft Record

<!-- go@26-30 -->
<!-- python@20-29 -->
<!-- javascript@18-24 -->

Log each aircraft sighting with all available data. The timestamp is automatically set to the current time.

# Query Examples

<!-- shell@1 -->
<!-- go@17 -->
<!-- python@31 -->
<!-- javascript@26 -->

Query with: `SELECT * FROM sightings WHERE type='C-17' ORDER BY seen_at DESC LIMIT 10`
