---
title: Prometheus Metrics
excerpt: Expose aircraft metrics for Prometheus scraping.
hidden: false
recipe:
  color: '#E6522C'
  icon: 📊
difficulty: intermediate
tags: [data, monitoring, prometheus, metrics]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Prometheus server configured
- Python: `pip install sseclient-py requests prometheus_client`

## What You'll Build

A Prometheus metrics endpoint that exposes aircraft counts, types, and statistics for monitoring and alerting.

```shell Shell
pip install sseclient-py requests prometheus_client
python prometheus_exporter.py
# Metrics available at http://localhost:9100/metrics
```

```python Python
import json
import os
import sys
import threading
import time
from collections import Counter
import requests
import sseclient
from prometheus_client import start_http_server, Gauge, Counter as PromCounter, Summary

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
METRICS_PORT = int(os.getenv("METRICS_PORT", "9100"))

# Gauges (current values)
aircraft_total = Gauge("skyspy_aircraft_total", "Total aircraft currently tracked")
aircraft_military = Gauge("skyspy_aircraft_military", "Military aircraft currently tracked")
aircraft_closest = Gauge("skyspy_aircraft_closest_nm", "Distance to closest aircraft in nm")
aircraft_avg_altitude = Gauge("skyspy_aircraft_avg_altitude_ft", "Average altitude in feet")
aircraft_by_type = Gauge("skyspy_aircraft_by_type", "Aircraft count by type", ["type"])

# Counters (cumulative)
aircraft_seen = PromCounter("skyspy_aircraft_seen_total", "Total unique aircraft seen")
military_seen = PromCounter("skyspy_military_seen_total", "Total military aircraft seen")
emergency_seen = PromCounter("skyspy_emergency_total", "Emergency squawks seen", ["squawk"])

# Summary (distributions)
altitude_summary = Summary("skyspy_altitude", "Aircraft altitude distribution")
speed_summary = Summary("skyspy_speed", "Aircraft speed distribution")

aircraft = {}
seen_ever = set()


def update_metrics(aircraft_list):
    """Update Prometheus metrics from aircraft list."""
    global aircraft

    # Update current aircraft
    aircraft = {a["hex"]: a for a in aircraft_list}

    # Basic gauges
    aircraft_total.set(len(aircraft))

    military = [a for a in aircraft_list if a.get("military")]
    aircraft_military.set(len(military))

    distances = [a.get("distance", 999) for a in aircraft_list if a.get("distance")]
    if distances:
        aircraft_closest.set(min(distances))

    altitudes = [a.get("alt", 0) for a in aircraft_list if a.get("alt")]
    if altitudes:
        aircraft_avg_altitude.set(sum(altitudes) / len(altitudes))
        for alt in altitudes:
            altitude_summary.observe(alt)

    speeds = [a.get("gs", 0) for a in aircraft_list if a.get("gs")]
    for speed in speeds:
        speed_summary.observe(speed)

    # Type distribution
    types = Counter(a.get("type", "unknown") for a in aircraft_list)
    for aircraft_type, count in types.items():
        aircraft_by_type.labels(type=aircraft_type).set(count)

    # Cumulative counters
    for a in aircraft_list:
        hex_code = a.get("hex")
        if hex_code not in seen_ever:
            seen_ever.add(hex_code)
            aircraft_seen.inc()
            if a.get("military"):
                military_seen.inc()

        squawk = a.get("squawk", "")
        if squawk in ["7700", "7600", "7500"]:
            emergency_seen.labels(squawk=squawk).inc()


def stream_aircraft():
    """Stream aircraft updates."""
    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    aircraft_list = data.get("aircraft", [])
                    if aircraft_list:
                        update_metrics(aircraft_list)

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            time.sleep(5)


def main():
    # Start metrics server
    start_http_server(METRICS_PORT)
    print(f"Prometheus metrics at http://localhost:{METRICS_PORT}/metrics")

    # Start aircraft stream
    stream_aircraft()


if __name__ == "__main__":
    main()
```

```go Go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "strings"
    "sync"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    aircraftTotal = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "skyspy_aircraft_total",
        Help: "Total aircraft currently tracked",
    })
    aircraftMilitary = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "skyspy_aircraft_military",
        Help: "Military aircraft currently tracked",
    })
    aircraftByType = prometheus.NewGaugeVec(prometheus.GaugeOpts{
        Name: "skyspy_aircraft_by_type",
        Help: "Aircraft count by type",
    }, []string{"type"})
)

var mu sync.RWMutex
var aircraft = make(map[string]map[string]interface{})

func init() {
    prometheus.MustRegister(aircraftTotal)
    prometheus.MustRegister(aircraftMilitary)
    prometheus.MustRegister(aircraftByType)
}

func main() {
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    go streamAircraft(skyspyURL)

    http.Handle("/metrics", promhttp.Handler())
    fmt.Println("Prometheus metrics at http://localhost:9100/metrics")
    http.ListenAndServe(":9100", nil)
}

func streamAircraft(skyspyURL string) {
    for {
        resp, _ := http.Get(skyspyURL + "/api/v1/map/sse")
        scanner := bufio.NewScanner(resp.Body)

        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data map[string]interface{}
                json.Unmarshal([]byte(line[5:]), &data)

                if acList, ok := data["aircraft"].([]interface{}); ok {
                    mu.Lock()
                    aircraft = make(map[string]map[string]interface{})
                    military := 0
                    types := make(map[string]int)

                    for _, a := range acList {
                        ac := a.(map[string]interface{})
                        hex := fmt.Sprint(ac["hex"])
                        aircraft[hex] = ac

                        if ac["military"] == true {
                            military++
                        }
                        if t, ok := ac["type"].(string); ok {
                            types[t]++
                        }
                    }

                    aircraftTotal.Set(float64(len(aircraft)))
                    aircraftMilitary.Set(float64(military))
                    for t, count := range types {
                        aircraftByType.WithLabelValues(t).Set(float64(count))
                    }
                    mu.Unlock()
                }
            }
        }
        resp.Body.Close()
    }
}
```

```yaml Prometheus Config
# prometheus.yml
scrape_configs:
  - job_name: 'skyspy'
    static_configs:
      - targets: ['localhost:9100']
    scrape_interval: 15s
```

```text Metrics Output
# HELP skyspy_aircraft_total Total aircraft currently tracked
# TYPE skyspy_aircraft_total gauge
skyspy_aircraft_total 23

# HELP skyspy_aircraft_military Military aircraft currently tracked
# TYPE skyspy_aircraft_military gauge
skyspy_aircraft_military 2

# HELP skyspy_aircraft_by_type Aircraft count by type
# TYPE skyspy_aircraft_by_type gauge
skyspy_aircraft_by_type{type="B738"} 5
skyspy_aircraft_by_type{type="A321"} 3
skyspy_aircraft_by_type{type="C17"} 1

# HELP skyspy_aircraft_seen_total Total unique aircraft seen
# TYPE skyspy_aircraft_seen_total counter
skyspy_aircraft_seen_total 1547
```

## Metrics Types

| Metric | Type | Description |
|--------|------|-------------|
| `skyspy_aircraft_total` | Gauge | Current aircraft count |
| `skyspy_aircraft_military` | Gauge | Current military count |
| `skyspy_aircraft_closest_nm` | Gauge | Closest aircraft distance |
| `skyspy_aircraft_avg_altitude_ft` | Gauge | Average altitude |
| `skyspy_aircraft_by_type` | Gauge (labeled) | Count per type |
| `skyspy_aircraft_seen_total` | Counter | Cumulative unique aircraft |
| `skyspy_emergency_total` | Counter (labeled) | Emergency squawks |

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `METRICS_PORT` | Prometheus metrics port | `9100` |

### Alert Rules

```yaml
# rules.yml
groups:
  - name: skyspy
    rules:
      - alert: HighMilitaryActivity
        expr: skyspy_aircraft_military > 5
        for: 5m
        labels:
          severity: info
        annotations:
          summary: High military aircraft activity

      - alert: EmergencySquawk
        expr: increase(skyspy_emergency_total[1h]) > 0
        labels:
          severity: critical
        annotations:
          summary: Emergency squawk detected
```

### Grafana Queries

```promql
# Aircraft count over 24h
avg_over_time(skyspy_aircraft_total[24h])

# Military percentage
skyspy_aircraft_military / skyspy_aircraft_total * 100

# Top aircraft types
topk(5, skyspy_aircraft_by_type)

# New aircraft per hour
increase(skyspy_aircraft_seen_total[1h])
```

## Testing & Verification

1. **Start the exporter**
2. **Check metrics**:
   ```shell
   curl http://localhost:9100/metrics
   ```
3. **Configure Prometheus** to scrape the endpoint
4. **Verify in Prometheus UI** - targets should show as UP

## Troubleshooting

### No Data in Prometheus

**Problem:** Target is up but no skyspy metrics
**Solution:**
- Verify exporter is receiving aircraft data
- Check for errors in exporter output
- Ensure metrics port isn't blocked

### High Cardinality

**Problem:** Too many time series
**Solution:**
- Limit aircraft type labels to top N
- Aggregate less common types as "other"

## Related Recipes

- [Grafana Dashboard](/docs/grafana-dashboard) - Visualization
- [InfluxDB Logging](/docs/influxdb-logging) - Time-series storage
- [Home Assistant Integration](/docs/home-assistant) - Smart home
- [Terminal Dashboard](/docs/terminal-dashboard) - CLI display
