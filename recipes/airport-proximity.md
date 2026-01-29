---
title: Airport Proximity
description: Track arrivals and departures near airports.
hidden: false
recipe:
  color: '#0369A1'
  icon: 🛬
---
```shell Shell
curl -s "http://localhost:5000/api/v1/aircraft" | \
  jq '.aircraft[] | select(.alt < 10000) | select(.lat > 33.9 and .lat < 34.0)'
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"math"
	"net/http"
)

type Airport struct {
	Name     string
	Lat, Lon float64
	Radius   float64 // miles
}

var airports = []Airport{
	{"LAX", 33.9425, -118.4081, 15},
	{"SFO", 37.6213, -122.3790, 15},
	{"JFK", 40.6413, -73.7781, 15},
}

func haversine(lat1, lon1, lat2, lon2 float64) float64 {
	const R = 3959
	dLat := (lat2 - lat1) * math.Pi / 180
	dLon := (lon2 - lon1) * math.Pi / 180
	a := math.Sin(dLat/2)*math.Sin(dLat/2) +
		math.Cos(lat1*math.Pi/180)*math.Cos(lat2*math.Pi/180)*math.Sin(dLon/2)*math.Sin(dLon/2)
	return R * 2 * math.Atan2(math.Sqrt(a), math.Sqrt(1-a))
}

func main() {
	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct {
		Aircraft []struct {
			Hex, Flight string
			Alt         int
			Lat, Lon    float64
			Baro        int `json:"baro_rate"`
		} `json:"aircraft"`
	}
	json.NewDecoder(resp.Body).Decode(&data)

	for _, airport := range airports {
		fmt.Printf("\n%s:\n", airport.Name)
		for _, ac := range data.Aircraft {
			dist := haversine(airport.Lat, airport.Lon, ac.Lat, ac.Lon)
			if dist <= airport.Radius && ac.Alt < 15000 {
				status := "cruising"
				if ac.Baro < -500 { status = "ARRIVING" }
				if ac.Baro > 500 { status = "DEPARTING" }
				fmt.Printf("  %s - %d ft (%s)\n", ac.Flight, ac.Alt, status)
			}
		}
	}
}
```

```python Python
import math
import requests

SKYSPY_URL = "http://localhost:5000"

AIRPORTS = [
    {"name": "LAX", "lat": 33.9425, "lon": -118.4081, "radius": 15},
    {"name": "SFO", "lat": 37.6213, "lon": -122.3790, "radius": 15},
    {"name": "JFK", "lat": 40.6413, "lon": -73.7781, "radius": 15},
]

def haversine(lat1, lon1, lat2, lon2):
    R = 3959
    dlat = math.radians(lat2 - lat1)
    dlon = math.radians(lon2 - lon1)
    a = math.sin(dlat/2)**2 + math.cos(math.radians(lat1)) * math.cos(math.radians(lat2)) * math.sin(dlon/2)**2
    return R * 2 * math.atan2(math.sqrt(a), math.sqrt(1-a))

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

for airport in AIRPORTS:
    print(f"\n{airport['name']}:")
    for ac in aircraft:
        dist = haversine(airport["lat"], airport["lon"], ac.get("lat", 0), ac.get("lon", 0))
        if dist <= airport["radius"] and ac.get("alt", 99999) < 15000:
            baro_rate = ac.get("baro_rate", 0)
            status = "ARRIVING" if baro_rate < -500 else "DEPARTING" if baro_rate > 500 else "cruising"
            print(f"  {ac.get('flight', ac['hex'])} - {ac.get('alt', 0):,} ft ({status})")
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

const AIRPORTS = [
    { name: 'LAX', lat: 33.9425, lon: -118.4081, radius: 15 },
    { name: 'SFO', lat: 37.6213, lon: -122.3790, radius: 15 },
    { name: 'JFK', lat: 40.6413, lon: -73.7781, radius: 15 },
];

function haversine(lat1, lon1, lat2, lon2) {
    const R = 3959;
    const dLat = (lat2 - lat1) * Math.PI / 180;
    const dLon = (lon2 - lon1) * Math.PI / 180;
    const a = Math.sin(dLat/2)**2 + Math.cos(lat1*Math.PI/180) * Math.cos(lat2*Math.PI/180) * Math.sin(dLon/2)**2;
    return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
}

async function trackAirportTraffic() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    for (const airport of AIRPORTS) {
        console.log(`\n${airport.name}:`);
        for (const ac of data.aircraft || []) {
            const dist = haversine(airport.lat, airport.lon, ac.lat, ac.lon);
            if (dist <= airport.radius && ac.alt < 15000) {
                const status = ac.baro_rate < -500 ? 'ARRIVING' : ac.baro_rate > 500 ? 'DEPARTING' : 'cruising';
                console.log(`  ${ac.flight || ac.hex} - ${ac.alt?.toLocaleString()} ft (${status})`);
            }
        }
    }
}

trackAirportTraffic();
```

```json Response Example
{"airport": "LAX", "arrivals": 12, "departures": 8, "in_pattern": 5}
```

# Define Airports

<!-- go@11-18 -->
<!-- python@5-9 -->
<!-- javascript@3-7 -->

List airports with coordinates and monitoring radius. Add any airports in your area.

# Calculate Distance

<!-- shell@1-2 -->
<!-- go@20-27 -->
<!-- python@11-16 -->
<!-- javascript@9-14 -->

Use haversine formula to find aircraft within range. Filter by altitude to focus on approaching/departing traffic.

# Detect Arrival/Departure

<!-- go@43-48 -->
<!-- python@25-28 -->
<!-- javascript@24-27 -->

Use baro_rate (vertical speed) to determine if aircraft is arriving (descending) or departing (climbing).
