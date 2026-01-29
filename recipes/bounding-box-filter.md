---
title: Bounding Box Filter
description: Filter aircraft within a geographic bounding box.
hidden: false
recipe:
  color: '#10B981'
  icon: 📍
---
```shell Shell
curl "http://localhost:5000/api/v1/aircraft?bounds=33.9,-118.5,34.2,-118.1"
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type BoundingBox struct {
	MinLat, MaxLat float64
	MinLon, MaxLon float64
}

func (b BoundingBox) Contains(lat, lon float64) bool {
	return lat >= b.MinLat && lat <= b.MaxLat &&
		lon >= b.MinLon && lon <= b.MaxLon
}

func main() {
	lax := BoundingBox{
		MinLat: 33.9, MaxLat: 34.2,
		MinLon: -118.5, MaxLon: -118.1,
	}

	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct {
		Aircraft []struct {
			Hex string  `json:"hex"`
			Lat float64 `json:"lat"`
			Lon float64 `json:"lon"`
		} `json:"aircraft"`
	}
	json.NewDecoder(resp.Body).Decode(&data)

	for _, ac := range data.Aircraft {
		if lax.Contains(ac.Lat, ac.Lon) {
			fmt.Printf("Aircraft %s in LAX area\n", ac.Hex)
		}
	}
}
```

```python Python
import requests

SKYSPY_URL = "http://localhost:5000"

class BoundingBox:
    def __init__(self, min_lat, max_lat, min_lon, max_lon):
        self.min_lat = min_lat
        self.max_lat = max_lat
        self.min_lon = min_lon
        self.max_lon = max_lon

    def contains(self, lat, lon):
        return (self.min_lat <= lat <= self.max_lat and
                self.min_lon <= lon <= self.max_lon)

# LAX area bounding box
lax = BoundingBox(33.9, 34.2, -118.5, -118.1)

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

for ac in aircraft:
    if lax.contains(ac.get("lat", 0), ac.get("lon", 0)):
        print(f"Aircraft {ac['hex']} in LAX area")
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

class BoundingBox {
    constructor(minLat, maxLat, minLon, maxLon) {
        this.minLat = minLat;
        this.maxLat = maxLat;
        this.minLon = minLon;
        this.maxLon = maxLon;
    }

    contains(lat, lon) {
        return lat >= this.minLat && lat <= this.maxLat &&
               lon >= this.minLon && lon <= this.maxLon;
    }
}

// LAX area bounding box
const lax = new BoundingBox(33.9, 34.2, -118.5, -118.1);

async function filterAircraft() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    for (const ac of data.aircraft || []) {
        if (lax.contains(ac.lat, ac.lon)) {
            console.log(`Aircraft ${ac.hex} in LAX area`);
        }
    }
}

filterAircraft();
```

```json Response Example
{"aircraft": [{"hex": "A12345", "lat": 34.05, "lon": -118.25, "flight": "UAL123"}]}
```

# Define Bounding Box

<!-- go@10-17 -->
<!-- python@5-15 -->
<!-- javascript@3-15 -->

Create a bounding box with min/max latitude and longitude. The contains method checks if a point falls within the bounds.

# Set Your Area

<!-- shell@1 -->
<!-- go@19-23 -->
<!-- python@17-18 -->
<!-- javascript@17-18 -->

Define the corners of your monitoring area. This example uses the LAX airport region. Format: min_lat, min_lon, max_lat, max_lon.

# Filter Aircraft

<!-- go@25-38 -->
<!-- python@20-24 -->
<!-- javascript@20-30 -->

Fetch aircraft data and check each position against your bounding box. Only aircraft within the defined area will match.
