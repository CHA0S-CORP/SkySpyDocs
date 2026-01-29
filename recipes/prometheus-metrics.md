---
title: Prometheus Metrics
description: Expose aircraft metrics for Prometheus scraping.
hidden: false
recipe:
  color: '#E6522C'
  icon: 📈
---
```shell Shell
curl http://localhost:9090/metrics
```

```go Go
package main

import (
	"encoding/json"
	"net/http"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
	aircraftCount = prometheus.NewGaugeVec(
		prometheus.GaugeOpts{
			Name: "skyspy_aircraft_total",
			Help: "Number of aircraft currently tracked",
		},
		[]string{"type"},
	)
	militaryCount = prometheus.NewGauge(prometheus.GaugeOpts{
		Name: "skyspy_military_aircraft",
		Help: "Number of military aircraft",
	})
)

func init() {
	prometheus.MustRegister(aircraftCount, militaryCount)
}

func updateMetrics() {
	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct{ Aircraft []struct{ Military bool; Type string } `json:"aircraft"` }
	json.NewDecoder(resp.Body).Decode(&data)
	resp.Body.Close()

	counts := make(map[string]float64)
	military := 0.0
	for _, ac := range data.Aircraft {
		counts[ac.Type]++
		if ac.Military { military++ }
	}

	for t, c := range counts { aircraftCount.WithLabelValues(t).Set(c) }
	militaryCount.Set(military)
}

func main() {
	go func() { for { updateMetrics() } }()
	http.Handle("/metrics", promhttp.Handler())
	http.ListenAndServe(":9090", nil)
}
```

```python Python
from prometheus_client import Gauge, start_http_server
import requests
import time

aircraft_count = Gauge('skyspy_aircraft_total', 'Aircraft tracked', ['type'])
military_count = Gauge('skyspy_military_aircraft', 'Military aircraft count')

SKYSPY_URL = "http://localhost:5000"

def update_metrics():
    response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
    aircraft = response.json().get("aircraft", [])

    type_counts = {}
    military = 0
    for ac in aircraft:
        ac_type = ac.get("type", "unknown")
        type_counts[ac_type] = type_counts.get(ac_type, 0) + 1
        if ac.get("military"):
            military += 1

    for ac_type, count in type_counts.items():
        aircraft_count.labels(type=ac_type).set(count)
    military_count.set(military)

start_http_server(9090)
while True:
    update_metrics()
    time.sleep(5)
```

```javascript JavaScript
const { register, Gauge } = require('prom-client');
const http = require('http');

const SKYSPY_URL = 'http://localhost:5000';

const aircraftCount = new Gauge({
    name: 'skyspy_aircraft_total',
    help: 'Aircraft tracked',
    labelNames: ['type'],
});

const militaryCount = new Gauge({
    name: 'skyspy_military_aircraft',
    help: 'Military aircraft count',
});

async function updateMetrics() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    const typeCounts = {};
    let military = 0;
    for (const ac of data.aircraft || []) {
        const type = ac.type || 'unknown';
        typeCounts[type] = (typeCounts[type] || 0) + 1;
        if (ac.military) military++;
    }

    for (const [type, count] of Object.entries(typeCounts)) {
        aircraftCount.labels(type).set(count);
    }
    militaryCount.set(military);
}

setInterval(updateMetrics, 5000);

http.createServer(async (req, res) => {
    res.setHeader('Content-Type', register.contentType);
    res.end(await register.metrics());
}).listen(9090);
```

```json Response Example
skyspy_aircraft_total{type="B738"} 5
skyspy_military_aircraft 2
```

# Define Metrics

<!-- go@11-22 -->
<!-- python@4-5 -->
<!-- javascript@6-14 -->

Create Prometheus gauges for aircraft counts. Use labels to track counts by aircraft type.

# Collect from SkySpy

<!-- go@28-40 -->
<!-- python@9-22 -->
<!-- javascript@16-29 -->

Fetch aircraft data and aggregate by type. Count military aircraft separately for dedicated monitoring.

# Expose Metrics Endpoint

<!-- shell@1 -->
<!-- go@42-45 -->
<!-- python@24-27 -->
<!-- javascript@31-37 -->

Serve metrics on port 9090. Add `http://localhost:9090/metrics` as a scrape target in prometheus.yml.
