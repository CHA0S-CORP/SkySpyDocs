---
title: Grafana Dashboard
excerpt: Real-time aircraft metrics visualization with Grafana.
hidden: false
recipe:
  color: '#F46800'
  icon: 📈
difficulty: intermediate
tags: [data, visualization, grafana, metrics]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Grafana instance (self-hosted or Grafana Cloud)
- InfluxDB or Prometheus for data storage (this recipe uses InfluxDB)

## What You'll Build

A Grafana dashboard showing real-time aircraft metrics including counts, types, altitudes, and military activity over time.

```shell Shell
# Start InfluxDB and Grafana with Docker
docker run -d --name influxdb -p 8086:8086 influxdb:2.7
docker run -d --name grafana -p 3000:3000 grafana/grafana

# Initialize InfluxDB
influx setup --username admin --password adminpass --org skyspy --bucket aircraft --force

# Start the metrics exporter
pip install sseclient-py requests influxdb-client
python grafana_metrics.py
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

func main() {
    influxURL := os.Getenv("INFLUX_URL")
    if influxURL == "" {
        influxURL = "http://localhost:8086"
    }
    influxToken := os.Getenv("INFLUX_TOKEN")
    influxOrg := os.Getenv("INFLUX_ORG")
    influxBucket := os.Getenv("INFLUX_BUCKET")
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    client := influxdb2.NewClient(influxURL, influxToken)
    defer client.Close()
    writeAPI := client.WriteAPIBlocking(influxOrg, influxBucket)

    for {
        resp, _ := http.Get(skyspyURL + "/api/v1/map/sse")
        scanner := bufio.NewScanner(resp.Body)

        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data map[string]interface{}
                json.Unmarshal([]byte(line[5:]), &data)

                if aircraft, ok := data["aircraft"].([]interface{}); ok {
                    now := time.Now()

                    // Write summary metrics
                    total := len(aircraft)
                    military := 0
                    var totalAlt, totalSpeed float64

                    for _, a := range aircraft {
                        ac := a.(map[string]interface{})
                        if ac["military"] == true {
                            military++
                        }
                        if alt, ok := ac["alt"].(float64); ok {
                            totalAlt += alt
                        }
                        if gs, ok := ac["gs"].(float64); ok {
                            totalSpeed += gs
                        }
                    }

                    avgAlt := 0.0
                    avgSpeed := 0.0
                    if total > 0 {
                        avgAlt = totalAlt / float64(total)
                        avgSpeed = totalSpeed / float64(total)
                    }

                    point := influxdb2.NewPoint("aircraft_summary",
                        map[string]string{},
                        map[string]interface{}{
                            "total":         total,
                            "military":      military,
                            "avg_altitude":  avgAlt,
                            "avg_speed":     avgSpeed,
                        },
                        now,
                    )
                    writeAPI.WritePoint(context.Background(), point)
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
import sys
from datetime import datetime
from collections import Counter
import requests
import sseclient
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
INFLUX_URL = os.getenv("INFLUX_URL", "http://localhost:8086")
INFLUX_TOKEN = os.getenv("INFLUX_TOKEN")
INFLUX_ORG = os.getenv("INFLUX_ORG", "skyspy")
INFLUX_BUCKET = os.getenv("INFLUX_BUCKET", "aircraft")

if not INFLUX_TOKEN:
    print("Error: INFLUX_TOKEN required")
    sys.exit(1)

client = InfluxDBClient(url=INFLUX_URL, token=INFLUX_TOKEN, org=INFLUX_ORG)
write_api = client.write_api(write_options=SYNCHRONOUS)


def write_metrics(aircraft_list):
    """Write aircraft metrics to InfluxDB."""
    now = datetime.utcnow()

    # Summary metrics
    total = len(aircraft_list)
    military = sum(1 for a in aircraft_list if a.get("military"))
    altitudes = [a.get("alt", 0) for a in aircraft_list if a.get("alt")]
    speeds = [a.get("gs", 0) for a in aircraft_list if a.get("gs")]
    distances = [a.get("distance", 0) for a in aircraft_list if a.get("distance")]

    summary = Point("aircraft_summary") \
        .field("total", total) \
        .field("military", military) \
        .field("avg_altitude", sum(altitudes) / len(altitudes) if altitudes else 0) \
        .field("avg_speed", sum(speeds) / len(speeds) if speeds else 0) \
        .field("min_distance", min(distances) if distances else 0) \
        .time(now)

    write_api.write(bucket=INFLUX_BUCKET, record=summary)

    # Aircraft type distribution
    types = Counter(a.get("type", "Unknown") for a in aircraft_list)
    for aircraft_type, count in types.items():
        point = Point("aircraft_types") \
            .tag("type", aircraft_type) \
            .field("count", count) \
            .time(now)
        write_api.write(bucket=INFLUX_BUCKET, record=point)

    # Individual aircraft (for detailed tracking)
    for aircraft in aircraft_list:
        if aircraft.get("lat") and aircraft.get("lon"):
            point = Point("aircraft") \
                .tag("hex", aircraft.get("hex")) \
                .tag("type", aircraft.get("type", "Unknown")) \
                .tag("military", str(aircraft.get("military", False))) \
                .field("altitude", aircraft.get("alt", 0)) \
                .field("speed", aircraft.get("gs", 0)) \
                .field("distance", aircraft.get("distance", 0)) \
                .field("lat", aircraft.get("lat")) \
                .field("lon", aircraft.get("lon")) \
                .time(now)
            write_api.write(bucket=INFLUX_BUCKET, record=point)

    print(f"Wrote metrics: {total} aircraft, {military} military")


def main():
    print(f"Grafana metrics exporter connected to {SKYSPY_URL}...")
    print(f"Writing to InfluxDB at {INFLUX_URL}")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    aircraft_list = data.get("aircraft", [])
                    if aircraft_list:
                        write_metrics(aircraft_list)

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            import time
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const { InfluxDB, Point } = require('@influxdata/influxdb-client');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const INFLUX_URL = process.env.INFLUX_URL || 'http://localhost:8086';
const INFLUX_TOKEN = process.env.INFLUX_TOKEN;
const INFLUX_ORG = process.env.INFLUX_ORG || 'skyspy';
const INFLUX_BUCKET = process.env.INFLUX_BUCKET || 'aircraft';

if (!INFLUX_TOKEN) {
  console.error('Error: INFLUX_TOKEN required');
  process.exit(1);
}

const influx = new InfluxDB({ url: INFLUX_URL, token: INFLUX_TOKEN });
const writeApi = influx.getWriteApi(INFLUX_ORG, INFLUX_BUCKET);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);
  const aircraft = data.aircraft || [];

  const military = aircraft.filter(a => a.military).length;
  const altitudes = aircraft.map(a => a.alt || 0).filter(a => a > 0);
  const avgAlt = altitudes.length ? altitudes.reduce((a, b) => a + b) / altitudes.length : 0;

  const point = new Point('aircraft_summary')
    .intField('total', aircraft.length)
    .intField('military', military)
    .floatField('avg_altitude', avgAlt);

  writeApi.writePoint(point);
  console.log(`Wrote: ${aircraft.length} aircraft`);
});
```

```json Grafana Dashboard JSON
{
  "title": "SkySpy Aircraft Monitor",
  "panels": [
    {
      "title": "Aircraft Count",
      "type": "stat",
      "gridPos": {"x": 0, "y": 0, "w": 6, "h": 4},
      "targets": [{
        "query": "from(bucket: \"aircraft\") |> range(start: -5m) |> filter(fn: (r) => r._measurement == \"aircraft_summary\" and r._field == \"total\") |> last()"
      }]
    },
    {
      "title": "Military Aircraft",
      "type": "stat",
      "gridPos": {"x": 6, "y": 0, "w": 6, "h": 4},
      "targets": [{
        "query": "from(bucket: \"aircraft\") |> range(start: -5m) |> filter(fn: (r) => r._measurement == \"aircraft_summary\" and r._field == \"military\") |> last()"
      }]
    },
    {
      "title": "Aircraft Over Time",
      "type": "timeseries",
      "gridPos": {"x": 0, "y": 4, "w": 12, "h": 8},
      "targets": [{
        "query": "from(bucket: \"aircraft\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"aircraft_summary\" and r._field == \"total\")"
      }]
    }
  ]
}
```

## Set Up InfluxDB

<!-- shell@2-6 -->

Start InfluxDB with Docker and create a bucket for aircraft data. Get the API token from the InfluxDB UI or CLI.

## Write Time-Series Data

<!-- python@25-55 -->

The exporter writes three types of metrics:
- `aircraft_summary`: Total counts, averages
- `aircraft_types`: Distribution by type
- `aircraft`: Individual aircraft positions

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `INFLUX_URL` | InfluxDB URL | `http://localhost:8086` |
| `INFLUX_TOKEN` | InfluxDB API token | Required |
| `INFLUX_ORG` | InfluxDB organization | `skyspy` |
| `INFLUX_BUCKET` | InfluxDB bucket | `aircraft` |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Grafana Data Source

1. Go to Configuration → Data Sources
2. Add InfluxDB (Flux query language)
3. Set URL, organization, token, and bucket
4. Test connection

### Sample Queries

**Aircraft count over 24 hours:**
```flux
from(bucket: "aircraft")
  |> range(start: -24h)
  |> filter(fn: (r) => r._measurement == "aircraft_summary")
  |> filter(fn: (r) => r._field == "total")
  |> aggregateWindow(every: 5m, fn: mean)
```

**Military vs civilian ratio:**
```flux
from(bucket: "aircraft")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "aircraft_summary")
  |> filter(fn: (r) => r._field == "total" or r._field == "military")
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
```

**Top aircraft types:**
```flux
from(bucket: "aircraft")
  |> range(start: -24h)
  |> filter(fn: (r) => r._measurement == "aircraft_types")
  |> group(columns: ["type"])
  |> sum()
  |> sort(desc: true)
  |> limit(n: 10)
```

## Dashboard Panels

### Stat Panels
- Current aircraft count
- Military aircraft count
- Average altitude
- Closest aircraft distance

### Time Series
- Aircraft count over time
- Military activity trends
- Altitude distribution

### Pie Chart
- Aircraft types distribution
- Civil vs military ratio

### Geomap
- Aircraft positions (requires geo plugin)

## Alerts

Set up Grafana alerts for conditions like:

```yaml
# Alert when more than 5 military aircraft
- name: Military Activity High
  condition: military > 5
  for: 5m
  annotations:
    summary: High military aircraft activity detected
```

## Testing & Verification

1. **Verify InfluxDB** is receiving data:
   ```shell
   influx query 'from(bucket: "aircraft") |> range(start: -5m) |> limit(n: 5)'
   ```
2. **Check Grafana data source** connection
3. **Import dashboard** or create panels manually
4. **Verify live updates** in Grafana

## Troubleshooting

### No Data in Grafana

**Problem:** Panels show "No data"
**Solution:**
- Check InfluxDB has data (use CLI or UI)
- Verify bucket name matches
- Check time range in Grafana panel

### Write Errors

**Problem:** `write: unauthorized`
**Solution:**
- Verify token has write permissions
- Check token hasn't expired
- Regenerate token if needed

### High Memory Usage

**Problem:** InfluxDB consuming too much memory
**Solution:**
- Reduce retention period
- Aggregate data to longer intervals
- Use continuous queries

## Related Recipes

- [InfluxDB Logging](/docs/influxdb-logging) - Detailed time-series storage
- [Prometheus Metrics](/docs/prometheus-metrics) - Prometheus integration
- [Build a Live Dashboard](/docs/live-dashboard) - Custom dashboard
- [Export to CSV](/docs/export-csv) - File-based export
