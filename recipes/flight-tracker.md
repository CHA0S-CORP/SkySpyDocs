---
title: Flight Tracker
description: Track specific flights by callsign or registration.
hidden: false
recipe:
  color: '#0891B2'
  icon: 🔍
---
```shell Shell
curl -s "http://localhost:5000/api/v1/aircraft" | \
  jq '.aircraft[] | select(.flight == "UAL123" or .registration == "N12345")'
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"time"
)

var watchList = map[string]bool{
	"UAL123":  true,
	"N12345":  true,
	"RCH419":  true,
}

type Position struct {
	Lat, Lon float64
	Alt      int
	Time     time.Time
}

var tracks = make(map[string][]Position)

func main() {
	for {
		resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
		var data struct {
			Aircraft []struct {
				Hex, Flight, Registration string
				Lat, Lon                  float64
				Alt                       int
			} `json:"aircraft"`
		}
		json.NewDecoder(resp.Body).Decode(&data)
		resp.Body.Close()

		for _, ac := range data.Aircraft {
			if watchList[ac.Flight] || watchList[ac.Registration] {
				key := ac.Flight
				if key == "" { key = ac.Registration }
				tracks[key] = append(tracks[key], Position{ac.Lat, ac.Lon, ac.Alt, time.Now()})
				fmt.Printf("%s: %.4f, %.4f @ %d ft\n", key, ac.Lat, ac.Lon, ac.Alt)
			}
		}
		time.Sleep(5 * time.Second)
	}
}
```

```python Python
import time
import requests

SKYSPY_URL = "http://localhost:5000"

WATCH_LIST = {"UAL123", "N12345", "RCH419"}
tracks = {}

def track_flights():
    while True:
        response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
        aircraft = response.json().get("aircraft", [])

        for ac in aircraft:
            flight = ac.get("flight", "")
            reg = ac.get("registration", "")

            if flight in WATCH_LIST or reg in WATCH_LIST:
                key = flight or reg
                if key not in tracks:
                    tracks[key] = []

                tracks[key].append({
                    "lat": ac.get("lat"),
                    "lon": ac.get("lon"),
                    "alt": ac.get("alt"),
                    "time": time.time()
                })
                print(f"{key}: {ac.get('lat'):.4f}, {ac.get('lon'):.4f} @ {ac.get('alt'):,} ft")

        time.sleep(5)

track_flights()
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

const WATCH_LIST = new Set(['UAL123', 'N12345', 'RCH419']);
const tracks = {};

async function trackFlights() {
    while (true) {
        const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
        const data = await response.json();

        for (const ac of data.aircraft || []) {
            const flight = ac.flight || '';
            const reg = ac.registration || '';

            if (WATCH_LIST.has(flight) || WATCH_LIST.has(reg)) {
                const key = flight || reg;
                if (!tracks[key]) tracks[key] = [];

                tracks[key].push({
                    lat: ac.lat,
                    lon: ac.lon,
                    alt: ac.alt,
                    time: Date.now()
                });
                console.log(`${key}: ${ac.lat?.toFixed(4)}, ${ac.lon?.toFixed(4)} @ ${ac.alt?.toLocaleString()} ft`);
            }
        }

        await new Promise(r => setTimeout(r, 5000));
    }
}

trackFlights();
```

```json Response Example
{"flight": "UAL123", "positions": [{"lat": 34.05, "lon": -118.25, "alt": 35000, "time": "2024-01-15T10:30:00Z"}]}
```

# Define Watch List

<!-- go@10-14 -->
<!-- python@5 -->
<!-- javascript@3 -->

Create a set of callsigns or registration numbers to track. Add flights you want to follow.

# Record Positions

<!-- go@35-40 -->
<!-- python@18-28 -->
<!-- javascript@15-24 -->

When a watched flight is detected, record its position with timestamp. Build a track history over time.

# Continuous Monitoring

<!-- shell@1-2 -->
<!-- go@24-43 -->
<!-- python@10-32 -->
<!-- javascript@7-30 -->

Poll the API every few seconds to capture the flight path. Export tracks for visualization or analysis.
