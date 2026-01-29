---
title: InfluxDB Logging
excerpt: Store aircraft data in InfluxDB for time-series analysis.
hidden: false
recipe:
  color: '#22ADF6'
  icon: 📈
difficulty: intermediate
tags: [data, influxdb, time-series, analytics]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- InfluxDB 2.x running (local or cloud)
- InfluxDB organization, bucket, and API token
- Python: `pip install sseclient-py requests influxdb-client`

## What You'll Build

A data pipeline that streams aircraft positions to InfluxDB for time-series analysis, historical queries, and Grafana visualization.

```shell Shell
pip install sseclient-py requests influxdb-client
export INFLUX_URL="http://localhost:8086"
export INFLUX_TOKEN="your-token"
export INFLUX_ORG="your-org"
export INFLUX_BUCKET="skyspy"
python influxdb_logger.py
```

```python Python
import json
import os
import sys
import time
import requests
import sseclient
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
INFLUX_URL = os.getenv("INFLUX_URL", "http://localhost:8086")
INFLUX_TOKEN = os.getenv("INFLUX_TOKEN")
INFLUX_ORG = os.getenv("INFLUX_ORG")
INFLUX_BUCKET = os.getenv("INFLUX_BUCKET", "skyspy")
WRITE_INTERVAL = int(os.getenv("WRITE_INTERVAL", "10"))

if not all([INFLUX_TOKEN, INFLUX_ORG]):
    print("Error: INFLUX_TOKEN and INFLUX_ORG required")
    sys.exit(1)

# Rate limiting per aircraft
last_write = {}


def create_point(aircraft):
    """Create an InfluxDB point from aircraft data."""
    hex_code = aircraft.get("hex", "unknown")

    point = Point("aircraft") \
        .tag("hex", hex_code) \
        .tag("flight", aircraft.get("flight", "").strip() or "unknown") \
        .tag("type", aircraft.get("type", "unknown")) \
        .tag("category", aircraft.get("category", "unknown")) \
        .tag("military", str(aircraft.get("military", False)).lower())

    # Add numeric fields
    if aircraft.get("alt") is not None:
        point = point.field("altitude", float(aircraft["alt"]))
    if aircraft.get("gs") is not None:
        point = point.field("speed", float(aircraft["gs"]))
    if aircraft.get("track") is not None:
        point = point.field("track", float(aircraft["track"]))
    if aircraft.get("lat") is not None:
        point = point.field("lat", float(aircraft["lat"]))
    if aircraft.get("lon") is not None:
        point = point.field("lon", float(aircraft["lon"]))
    if aircraft.get("distance") is not None:
        point = point.field("distance", float(aircraft["distance"]))
    if aircraft.get("baro_rate") is not None:
        point = point.field("vertical_rate", float(aircraft["baro_rate"]))
    if aircraft.get("squawk"):
        point = point.field("squawk", aircraft["squawk"])

    return point


def main():
    print(f"InfluxDB Logger connected to {SKYSPY_URL}")
    print(f"Writing to {INFLUX_URL} / {INFLUX_ORG} / {INFLUX_BUCKET}")

    client = InfluxDBClient(url=INFLUX_URL, token=INFLUX_TOKEN, org=INFLUX_ORG)
    write_api = client.write_api(write_options=SYNCHRONOUS)

    points_written = 0

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            sse_client = sseclient.SSEClient(response)

            for event in sse_client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    points = []
                    now = time.time()

                    for aircraft in data.get("aircraft", []):
                        hex_code = aircraft.get("hex")

                        # Rate limit per aircraft
                        if hex_code in last_write:
                            if now - last_write[hex_code] < WRITE_INTERVAL:
                                continue

                        # Only write if we have position
                        if aircraft.get("lat") and aircraft.get("lon"):
                            points.append(create_point(aircraft))
                            last_write[hex_code] = now

                    if points:
                        write_api.write(bucket=INFLUX_BUCKET, record=points)
                        points_written += len(points)
                        print(f"Wrote {len(points)} points (total: {points_written})")

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```go Go
package main

import (
	"bufio"
	"context"
	"encoding/json"
	"fmt"
	"net/http"
	"os"
	"strings"
	"time"

	influxdb2 "github.com/influxdata/influxdb-client-go/v2"
)

var lastWrite = make(map[string]time.Time)

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	influxURL := os.Getenv("INFLUX_URL")
	if influxURL == "" {
		influxURL = "http://localhost:8086"
	}

	token := os.Getenv("INFLUX_TOKEN")
	org := os.Getenv("INFLUX_ORG")
	bucket := os.Getenv("INFLUX_BUCKET")
	if bucket == "" {
		bucket = "skyspy"
	}

	if token == "" || org == "" {
		fmt.Println("Error: INFLUX_TOKEN and INFLUX_ORG required")
		os.Exit(1)
	}

	client := influxdb2.NewClient(influxURL, token)
	writeAPI := client.WriteAPIBlocking(org, bucket)

	fmt.Printf("InfluxDB Logger connected to %s\n", skyspyURL)
	fmt.Printf("Writing to %s / %s / %s\n", influxURL, org, bucket)

	pointsWritten := 0

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
						if last, exists := lastWrite[hex]; exists {
							if now.Sub(last) < 10*time.Second {
								continue
							}
						}

						// Need position
						lat, hasLat := ac["lat"].(float64)
						lon, hasLon := ac["lon"].(float64)
						if !hasLat || !hasLon {
							continue
						}

						flight := strings.TrimSpace(fmt.Sprint(ac["flight"]))
						if flight == "<nil>" {
							flight = "unknown"
						}

						acType := fmt.Sprint(ac["type"])
						if acType == "<nil>" {
							acType = "unknown"
						}

						military := ac["military"] == true

						p := influxdb2.NewPoint(
							"aircraft",
							map[string]string{
								"hex":      hex,
								"flight":   flight,
								"type":     acType,
								"military": fmt.Sprint(military),
							},
							map[string]interface{}{
								"lat": lat,
								"lon": lon,
							},
							now,
						)

						if alt, ok := ac["alt"].(float64); ok {
							p = p.AddField("altitude", alt)
						}
						if gs, ok := ac["gs"].(float64); ok {
							p = p.AddField("speed", gs)
						}
						if dist, ok := ac["distance"].(float64); ok {
							p = p.AddField("distance", dist)
						}

						writeAPI.WritePoint(context.Background(), p)
						lastWrite[hex] = now
						pointsWritten++
					}

					if pointsWritten > 0 && pointsWritten%100 == 0 {
						fmt.Printf("Total points written: %d\n", pointsWritten)
					}
				}
			}
		}
		resp.Body.Close()
	}
}
```

```javascript JavaScript
const EventSource = require('eventsource');
const { InfluxDB, Point } = require('@influxdata/influxdb-client');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const INFLUX_URL = process.env.INFLUX_URL || 'http://localhost:8086';
const INFLUX_TOKEN = process.env.INFLUX_TOKEN;
const INFLUX_ORG = process.env.INFLUX_ORG;
const INFLUX_BUCKET = process.env.INFLUX_BUCKET || 'skyspy';
const WRITE_INTERVAL = parseInt(process.env.WRITE_INTERVAL || '10') * 1000;

if (!INFLUX_TOKEN || !INFLUX_ORG) {
  console.error('Error: INFLUX_TOKEN and INFLUX_ORG required');
  process.exit(1);
}

const client = new InfluxDB({ url: INFLUX_URL, token: INFLUX_TOKEN });
const writeApi = client.getWriteApi(INFLUX_ORG, INFLUX_BUCKET);

const lastWrite = new Map();
let pointsWritten = 0;

console.log(`InfluxDB Logger connected to ${SKYSPY_URL}`);
console.log(`Writing to ${INFLUX_URL} / ${INFLUX_ORG} / ${INFLUX_BUCKET}`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);
  const now = Date.now();

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;

    // Rate limit
    if (lastWrite.has(hex) && now - lastWrite.get(hex) < WRITE_INTERVAL) {
      continue;
    }

    // Need position
    if (!aircraft.lat || !aircraft.lon) continue;

    const point = new Point('aircraft')
      .tag('hex', hex)
      .tag('flight', (aircraft.flight || 'unknown').trim())
      .tag('type', aircraft.type || 'unknown')
      .tag('military', String(aircraft.military || false))
      .floatField('lat', aircraft.lat)
      .floatField('lon', aircraft.lon);

    if (aircraft.alt != null) point.floatField('altitude', aircraft.alt);
    if (aircraft.gs != null) point.floatField('speed', aircraft.gs);
    if (aircraft.distance != null) point.floatField('distance', aircraft.distance);
    if (aircraft.track != null) point.floatField('track', aircraft.track);

    writeApi.writePoint(point);
    lastWrite.set(hex, now);
    pointsWritten++;

    if (pointsWritten % 100 === 0) {
      console.log(`Total points written: ${pointsWritten}`);
    }
  }
});

es.onerror = (err) => console.error('SSE error:', err);

// Flush on exit
process.on('SIGINT', async () => {
  await writeApi.close();
  process.exit(0);
});
```

## InfluxDB Setup

### Docker Quick Start

```shell
docker run -d --name influxdb \
  -p 8086:8086 \
  -e DOCKER_INFLUXDB_INIT_MODE=setup \
  -e DOCKER_INFLUXDB_INIT_USERNAME=admin \
  -e DOCKER_INFLUXDB_INIT_PASSWORD=adminpassword \
  -e DOCKER_INFLUXDB_INIT_ORG=skyspy \
  -e DOCKER_INFLUXDB_INIT_BUCKET=aircraft \
  -v influxdb-data:/var/lib/influxdb2 \
  influxdb:2.7
```

### Create API Token

```shell
# Via UI: http://localhost:8086 -> Data -> API Tokens -> Generate

# Or via CLI
influx auth create \
  --org skyspy \
  --description "SkySpy writer" \
  --write-bucket aircraft
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `INFLUX_URL` | InfluxDB server URL | `http://localhost:8086` |
| `INFLUX_TOKEN` | InfluxDB API token | Required |
| `INFLUX_ORG` | InfluxDB organization | Required |
| `INFLUX_BUCKET` | InfluxDB bucket | `skyspy` |
| `WRITE_INTERVAL` | Seconds between writes per aircraft | `10` |

## Data Schema

### Measurement: `aircraft`

**Tags (indexed):**
| Tag | Description |
|-----|-------------|
| `hex` | ICAO hex code |
| `flight` | Callsign/flight number |
| `type` | Aircraft type code |
| `military` | "true" or "false" |

**Fields (values):**
| Field | Type | Description |
|-------|------|-------------|
| `lat` | float | Latitude |
| `lon` | float | Longitude |
| `altitude` | float | Altitude in feet |
| `speed` | float | Ground speed in knots |
| `track` | float | Track angle |
| `distance` | float | Distance from receiver |
| `vertical_rate` | float | Climb/descent rate |

## Flux Queries

### Current Aircraft Count

```flux
from(bucket: "skyspy")
  |> range(start: -5m)
  |> filter(fn: (r) => r._measurement == "aircraft")
  |> filter(fn: (r) => r._field == "lat")
  |> distinct(column: "hex")
  |> count()
```

### Aircraft Positions in Last Hour

```flux
from(bucket: "skyspy")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "aircraft")
  |> filter(fn: (r) => r._field == "lat" or r._field == "lon")
  |> pivot(rowKey: ["_time", "hex"], columnKey: ["_field"], valueColumn: "_value")
```

### Military Aircraft History

```flux
from(bucket: "skyspy")
  |> range(start: -24h)
  |> filter(fn: (r) => r._measurement == "aircraft")
  |> filter(fn: (r) => r.military == "true")
  |> filter(fn: (r) => r._field == "altitude")
  |> aggregateWindow(every: 1m, fn: mean)
```

### Unique Aircraft Per Hour

```flux
from(bucket: "skyspy")
  |> range(start: -24h)
  |> filter(fn: (r) => r._measurement == "aircraft")
  |> filter(fn: (r) => r._field == "lat")
  |> aggregateWindow(every: 1h, fn: (tables=<-, column) =>
      tables |> distinct(column: "hex") |> count()
  )
```

### Top Aircraft Types

```flux
from(bucket: "skyspy")
  |> range(start: -24h)
  |> filter(fn: (r) => r._measurement == "aircraft")
  |> filter(fn: (r) => r._field == "lat")
  |> distinct(column: "hex")
  |> group(columns: ["type"])
  |> count()
  |> sort(columns: ["_value"], desc: true)
  |> limit(n: 10)
```

## Retention Policies

```shell
# Create bucket with 30-day retention
influx bucket create \
  --name skyspy-archive \
  --org skyspy \
  --retention 30d

# Downsample old data
influx task create --file downsample.flux
```

```flux
// downsample.flux - Run hourly
option task = {name: "Downsample Aircraft", every: 1h}

from(bucket: "skyspy")
  |> range(start: -2h, stop: -1h)
  |> filter(fn: (r) => r._measurement == "aircraft")
  |> aggregateWindow(every: 5m, fn: mean)
  |> to(bucket: "skyspy-archive")
```

## Testing & Verification

1. **Start InfluxDB** and create bucket
2. **Run the logger** and verify connection
3. **Check data in InfluxDB UI**:
   ```
   http://localhost:8086 -> Data Explorer
   ```
4. **Run a test query**:
   ```shell
   influx query 'from(bucket:"skyspy") |> range(start:-5m) |> limit(n:10)'
   ```

## Troubleshooting

### No Data Being Written

**Problem:** Logger running but no points in InfluxDB
**Solution:**
- Verify token has write permissions
- Check bucket name matches
- Ensure aircraft have positions (lat/lon)
- Check for connection errors in output

### High Cardinality Warning

**Problem:** InfluxDB warns about high cardinality
**Solution:**
- Reduce number of tags
- Use hex as only required tag
- Move flight/type to fields if not filtering

### Slow Queries

**Problem:** Queries taking too long
**Solution:**
- Add time range filters
- Use appropriate retention policies
- Create continuous downsampling tasks
- Index frequently filtered tags

## Related Recipes

- [Grafana Dashboard](/docs/grafana-dashboard) - Visualize InfluxDB data
- [Prometheus Metrics](/docs/prometheus-metrics) - Alternative metrics
- [PostgreSQL Archive](/docs/postgresql-archive) - Relational storage
- [JSON Lines Logging](/docs/jsonl-logging) - File-based logging
