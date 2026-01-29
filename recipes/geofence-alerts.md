---
title: Geofence Alerts
description: Get alerts when aircraft enter or exit defined zones.
hidden: false
recipe:
  color: '#059669'
  icon: 🗺️
---
```shell Shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "LAX Arrivals",
    "conditions": {
      "operator": "AND",
      "conditions": [
        {"field": "lat", "operator": "gte", "value": 33.9},
        {"field": "lat", "operator": "lte", "value": 34.0},
        {"field": "lon", "operator": "gte", "value": -118.5},
        {"field": "lon", "operator": "lte", "value": -118.3}
      ]
    }
  }'
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type Geofence struct {
	Name   string
	MinLat, MaxLat, MinLon, MaxLon float64
}

func (g Geofence) Contains(lat, lon float64) bool {
	return lat >= g.MinLat && lat <= g.MaxLat && lon >= g.MinLon && lon <= g.MaxLon
}

var zones = []Geofence{
	{"LAX Area", 33.9, 34.0, -118.5, -118.3},
	{"Downtown", 34.0, 34.1, -118.3, -118.2},
}

func main() {
	inside := make(map[string]map[string]bool)
	for _, z := range zones { inside[z.Name] = make(map[string]bool) }

	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct{ Aircraft []struct{ Hex string; Lat, Lon float64 } `json:"aircraft"` }
	json.NewDecoder(resp.Body).Decode(&data)

	for _, ac := range data.Aircraft {
		for _, zone := range zones {
			wasInside := inside[zone.Name][ac.Hex]
			isInside := zone.Contains(ac.Lat, ac.Lon)

			if isInside && !wasInside {
				fmt.Printf("ENTER: %s entered %s\n", ac.Hex, zone.Name)
			} else if !isInside && wasInside {
				fmt.Printf("EXIT: %s left %s\n", ac.Hex, zone.Name)
			}
			inside[zone.Name][ac.Hex] = isInside
		}
	}
}
```

```python Python
import requests

SKYSPY_URL = "http://localhost:5000"

class Geofence:
    def __init__(self, name, min_lat, max_lat, min_lon, max_lon):
        self.name = name
        self.min_lat, self.max_lat = min_lat, max_lat
        self.min_lon, self.max_lon = min_lon, max_lon

    def contains(self, lat, lon):
        return (self.min_lat <= lat <= self.max_lat and
                self.min_lon <= lon <= self.max_lon)

zones = [
    Geofence("LAX Area", 33.9, 34.0, -118.5, -118.3),
    Geofence("Downtown", 34.0, 34.1, -118.3, -118.2),
]

inside = {z.name: set() for z in zones}

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
for ac in response.json().get("aircraft", []):
    lat, lon, hex_code = ac.get("lat", 0), ac.get("lon", 0), ac["hex"]

    for zone in zones:
        was_inside = hex_code in inside[zone.name]
        is_inside = zone.contains(lat, lon)

        if is_inside and not was_inside:
            print(f"ENTER: {hex_code} entered {zone.name}")
        elif not is_inside and was_inside:
            print(f"EXIT: {hex_code} left {zone.name}")

        if is_inside:
            inside[zone.name].add(hex_code)
        else:
            inside[zone.name].discard(hex_code)
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

class Geofence {
    constructor(name, minLat, maxLat, minLon, maxLon) {
        this.name = name;
        this.minLat = minLat; this.maxLat = maxLat;
        this.minLon = minLon; this.maxLon = maxLon;
    }

    contains(lat, lon) {
        return lat >= this.minLat && lat <= this.maxLat &&
               lon >= this.minLon && lon <= this.maxLon;
    }
}

const zones = [
    new Geofence('LAX Area', 33.9, 34.0, -118.5, -118.3),
    new Geofence('Downtown', 34.0, 34.1, -118.3, -118.2),
];

const inside = Object.fromEntries(zones.map(z => [z.name, new Set()]));

async function checkGeofences() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    for (const ac of data.aircraft || []) {
        for (const zone of zones) {
            const wasInside = inside[zone.name].has(ac.hex);
            const isInside = zone.contains(ac.lat, ac.lon);

            if (isInside && !wasInside) console.log(`ENTER: ${ac.hex} entered ${zone.name}`);
            if (!isInside && wasInside) console.log(`EXIT: ${ac.hex} left ${zone.name}`);

            isInside ? inside[zone.name].add(ac.hex) : inside[zone.name].delete(ac.hex);
        }
    }
}

setInterval(checkGeofences, 5000);
```

```json Response Example
{"event": "enter", "aircraft": "A12345", "zone": "LAX Area", "timestamp": "2024-01-15T10:30:00Z"}
```

# Define Geofence Zones

<!-- shell@3-13 -->
<!-- go@10-20 -->
<!-- python@5-17 -->
<!-- javascript@3-17 -->

Create zones with name and bounding coordinates. Define multiple zones for different areas of interest.

# Track Zone State

<!-- go@22-23 -->
<!-- python@20 -->
<!-- javascript@19-20 -->

Maintain a map of which aircraft are currently inside each zone. This enables enter/exit detection.

# Detect Enter/Exit Events

<!-- go@29-39 -->
<!-- python@26-37 -->
<!-- javascript@27-35 -->

Compare current position with previous state. Trigger alerts when aircraft cross zone boundaries.
