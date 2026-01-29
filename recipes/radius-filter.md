---
title: Radius Filter
description: Filter aircraft within a radius of a point.
hidden: false
recipe:
  color: '#8B5CF6'
  icon: 🎯
---
```shell Shell
curl "http://localhost:5000/api/v1/aircraft?lat=34.0522&lon=-118.2437&radius=50"
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"math"
	"net/http"
)

func haversine(lat1, lon1, lat2, lon2 float64) float64 {
	const R = 3959 // Earth radius in miles
	dLat := (lat2 - lat1) * math.Pi / 180
	dLon := (lon2 - lon1) * math.Pi / 180
	a := math.Sin(dLat/2)*math.Sin(dLat/2) +
		math.Cos(lat1*math.Pi/180)*math.Cos(lat2*math.Pi/180)*
			math.Sin(dLon/2)*math.Sin(dLon/2)
	c := 2 * math.Atan2(math.Sqrt(a), math.Sqrt(1-a))
	return R * c
}

func main() {
	centerLat, centerLon := 34.0522, -118.2437 // Los Angeles
	radiusMiles := 50.0

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
		dist := haversine(centerLat, centerLon, ac.Lat, ac.Lon)
		if dist <= radiusMiles {
			fmt.Printf("Aircraft %s is %.1f miles away\n", ac.Hex, dist)
		}
	}
}
```

```python Python
import math
import requests

SKYSPY_URL = "http://localhost:5000"

def haversine(lat1, lon1, lat2, lon2):
    R = 3959  # Earth radius in miles
    dlat = math.radians(lat2 - lat1)
    dlon = math.radians(lon2 - lon1)
    a = (math.sin(dlat / 2) ** 2 +
         math.cos(math.radians(lat1)) * math.cos(math.radians(lat2)) *
         math.sin(dlon / 2) ** 2)
    return R * 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))

# Center point (Los Angeles) and radius
center_lat, center_lon = 34.0522, -118.2437
radius_miles = 50

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

for ac in aircraft:
    dist = haversine(center_lat, center_lon, ac.get("lat", 0), ac.get("lon", 0))
    if dist <= radius_miles:
        print(f"Aircraft {ac['hex']} is {dist:.1f} miles away")
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

function haversine(lat1, lon1, lat2, lon2) {
    const R = 3959; // Earth radius in miles
    const dLat = (lat2 - lat1) * Math.PI / 180;
    const dLon = (lon2 - lon1) * Math.PI / 180;
    const a = Math.sin(dLat / 2) ** 2 +
              Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
              Math.sin(dLon / 2) ** 2;
    return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
}

// Center point (Los Angeles) and radius
const centerLat = 34.0522, centerLon = -118.2437;
const radiusMiles = 50;

async function filterByRadius() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    for (const ac of data.aircraft || []) {
        const dist = haversine(centerLat, centerLon, ac.lat, ac.lon);
        if (dist <= radiusMiles) {
            console.log(`Aircraft ${ac.hex} is ${dist.toFixed(1)} miles away`);
        }
    }
}

filterByRadius();
```

```json Response Example
{"aircraft": [{"hex": "A12345", "lat": 34.05, "lon": -118.25, "distance": 2.3}]}
```

# Haversine Distance Function

<!-- go@10-18 -->
<!-- python@6-13 -->
<!-- javascript@3-10 -->

Calculate the great-circle distance between two points on Earth using the Haversine formula. Returns distance in miles.

# Define Center and Radius

<!-- shell@1 -->
<!-- go@20-23 -->
<!-- python@15-17 -->
<!-- javascript@12-14 -->

Set your monitoring center point (latitude, longitude) and the radius in miles. This example monitors 50 miles around Los Angeles.

# Filter by Distance

<!-- go@25-38 -->
<!-- python@19-24 -->
<!-- javascript@16-27 -->

Calculate the distance from each aircraft to your center point. Only aircraft within the radius are included in results.
