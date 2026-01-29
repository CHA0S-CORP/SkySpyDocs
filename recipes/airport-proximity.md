---
title: Airport Proximity Monitor
excerpt: Track arrivals and departures near airports.
hidden: false
recipe:
  color: '#0EA5E9'
  icon: 🛬
difficulty: intermediate
tags: [geographic, airport, arrivals, departures]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Airport ICAO code or coordinates of the airport you want to monitor

## What You'll Build

A monitor that tracks aircraft near airports and classifies them as arrivals (descending) or departures (climbing) based on their vertical rate. Includes a built-in database of major airports with coordinates.

```shell Shell
pip install sseclient-py requests
export AIRPORT_CODE="KJFK"
export RADIUS_NM="15"
python airport_proximity.py
```

```go Go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "math"
    "net/http"
    "os"
    "strconv"
    "strings"
)

const earthRadiusNm = 3440.065

// Airport database with coordinates
var airports = map[string]struct {
    lat, lon float64
    name     string
}{
    "KJFK": {40.6413, -73.7781, "John F. Kennedy International"},
    "KLAX": {33.9425, -118.4081, "Los Angeles International"},
    "KORD": {41.9742, -87.9073, "Chicago O'Hare International"},
    "KATL": {33.6407, -84.4277, "Hartsfield-Jackson Atlanta"},
    "KDFW": {32.8998, -97.0403, "Dallas/Fort Worth International"},
    "EGLL": {51.4700, -0.4543, "London Heathrow"},
    "EHAM": {52.3086, 4.7639, "Amsterdam Schiphol"},
    "LFPG": {49.0097, 2.5479, "Paris Charles de Gaulle"},
    "EDDF": {50.0379, 8.5622, "Frankfurt am Main"},
    "RJTT": {35.5494, 139.7798, "Tokyo Haneda"},
}

var (
    airportLat float64
    airportLon float64
    radiusNm   float64
)

func main() {
    airportCode := os.Getenv("AIRPORT_CODE")
    if airportCode == "" {
        airportCode = "KJFK"
    }

    airport, ok := airports[airportCode]
    if !ok {
        fmt.Printf("Unknown airport: %s\n", airportCode)
        os.Exit(1)
    }

    airportLat = airport.lat
    airportLon = airport.lon

    radiusNm, _ = strconv.ParseFloat(os.Getenv("RADIUS_NM"), 64)
    if radiusNm == 0 {
        radiusNm = 15
    }

    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    fmt.Printf("Monitoring %s (%s)\n", airport.name, airportCode)
    fmt.Printf("Radius: %.1fnm around %.4f, %.4f\n", radiusNm, airportLat, airportLon)

    tracked := make(map[string]string)

    for {
        resp, _ := http.Get(skyspyURL + "/api/v1/map/sse")
        scanner := bufio.NewScanner(resp.Body)

        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data map[string]interface{}
                json.Unmarshal([]byte(line[5:]), &data)

                if aircraft, ok := data["aircraft"].([]interface{}); ok {
                    for _, a := range aircraft {
                        ac := a.(map[string]interface{})
                        hex := fmt.Sprint(ac["hex"])

                        lat, latOk := ac["lat"].(float64)
                        lon, lonOk := ac["lon"].(float64)

                        if !latOk || !lonOk {
                            continue
                        }

                        dist := haversine(airportLat, airportLon, lat, lon)
                        if dist > radiusNm {
                            continue
                        }

                        vertRate, _ := ac["baro_rate"].(float64)
                        status := classifyFlight(vertRate)

                        if tracked[hex] != status {
                            flight := fmt.Sprint(ac["flight"])
                            alt, _ := ac["alt"].(float64)
                            fmt.Printf("[%s] %s at %.1fnm, %0.fft (%.0f fpm)\n",
                                status, flight, dist, alt, vertRate)
                            tracked[hex] = status
                        }
                    }
                }
            }
        }
        resp.Body.Close()
    }
}

func classifyFlight(vertRate float64) string {
    if vertRate < -300 {
        return "ARRIVAL"
    } else if vertRate > 300 {
        return "DEPARTURE"
    }
    return "LEVEL"
}

func haversine(lat1, lon1, lat2, lon2 float64) float64 {
    dLat := (lat2 - lat1) * math.Pi / 180
    dLon := (lon2 - lon1) * math.Pi / 180
    lat1Rad := lat1 * math.Pi / 180
    lat2Rad := lat2 * math.Pi / 180

    a := math.Sin(dLat/2)*math.Sin(dLat/2) +
        math.Cos(lat1Rad)*math.Cos(lat2Rad)*math.Sin(dLon/2)*math.Sin(dLon/2)
    c := 2 * math.Atan2(math.Sqrt(a), math.Sqrt(1-a))
    return earthRadiusNm * c
}
```

```python Python
import json
import math
import os
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

# Airport database with coordinates
AIRPORTS = {
    "KJFK": {"lat": 40.6413, "lon": -73.7781, "name": "John F. Kennedy International"},
    "KLAX": {"lat": 33.9425, "lon": -118.4081, "name": "Los Angeles International"},
    "KORD": {"lat": 41.9742, "lon": -87.9073, "name": "Chicago O'Hare International"},
    "KATL": {"lat": 33.6407, "lon": -84.4277, "name": "Hartsfield-Jackson Atlanta"},
    "KDFW": {"lat": 32.8998, "lon": -97.0403, "name": "Dallas/Fort Worth International"},
    "EGLL": {"lat": 51.4700, "lon": -0.4543, "name": "London Heathrow"},
    "EHAM": {"lat": 52.3086, "lon": 4.7639, "name": "Amsterdam Schiphol"},
    "LFPG": {"lat": 49.0097, "lon": 2.5479, "name": "Paris Charles de Gaulle"},
    "EDDF": {"lat": 50.0379, "lon": 8.5622, "name": "Frankfurt am Main"},
    "RJTT": {"lat": 35.5494, "lon": 139.7798, "name": "Tokyo Haneda"},
}

# Configuration
AIRPORT_CODE = os.getenv("AIRPORT_CODE", "KJFK")
RADIUS_NM = float(os.getenv("RADIUS_NM", "15"))

# Vertical rate thresholds (feet per minute)
DESCENT_THRESHOLD = -300  # Descending = arrival
CLIMB_THRESHOLD = 300     # Climbing = departure

EARTH_RADIUS_NM = 3440.065

tracked = {}


def haversine_distance(lat1, lon1, lat2, lon2):
    """Calculate distance between two points using Haversine formula."""
    lat1_rad = math.radians(lat1)
    lat2_rad = math.radians(lat2)
    delta_lat = math.radians(lat2 - lat1)
    delta_lon = math.radians(lon2 - lon1)

    a = (math.sin(delta_lat / 2) ** 2 +
         math.cos(lat1_rad) * math.cos(lat2_rad) * math.sin(delta_lon / 2) ** 2)
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))

    return EARTH_RADIUS_NM * c


def classify_flight(vert_rate):
    """Classify aircraft as arrival, departure, or level flight."""
    if vert_rate is None:
        return "UNKNOWN"
    if vert_rate < DESCENT_THRESHOLD:
        return "ARRIVAL"
    elif vert_rate > CLIMB_THRESHOLD:
        return "DEPARTURE"
    return "LEVEL"


def process_aircraft(aircraft, airport_lat, airport_lon):
    """Process aircraft near the airport."""
    hex_code = aircraft.get("hex")
    lat = aircraft.get("lat")
    lon = aircraft.get("lon")

    if lat is None or lon is None:
        return

    distance = haversine_distance(airport_lat, airport_lon, lat, lon)

    if distance > RADIUS_NM:
        return

    vert_rate = aircraft.get("baro_rate", 0)
    status = classify_flight(vert_rate)

    # Only report if status changed
    if tracked.get(hex_code) == status:
        return

    flight = aircraft.get("flight", hex_code)
    altitude = aircraft.get("alt", 0)
    ground_speed = aircraft.get("gs", 0)

    status_emoji = {
        "ARRIVAL": "v",
        "DEPARTURE": "^",
        "LEVEL": "-",
        "UNKNOWN": "?"
    }

    print(f"[{status_emoji[status]} {status:9}] {flight:8} | "
          f"{distance:5.1f}nm | {altitude:6,}ft | {vert_rate:+6.0f}fpm | {ground_speed:3.0f}kts")

    tracked[hex_code] = status


def main():
    if AIRPORT_CODE not in AIRPORTS:
        print(f"Unknown airport: {AIRPORT_CODE}")
        print(f"Available: {', '.join(AIRPORTS.keys())}")
        return

    airport = AIRPORTS[AIRPORT_CODE]
    print(f"Monitoring {airport['name']} ({AIRPORT_CODE})")
    print(f"Radius: {RADIUS_NM}nm around {airport['lat']}, {airport['lon']}")
    print("-" * 70)
    print(f"{'Status':<14} {'Flight':<8} | {'Dist':>5} | {'Alt':>8} | {'V/S':>8} | {'GS':>5}")
    print("-" * 70)

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    for aircraft in data.get("aircraft", []):
                        process_aircraft(aircraft, airport["lat"], airport["lon"])

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            import time
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const AIRPORT_CODE = process.env.AIRPORT_CODE || 'KJFK';
const RADIUS_NM = parseFloat(process.env.RADIUS_NM || '15');

const EARTH_RADIUS_NM = 3440.065;
const DESCENT_THRESHOLD = -300;
const CLIMB_THRESHOLD = 300;

// Airport database
const AIRPORTS = {
  KJFK: { lat: 40.6413, lon: -73.7781, name: 'John F. Kennedy International' },
  KLAX: { lat: 33.9425, lon: -118.4081, name: 'Los Angeles International' },
  KORD: { lat: 41.9742, lon: -87.9073, name: 'Chicago O\'Hare International' },
  KATL: { lat: 33.6407, lon: -84.4277, name: 'Hartsfield-Jackson Atlanta' },
  KDFW: { lat: 32.8998, lon: -97.0403, name: 'Dallas/Fort Worth International' },
  EGLL: { lat: 51.4700, lon: -0.4543, name: 'London Heathrow' },
  EHAM: { lat: 52.3086, lon: 4.7639, name: 'Amsterdam Schiphol' },
  LFPG: { lat: 49.0097, lon: 2.5479, name: 'Paris Charles de Gaulle' },
  EDDF: { lat: 50.0379, lon: 8.5622, name: 'Frankfurt am Main' },
  RJTT: { lat: 35.5494, lon: 139.7798, name: 'Tokyo Haneda' },
};

const tracked = new Map();

function haversine(lat1, lon1, lat2, lon2) {
  const toRad = x => x * Math.PI / 180;
  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);

  const a = Math.sin(dLat / 2) ** 2 +
            Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) * Math.sin(dLon / 2) ** 2;
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return EARTH_RADIUS_NM * c;
}

function classifyFlight(vertRate) {
  if (vertRate == null) return 'UNKNOWN';
  if (vertRate < DESCENT_THRESHOLD) return 'ARRIVAL';
  if (vertRate > CLIMB_THRESHOLD) return 'DEPARTURE';
  return 'LEVEL';
}

const airport = AIRPORTS[AIRPORT_CODE];
if (!airport) {
  console.error(`Unknown airport: ${AIRPORT_CODE}`);
  console.error(`Available: ${Object.keys(AIRPORTS).join(', ')}`);
  process.exit(1);
}

console.log(`Monitoring ${airport.name} (${AIRPORT_CODE})`);
console.log(`Radius: ${RADIUS_NM}nm around ${airport.lat}, ${airport.lon}`);
console.log('-'.repeat(70));

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const { lat, lon, hex } = aircraft;
    if (lat == null || lon == null) continue;

    const distance = haversine(airport.lat, airport.lon, lat, lon);
    if (distance > RADIUS_NM) continue;

    const vertRate = aircraft.baro_rate || 0;
    const status = classifyFlight(vertRate);

    if (tracked.get(hex) === status) continue;

    const flight = aircraft.flight || hex;
    const alt = aircraft.alt || 0;
    const indicator = { ARRIVAL: 'v', DEPARTURE: '^', LEVEL: '-', UNKNOWN: '?' }[status];

    console.log(`[${indicator} ${status.padEnd(9)}] ${flight.padEnd(8)} | ` +
                `${distance.toFixed(1).padStart(5)}nm | ${alt.toLocaleString().padStart(7)}ft | ` +
                `${(vertRate >= 0 ? '+' : '') + vertRate.toFixed(0).padStart(5)}fpm`);

    tracked.set(hex, status);
  }
});

es.onerror = (err) => {
  console.error('Connection error, reconnecting...');
};
```

```json Response Example
{
  "hex": "A12345",
  "flight": "UAL456",
  "lat": 40.6523,
  "lon": -73.7891,
  "alt": 3500,
  "baro_rate": -1200,
  "gs": 180,
  "status": "ARRIVAL",
  "distance_nm": 2.3,
  "airport": "KJFK"
}
```

## Airport Database

<!-- python@10-21 -->

The built-in airport database includes major international airports with their ICAO codes and coordinates. You can easily extend this database with additional airports.

### Adding Custom Airports

```python
# Add your local airport
AIRPORTS["KSFO"] = {
    "lat": 37.6213,
    "lon": -122.3790,
    "name": "San Francisco International"
}
```

## Flight Classification

<!-- python@42-50 -->

Aircraft are classified based on their vertical rate:

| Vertical Rate | Classification | Meaning |
|---------------|----------------|---------|
| < -300 fpm | ARRIVAL | Aircraft descending toward airport |
| > +300 fpm | DEPARTURE | Aircraft climbing away from airport |
| -300 to +300 fpm | LEVEL | Level flight or taxiing |

The 300 fpm threshold filters out minor altitude variations while capturing actual climb/descent phases.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `AIRPORT_CODE` | ICAO airport code | `KJFK` |
| `RADIUS_NM` | Monitoring radius in nautical miles | `15` |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Recommended Radius by Airport Size

| Airport Type | Recommended Radius |
|--------------|-------------------|
| Major hub | 15-20 nm |
| Regional | 10-15 nm |
| Small/GA | 5-10 nm |

## Testing & Verification

1. **Choose a busy airport** - Major hubs will have more traffic
2. **Start monitoring** - Run the script during peak hours
3. **Verify classifications** - Cross-reference with FlightAware or similar
4. **Adjust thresholds** - Modify descent/climb thresholds if needed

### Sample Output

```
Monitoring John F. Kennedy International (KJFK)
Radius: 15.0nm around 40.6413, -73.7781
----------------------------------------------------------------------
Status         Flight   |  Dist |      Alt |      V/S |    GS
----------------------------------------------------------------------
[v ARRIVAL   ] UAL456   |   8.3nm |   5,200ft |  -1,200fpm | 210kts
[^ DEPARTURE ] DAL789   |  12.1nm |   8,500ft |  +2,400fpm | 280kts
[v ARRIVAL   ] AAL123   |   3.2nm |   2,100ft |  -1,800fpm | 160kts
```

## Troubleshooting

### Aircraft Not Classified Correctly

**Problem:** Aircraft showing as LEVEL when they should be ARRIVAL/DEPARTURE
**Solution:**
- Adjust the vertical rate thresholds (try -200/+200 fpm for more sensitivity)
- Some aircraft may not report vertical rate - check for null values

### Missing Aircraft Near Airport

**Problem:** Known traffic not appearing in output
**Solution:**
- Increase radius to capture aircraft further out
- Verify SkySpy is receiving ADS-B data
- Check that aircraft have valid position data

### Wrong Airport Coordinates

**Problem:** Monitoring center seems off
**Solution:**
- Verify ICAO code is correct (K prefix for US airports)
- Add custom coordinates if airport not in database
- Double-check longitude sign (Western = negative)

## Related Recipes

- [Radius Filter](/docs/radius-filter) - General proximity monitoring
- [Altitude Filter](/docs/altitude-filter) - Vertical filtering
- [Track Aircraft](/docs/track-aircraft) - Track specific flights
- [Geofence Alerts](/docs/geofence-alerts) - Enter/exit zones
