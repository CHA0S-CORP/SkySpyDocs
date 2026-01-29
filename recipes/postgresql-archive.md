---
title: PostgreSQL Archive
excerpt: Persistent aircraft history with PostgreSQL
hidden: false
recipe:
  color: '#336791'
  icon: "🐘"
difficulty: intermediate
tags: [data, postgresql, database, analytics]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- PostgreSQL 12+ running (local, Docker, or cloud)
- Python: `pip install sseclient-py requests psycopg2-binary`
- Go: `go get github.com/lib/pq`
- Node.js: `npm install eventsource pg`

## What You'll Build

A production-ready data pipeline that streams aircraft positions to PostgreSQL for historical analysis, complex queries, and integration with analytics tools.

```shell Shell
# Install dependencies
pip install sseclient-py requests psycopg2-binary

# Set environment variables
export POSTGRES_HOST="localhost"
export POSTGRES_PORT="5432"
export POSTGRES_DB="skyspy"
export POSTGRES_USER="skyspy"
export POSTGRES_PASSWORD="your-password"
export SKYSPY_URL="http://localhost:5000"

# Run the logger
python postgresql_logger.py
```

```python Python
import json
import os
import sys
import time
from datetime import datetime
import requests
import sseclient
import psycopg2
from psycopg2.extras import execute_values

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
POSTGRES_HOST = os.getenv("POSTGRES_HOST", "localhost")
POSTGRES_PORT = os.getenv("POSTGRES_PORT", "5432")
POSTGRES_DB = os.getenv("POSTGRES_DB", "skyspy")
POSTGRES_USER = os.getenv("POSTGRES_USER", "skyspy")
POSTGRES_PASSWORD = os.getenv("POSTGRES_PASSWORD", "")
LOG_INTERVAL = int(os.getenv("LOG_INTERVAL", "60"))

# Rate limiting per aircraft
last_logged = {}


def get_connection():
    """Create PostgreSQL connection."""
    return psycopg2.connect(
        host=POSTGRES_HOST,
        port=POSTGRES_PORT,
        dbname=POSTGRES_DB,
        user=POSTGRES_USER,
        password=POSTGRES_PASSWORD
    )


def init_database(conn):
    """Initialize database schema."""
    cursor = conn.cursor()

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS aircraft_positions (
            id BIGSERIAL PRIMARY KEY,
            timestamp TIMESTAMPTZ DEFAULT NOW(),
            hex VARCHAR(10) NOT NULL,
            flight VARCHAR(20),
            aircraft_type VARCHAR(10),
            altitude INTEGER,
            speed INTEGER,
            track REAL,
            lat DOUBLE PRECISION,
            lon DOUBLE PRECISION,
            distance REAL,
            military BOOLEAN DEFAULT FALSE,
            squawk VARCHAR(10),
            category VARCHAR(5),
            vertical_rate INTEGER
        )
    """)

    # Create indexes for common queries
    cursor.execute("CREATE INDEX IF NOT EXISTS idx_positions_hex ON aircraft_positions(hex)")
    cursor.execute("CREATE INDEX IF NOT EXISTS idx_positions_timestamp ON aircraft_positions(timestamp)")
    cursor.execute("CREATE INDEX IF NOT EXISTS idx_positions_military ON aircraft_positions(military)")
    cursor.execute("CREATE INDEX IF NOT EXISTS idx_positions_flight ON aircraft_positions(flight)")
    cursor.execute("CREATE INDEX IF NOT EXISTS idx_positions_hex_timestamp ON aircraft_positions(hex, timestamp)")

    conn.commit()
    print("Database schema initialized")


def log_aircraft(cursor, aircraft):
    """Log aircraft position to database."""
    hex_code = aircraft.get("hex")
    now = time.time()

    # Rate limit per aircraft
    if hex_code in last_logged:
        if now - last_logged[hex_code] < LOG_INTERVAL:
            return False

    last_logged[hex_code] = now

    cursor.execute("""
        INSERT INTO aircraft_positions
        (hex, flight, aircraft_type, altitude, speed, track, lat, lon, distance, military, squawk, category, vertical_rate)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
    """, (
        hex_code,
        aircraft.get("flight", "").strip() or None,
        aircraft.get("type"),
        aircraft.get("alt"),
        aircraft.get("gs"),
        aircraft.get("track"),
        aircraft.get("lat"),
        aircraft.get("lon"),
        aircraft.get("distance"),
        aircraft.get("military", False),
        aircraft.get("squawk"),
        aircraft.get("category"),
        aircraft.get("baro_rate")
    ))

    return True


def main():
    print(f"PostgreSQL Logger connected to {SKYSPY_URL}")
    print(f"Database: {POSTGRES_HOST}:{POSTGRES_PORT}/{POSTGRES_DB}")

    conn = get_connection()
    init_database(conn)
    cursor = conn.cursor()

    logged_count = 0

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            sse_client = sseclient.SSEClient(response)

            for event in sse_client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)

                    for aircraft in data.get("aircraft", []):
                        if log_aircraft(cursor, aircraft):
                            logged_count += 1
                            print(f"Logged: {aircraft.get('flight', aircraft.get('hex'))} (total: {logged_count})")

                    conn.commit()

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            try:
                conn.rollback()
            except:
                conn = get_connection()
                cursor = conn.cursor()
            time.sleep(5)


if __name__ == "__main__":
    main()
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

	_ "github.com/lib/pq"
)

var lastLogged = make(map[string]time.Time)
var logInterval = 60 * time.Second

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	pgHost := os.Getenv("POSTGRES_HOST")
	if pgHost == "" {
		pgHost = "localhost"
	}
	pgPort := os.Getenv("POSTGRES_PORT")
	if pgPort == "" {
		pgPort = "5432"
	}
	pgDB := os.Getenv("POSTGRES_DB")
	if pgDB == "" {
		pgDB = "skyspy"
	}
	pgUser := os.Getenv("POSTGRES_USER")
	if pgUser == "" {
		pgUser = "skyspy"
	}
	pgPassword := os.Getenv("POSTGRES_PASSWORD")

	connStr := fmt.Sprintf("host=%s port=%s user=%s password=%s dbname=%s sslmode=disable",
		pgHost, pgPort, pgUser, pgPassword, pgDB)

	db, err := sql.Open("postgres", connStr)
	if err != nil {
		fmt.Printf("Failed to connect to database: %v\n", err)
		os.Exit(1)
	}
	defer db.Close()

	// Initialize schema
	db.Exec(`CREATE TABLE IF NOT EXISTS aircraft_positions (
		id BIGSERIAL PRIMARY KEY,
		timestamp TIMESTAMPTZ DEFAULT NOW(),
		hex VARCHAR(10) NOT NULL,
		flight VARCHAR(20),
		aircraft_type VARCHAR(10),
		altitude INTEGER,
		speed INTEGER,
		track REAL,
		lat DOUBLE PRECISION,
		lon DOUBLE PRECISION,
		distance REAL,
		military BOOLEAN DEFAULT FALSE,
		squawk VARCHAR(10),
		category VARCHAR(5),
		vertical_rate INTEGER
	)`)
	db.Exec("CREATE INDEX IF NOT EXISTS idx_positions_hex ON aircraft_positions(hex)")
	db.Exec("CREATE INDEX IF NOT EXISTS idx_positions_timestamp ON aircraft_positions(timestamp)")

	stmt, _ := db.Prepare(`INSERT INTO aircraft_positions
		(hex, flight, aircraft_type, altitude, speed, track, lat, lon, distance, military, squawk, category, vertical_rate)
		VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13)`)
	defer stmt.Close()

	fmt.Printf("PostgreSQL Logger connected to %s\n", skyspyURL)
	fmt.Printf("Database: %s:%s/%s\n", pgHost, pgPort, pgDB)

	loggedCount := 0

	for {
		resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
		if err != nil {
			time.Sleep(5 * time.Second)
			continue
		}

		scanner := bufio.NewScanner(resp.Body)
		for scanner.Scan() {
			line := scanner.Text()
			if strings.HasPrefix(line, "data:") {
				var data map[string]interface{}
				json.Unmarshal([]byte(line[5:]), &data)

				if aircraft, ok := data["aircraft"].([]interface{}); ok {
					now := time.Now()

					for _, a := range aircraft {
						ac := a.(map[string]interface{})
						hex := fmt.Sprint(ac["hex"])

						// Rate limit
						if last, exists := lastLogged[hex]; exists {
							if now.Sub(last) < logInterval {
								continue
							}
						}
						lastLogged[hex] = now

						flight := strings.TrimSpace(fmt.Sprint(ac["flight"]))
						if flight == "<nil>" {
							flight = ""
						}

						acType := fmt.Sprint(ac["type"])
						if acType == "<nil>" {
							acType = ""
						}

						stmt.Exec(
							hex,
							nullString(flight),
							nullString(acType),
							ac["alt"],
							ac["gs"],
							ac["track"],
							ac["lat"],
							ac["lon"],
							ac["distance"],
							ac["military"] == true,
							ac["squawk"],
							ac["category"],
							ac["baro_rate"],
						)

						loggedCount++
						if loggedCount%100 == 0 {
							fmt.Printf("Total logged: %d\n", loggedCount)
						}
					}
				}
			}
		}
		resp.Body.Close()
	}
}

func nullString(s string) interface{} {
	if s == "" {
		return nil
	}
	return s
}
```

```javascript JavaScript
const EventSource = require('eventsource');
const { Pool } = require('pg');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const LOG_INTERVAL = parseInt(process.env.LOG_INTERVAL || '60') * 1000;

const pool = new Pool({
  host: process.env.POSTGRES_HOST || 'localhost',
  port: parseInt(process.env.POSTGRES_PORT || '5432'),
  database: process.env.POSTGRES_DB || 'skyspy',
  user: process.env.POSTGRES_USER || 'skyspy',
  password: process.env.POSTGRES_PASSWORD || '',
});

const lastLogged = new Map();
let loggedCount = 0;

async function initDatabase() {
  const client = await pool.connect();
  try {
    await client.query(`
      CREATE TABLE IF NOT EXISTS aircraft_positions (
        id BIGSERIAL PRIMARY KEY,
        timestamp TIMESTAMPTZ DEFAULT NOW(),
        hex VARCHAR(10) NOT NULL,
        flight VARCHAR(20),
        aircraft_type VARCHAR(10),
        altitude INTEGER,
        speed INTEGER,
        track REAL,
        lat DOUBLE PRECISION,
        lon DOUBLE PRECISION,
        distance REAL,
        military BOOLEAN DEFAULT FALSE,
        squawk VARCHAR(10),
        category VARCHAR(5),
        vertical_rate INTEGER
      )
    `);
    await client.query('CREATE INDEX IF NOT EXISTS idx_positions_hex ON aircraft_positions(hex)');
    await client.query('CREATE INDEX IF NOT EXISTS idx_positions_timestamp ON aircraft_positions(timestamp)');
    console.log('Database schema initialized');
  } finally {
    client.release();
  }
}

async function logAircraft(aircraft) {
  const hex = aircraft.hex;
  const now = Date.now();

  // Rate limit per aircraft
  if (lastLogged.has(hex) && now - lastLogged.get(hex) < LOG_INTERVAL) {
    return;
  }
  lastLogged.set(hex, now);

  try {
    await pool.query(`
      INSERT INTO aircraft_positions
      (hex, flight, aircraft_type, altitude, speed, track, lat, lon, distance, military, squawk, category, vertical_rate)
      VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13)
    `, [
      hex,
      aircraft.flight?.trim() || null,
      aircraft.type || null,
      aircraft.alt || null,
      aircraft.gs || null,
      aircraft.track || null,
      aircraft.lat || null,
      aircraft.lon || null,
      aircraft.distance || null,
      aircraft.military || false,
      aircraft.squawk || null,
      aircraft.category || null,
      aircraft.baro_rate || null,
    ]);

    loggedCount++;
    console.log(`Logged: ${aircraft.flight || hex} (total: ${loggedCount})`);
  } catch (err) {
    console.error('Insert error:', err.message);
  }
}

async function main() {
  await initDatabase();

  console.log(`PostgreSQL Logger connected to ${SKYSPY_URL}`);
  console.log(`Database: ${process.env.POSTGRES_HOST || 'localhost'}:${process.env.POSTGRES_PORT || '5432'}/${process.env.POSTGRES_DB || 'skyspy'}`);

  const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

  es.addEventListener('aircraft_update', async (e) => {
    const data = JSON.parse(e.data);
    for (const aircraft of data.aircraft || []) {
      await logAircraft(aircraft);
    }
  });

  es.addEventListener('aircraft_new', async (e) => {
    const data = JSON.parse(e.data);
    for (const aircraft of data.aircraft || []) {
      await logAircraft(aircraft);
    }
  });

  es.onerror = (err) => console.error('SSE error:', err);
}

main().catch(console.error);
```

## Database Schema

The schema stores aircraft position snapshots with timestamps for historical analysis.

| Column | Type | Description |
|--------|------|-------------|
| `id` | BIGSERIAL | Auto-incrementing primary key |
| `timestamp` | TIMESTAMPTZ | When the position was logged |
| `hex` | VARCHAR(10) | ICAO hex code (indexed) |
| `flight` | VARCHAR(20) | Callsign/flight number |
| `aircraft_type` | VARCHAR(10) | Aircraft type code |
| `altitude` | INTEGER | Altitude in feet |
| `speed` | INTEGER | Ground speed in knots |
| `track` | REAL | Track angle in degrees |
| `lat` | DOUBLE PRECISION | Latitude |
| `lon` | DOUBLE PRECISION | Longitude |
| `distance` | REAL | Distance from receiver |
| `military` | BOOLEAN | Military aircraft flag |
| `squawk` | VARCHAR(10) | Transponder code |
| `category` | VARCHAR(5) | Aircraft category |
| `vertical_rate` | INTEGER | Climb/descent rate in ft/min |

## Docker Quick Start

```shell
# Start PostgreSQL with Docker
docker run -d --name skyspy-postgres \
  -p 5432:5432 \
  -e POSTGRES_USER=skyspy \
  -e POSTGRES_PASSWORD=skyspy123 \
  -e POSTGRES_DB=skyspy \
  -v skyspy-pgdata:/var/lib/postgresql/data \
  postgres:15

# Verify connection
docker exec -it skyspy-postgres psql -U skyspy -c "SELECT version();"

# Connect with psql
docker exec -it skyspy-postgres psql -U skyspy
```

### Docker Compose

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    container_name: skyspy-postgres
    environment:
      POSTGRES_USER: skyspy
      POSTGRES_PASSWORD: skyspy123
      POSTGRES_DB: skyspy
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  pgdata:
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `POSTGRES_HOST` | PostgreSQL host | `localhost` |
| `POSTGRES_PORT` | PostgreSQL port | `5432` |
| `POSTGRES_DB` | Database name | `skyspy` |
| `POSTGRES_USER` | Database user | `skyspy` |
| `POSTGRES_PASSWORD` | Database password | Required |
| `LOG_INTERVAL` | Seconds between logs per aircraft | `60` |

## Example SQL Queries

### Unique Aircraft Today

```sql
SELECT COUNT(DISTINCT hex) as unique_aircraft
FROM aircraft_positions
WHERE timestamp >= CURRENT_DATE;
```

### Most Seen Aircraft

```sql
SELECT hex, flight, aircraft_type,
       COUNT(*) as sightings,
       MIN(timestamp) as first_seen,
       MAX(timestamp) as last_seen
FROM aircraft_positions
WHERE timestamp >= NOW() - INTERVAL '7 days'
GROUP BY hex, flight, aircraft_type
ORDER BY sightings DESC
LIMIT 20;
```

### Flight Path for Specific Aircraft

```sql
SELECT timestamp, lat, lon, altitude, speed, track
FROM aircraft_positions
WHERE hex = 'A12345'
  AND timestamp >= NOW() - INTERVAL '24 hours'
ORDER BY timestamp;
```

### Military Aircraft Activity

```sql
SELECT hex, flight, aircraft_type,
       COUNT(*) as positions,
       MIN(timestamp) as first_seen,
       MAX(timestamp) as last_seen
FROM aircraft_positions
WHERE military = true
  AND timestamp >= NOW() - INTERVAL '24 hours'
GROUP BY hex, flight, aircraft_type
ORDER BY last_seen DESC;
```

### Hourly Statistics

```sql
SELECT
    DATE_TRUNC('hour', timestamp) as hour,
    COUNT(*) as positions,
    COUNT(DISTINCT hex) as unique_aircraft,
    ROUND(AVG(altitude)) as avg_altitude,
    ROUND(AVG(speed)) as avg_speed
FROM aircraft_positions
WHERE timestamp >= NOW() - INTERVAL '24 hours'
GROUP BY DATE_TRUNC('hour', timestamp)
ORDER BY hour DESC;
```

### Aircraft by Type

```sql
SELECT aircraft_type,
       COUNT(DISTINCT hex) as aircraft_count,
       COUNT(*) as total_positions
FROM aircraft_positions
WHERE aircraft_type IS NOT NULL
  AND timestamp >= NOW() - INTERVAL '7 days'
GROUP BY aircraft_type
ORDER BY aircraft_count DESC
LIMIT 20;
```

### Busiest Times

```sql
SELECT
    EXTRACT(HOUR FROM timestamp) as hour_of_day,
    ROUND(AVG(aircraft_count)) as avg_aircraft
FROM (
    SELECT timestamp, COUNT(DISTINCT hex) as aircraft_count
    FROM aircraft_positions
    WHERE timestamp >= NOW() - INTERVAL '7 days'
    GROUP BY DATE_TRUNC('minute', timestamp)
) subquery
GROUP BY EXTRACT(HOUR FROM timestamp)
ORDER BY hour_of_day;
```

### Geographic Bounding Box Query

```sql
SELECT DISTINCT hex, flight, aircraft_type, lat, lon, altitude
FROM aircraft_positions
WHERE lat BETWEEN 40.0 AND 42.0
  AND lon BETWEEN -75.0 AND -73.0
  AND timestamp >= NOW() - INTERVAL '1 hour'
ORDER BY timestamp DESC;
```

## Data Retention

### Automatic Cleanup with pg_cron

```sql
-- Install pg_cron extension (if available)
CREATE EXTENSION IF NOT EXISTS pg_cron;

-- Delete records older than 30 days, daily at 3 AM
SELECT cron.schedule('cleanup-old-positions', '0 3 * * *', $$
    DELETE FROM aircraft_positions
    WHERE timestamp < NOW() - INTERVAL '30 days'
$$);
```

### Manual Cleanup Script

```python
def cleanup_old_records(conn, days=30):
    cursor = conn.cursor()
    cursor.execute("""
        DELETE FROM aircraft_positions
        WHERE timestamp < NOW() - INTERVAL '%s days'
    """, (days,))
    deleted = cursor.rowcount
    conn.commit()
    print(f"Deleted {deleted} records older than {days} days")
    return deleted
```

### Partitioning for Large Datasets

```sql
-- Create partitioned table for better performance
CREATE TABLE aircraft_positions_partitioned (
    id BIGSERIAL,
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    hex VARCHAR(10) NOT NULL,
    flight VARCHAR(20),
    aircraft_type VARCHAR(10),
    altitude INTEGER,
    speed INTEGER,
    track REAL,
    lat DOUBLE PRECISION,
    lon DOUBLE PRECISION,
    distance REAL,
    military BOOLEAN DEFAULT FALSE,
    squawk VARCHAR(10),
    category VARCHAR(5),
    vertical_rate INTEGER,
    PRIMARY KEY (id, timestamp)
) PARTITION BY RANGE (timestamp);

-- Create monthly partitions
CREATE TABLE aircraft_positions_2024_01 PARTITION OF aircraft_positions_partitioned
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE aircraft_positions_2024_02 PARTITION OF aircraft_positions_partitioned
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

## Testing & Verification

1. **Start PostgreSQL** and verify connection:
   ```shell
   psql -h localhost -U skyspy -d skyspy -c "SELECT 1"
   ```

2. **Run the logger** and check output for successful inserts

3. **Verify data is being stored**:
   ```sql
   SELECT COUNT(*) FROM aircraft_positions;
   SELECT * FROM aircraft_positions ORDER BY timestamp DESC LIMIT 5;
   ```

4. **Check index usage**:
   ```sql
   EXPLAIN ANALYZE SELECT * FROM aircraft_positions WHERE hex = 'A12345';
   ```

5. **Monitor table size**:
   ```sql
   SELECT pg_size_pretty(pg_total_relation_size('aircraft_positions'));
   ```

## Troubleshooting

### Connection Refused

**Problem:** `could not connect to server: Connection refused`
**Solution:**
- Verify PostgreSQL is running: `pg_isready -h localhost -p 5432`
- Check host/port configuration
- Ensure PostgreSQL is accepting connections in `pg_hba.conf`

### Authentication Failed

**Problem:** `password authentication failed for user`
**Solution:**
- Verify username and password
- Check `pg_hba.conf` authentication method
- Ensure user has database access:
  ```sql
  GRANT ALL PRIVILEGES ON DATABASE skyspy TO skyspy;
  ```

### Slow Insert Performance

**Problem:** Inserts are slow with many aircraft
**Solution:**
- Use batch inserts with `execute_values` (Python)
- Increase `LOG_INTERVAL` to reduce write frequency
- Consider async inserts or connection pooling
- Use `UNLOGGED` tables for higher write throughput (no crash recovery)

### Table Growing Too Large

**Problem:** Database size increasing rapidly
**Solution:**
- Implement retention policy with scheduled cleanup
- Increase `LOG_INTERVAL` to log less frequently
- Use table partitioning for easier data management
- Run `VACUUM ANALYZE` periodically:
  ```sql
  VACUUM ANALYZE aircraft_positions;
  ```

### High Memory Usage

**Problem:** PostgreSQL using too much memory
**Solution:**
- Tune `shared_buffers` and `work_mem` in `postgresql.conf`
- Add connection pooling (PgBouncer)
- Limit concurrent connections

## Related Recipes

- [SQLite Local Database](/docs/sqlite-db) - Lightweight local storage
- [InfluxDB Logging](/docs/influxdb-logging) - Time-series database
- [Grafana Dashboard](/docs/grafana-dashboard) - Visualize PostgreSQL data
- [Export to CSV](/docs/export-csv) - Export data for analysis
