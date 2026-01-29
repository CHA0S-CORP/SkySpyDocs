---
title: PostgreSQL Archive
description: Store aircraft history in PostgreSQL for analysis.
hidden: false
recipe:
  color: '#336791'
  icon: 🐘
---
```shell Shell
psql -c "CREATE TABLE aircraft_sightings (
  id SERIAL PRIMARY KEY,
  hex VARCHAR(6) NOT NULL,
  flight VARCHAR(10),
  type VARCHAR(10),
  alt INTEGER,
  lat NUMERIC(9,6),
  lon NUMERIC(9,6),
  gs INTEGER,
  track INTEGER,
  seen_at TIMESTAMP DEFAULT NOW()
);"
```

```go Go
package main

import (
	"database/sql"
	"encoding/json"
	"net/http"
	"os"

	_ "github.com/lib/pq"
)

func main() {
	connStr := os.Getenv("DATABASE_URL")
	db, _ := sql.Open("postgres", connStr)
	defer db.Close()

	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct {
		Aircraft []struct {
			Hex, Flight, Type string
			Alt, Gs, Track    int
			Lat, Lon          float64
		} `json:"aircraft"`
	}
	json.NewDecoder(resp.Body).Decode(&data)
	resp.Body.Close()

	stmt, _ := db.Prepare(`INSERT INTO aircraft_sightings
		(hex, flight, type, alt, lat, lon, gs, track) VALUES ($1,$2,$3,$4,$5,$6,$7,$8)`)

	for _, ac := range data.Aircraft {
		stmt.Exec(ac.Hex, ac.Flight, ac.Type, ac.Alt, ac.Lat, ac.Lon, ac.Gs, ac.Track)
	}
}
```

```python Python
import os
import psycopg2
import requests

DATABASE_URL = os.getenv("DATABASE_URL")
SKYSPY_URL = "http://localhost:5000"

conn = psycopg2.connect(DATABASE_URL)
cursor = conn.cursor()

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

for ac in aircraft:
    cursor.execute("""
        INSERT INTO aircraft_sightings (hex, flight, type, alt, lat, lon, gs, track)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
    """, (
        ac.get("hex"), ac.get("flight"), ac.get("type"),
        ac.get("alt"), ac.get("lat"), ac.get("lon"),
        ac.get("gs"), ac.get("track")
    ))

conn.commit()
print(f"Archived {len(aircraft)} aircraft")
```

```javascript JavaScript
const { Pool } = require('pg');

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const SKYSPY_URL = 'http://localhost:5000';

async function archiveAircraft() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    const client = await pool.connect();
    try {
        for (const ac of data.aircraft || []) {
            await client.query(
                `INSERT INTO aircraft_sightings (hex, flight, type, alt, lat, lon, gs, track)
                 VALUES ($1, $2, $3, $4, $5, $6, $7, $8)`,
                [ac.hex, ac.flight, ac.type, ac.alt, ac.lat, ac.lon, ac.gs, ac.track]
            );
        }
        console.log(`Archived ${data.aircraft?.length || 0} aircraft`);
    } finally {
        client.release();
    }
}

archiveAircraft();
```

```json Response Example
{"rows_inserted": 150, "timestamp": "2024-01-15T10:30:00Z"}
```

# Create Database Schema

<!-- shell@1-12 -->
<!-- go@1-11 -->
<!-- python@1-9 -->
<!-- javascript@1-4 -->

Create a table with columns for all aircraft fields. Use SERIAL for auto-incrementing IDs and TIMESTAMP for tracking.

# Fetch Aircraft Data

<!-- go@14-24 -->
<!-- python@11-12 -->
<!-- javascript@6-8 -->

Get current aircraft from SkySpy. Run this on a schedule (cron) to build historical data.

# Insert Records

<!-- go@26-30 -->
<!-- python@14-22 -->
<!-- javascript@10-20 -->

Use parameterized queries to safely insert data. PostgreSQL handles the timestamp automatically.
