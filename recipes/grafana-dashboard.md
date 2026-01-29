---
title: Grafana Dashboard
description: Visualize aircraft metrics with Grafana and InfluxDB.
hidden: false
recipe:
  color: '#F46800'
  icon: 📊
---
```shell Shell
influx write \
  --bucket skyspy \
  --precision s \
  'aircraft,type=military count=5 1704067200'
```

```go Go
package main

import (
	"context"
	"os"
	"time"

	influxdb2 "github.com/influxdata/influxdb-client-go/v2"
)

func main() {
	token := os.Getenv("INFLUX_TOKEN")
	url := os.Getenv("INFLUX_URL")
	org := os.Getenv("INFLUX_ORG")
	bucket := os.Getenv("INFLUX_BUCKET")

	client := influxdb2.NewClient(url, token)
	defer client.Close()

	writeAPI := client.WriteAPIBlocking(org, bucket)

	p := influxdb2.NewPoint(
		"aircraft",
		map[string]string{"type": "military"},
		map[string]interface{}{"count": 5, "avg_alt": 35000},
		time.Now(),
	)

	writeAPI.WritePoint(context.Background(), p)
}
```

```python Python
import os
from datetime import datetime
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS

INFLUX_URL = os.getenv("INFLUX_URL", "http://localhost:8086")
INFLUX_TOKEN = os.getenv("INFLUX_TOKEN")
INFLUX_ORG = os.getenv("INFLUX_ORG")
INFLUX_BUCKET = os.getenv("INFLUX_BUCKET", "skyspy")

client = InfluxDBClient(url=INFLUX_URL, token=INFLUX_TOKEN, org=INFLUX_ORG)
write_api = client.write_api(write_options=SYNCHRONOUS)

def record_aircraft_stats(aircraft_type, count, avg_altitude):
    point = (
        Point("aircraft")
        .tag("type", aircraft_type)
        .field("count", count)
        .field("avg_alt", avg_altitude)
        .time(datetime.utcnow())
    )
    write_api.write(bucket=INFLUX_BUCKET, record=point)

# Example usage
record_aircraft_stats("military", 5, 35000)
```

```javascript JavaScript
const { InfluxDB, Point } = require('@influxdata/influxdb-client');

const INFLUX_URL = process.env.INFLUX_URL || 'http://localhost:8086';
const INFLUX_TOKEN = process.env.INFLUX_TOKEN;
const INFLUX_ORG = process.env.INFLUX_ORG;
const INFLUX_BUCKET = process.env.INFLUX_BUCKET || 'skyspy';

const client = new InfluxDB({ url: INFLUX_URL, token: INFLUX_TOKEN });
const writeApi = client.getWriteApi(INFLUX_ORG, INFLUX_BUCKET, 's');

function recordAircraftStats(type, count, avgAltitude) {
    const point = new Point('aircraft')
        .tag('type', type)
        .intField('count', count)
        .intField('avg_alt', avgAltitude);
    writeApi.writePoint(point);
}

// Example usage
recordAircraftStats('military', 5, 35000);
writeApi.close();
```

```json Response Example
{"status": 204, "message": "Data written successfully"}
```

# Configure InfluxDB Connection

<!-- shell@1-2 -->
<!-- go@1-17 -->
<!-- python@1-12 -->
<!-- javascript@1-9 -->

Set up your InfluxDB credentials. Create a bucket named "skyspy" and generate an API token with write permissions.

# Create Data Point

<!-- go@19-26 -->
<!-- python@14-22 -->
<!-- javascript@11-16 -->

Structure the data as a point with tags (for filtering) and fields (for values). Tags like "type" allow grouping in Grafana queries.

# Write to InfluxDB

<!-- shell@3-4 -->
<!-- go@28-28 -->
<!-- python@23-24 -->
<!-- javascript@18-20 -->

Send the data point to InfluxDB. In Grafana, create a dashboard querying: `from(bucket: "skyspy") |> filter(fn: (r) => r._measurement == "aircraft")`
