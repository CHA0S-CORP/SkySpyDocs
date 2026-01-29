---
title: Radius Filter
excerpt: Alert on aircraft within X miles of a specific point.
hidden: false
recipe:
  color: '#3B82F6'
  icon: 🎯
difficulty: beginner
tags: [geographic, filtering, location, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Center coordinates (latitude/longitude) of your location

## What You'll Build

A circular geographic filter that alerts when aircraft are within a specified distance of a center point. More intuitive than bounding boxes for monitoring "around me" scenarios.

```shell Shell
pip install sseclient-py requests
export CENTER_LAT="40.7128"
export CENTER_LON="-74.0060"
export RADIUS_NM="25"
python radius_filter.py
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

// Haversine formula constants
const earthRadiusNm = 3440.065 // Earth radius in nautical miles

var (
    centerLat float64
    centerLon float64
    radiusNm  float64
)

func main() {
    centerLat, _ = strconv.ParseFloat(os.Getenv("CENTER_LAT"), 64)
    centerLon, _ = strconv.ParseFloat(os.Getenv("CENTER_LON"), 64)
    radiusNm, _ = strconv.ParseFloat(os.Getenv("RADIUS_NM"), 64)

    if radiusNm == 0 {
        radiusNm = 25 // Default 25nm
    }

    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    fmt.Printf("Monitoring %.1fnm radius around %.4f, %.4f\n", radiusNm, centerLat, centerLon)

    alerted := make(map[string]bool)

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

                        if alerted[hex] {
                            continue
                        }

                        lat, latOk := ac["lat"].(float64)
                        lon, lonOk := ac["lon"].(float64)

                        if latOk && lonOk {
                            dist := haversine(centerLat, centerLon, lat, lon)
                            if dist <= radiusNm {
                                fmt.Printf("Aircraft in range: %s at %.1fnm\n", ac["flight"], dist)
                                alerted[hex] = true
                            }
                        }
                    }
                }
            }
        }
        resp.Body.Close()
    }
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
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

# Center point (your location)
CENTER_LAT = float(os.getenv("CENTER_LAT", "40.7128"))
CENTER_LON = float(os.getenv("CENTER_LON", "-74.0060"))

# Radius in nautical miles
RADIUS_NM = float(os.getenv("RADIUS_NM", "25"))

# Earth radius in nautical miles
EARTH_RADIUS_NM = 3440.065

alerted = set()


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


def bearing_to(lat1, lon1, lat2, lon2):
    """Calculate bearing from point 1 to point 2."""
    lat1_rad = math.radians(lat1)
    lat2_rad = math.radians(lat2)
    delta_lon = math.radians(lon2 - lon1)

    x = math.sin(delta_lon) * math.cos(lat2_rad)
    y = (math.cos(lat1_rad) * math.sin(lat2_rad) -
         math.sin(lat1_rad) * math.cos(lat2_rad) * math.cos(delta_lon))

    bearing = math.degrees(math.atan2(x, y))
    return (bearing + 360) % 360


def compass_direction(bearing):
    """Convert bearing to compass direction."""
    directions = ["N", "NNE", "NE", "ENE", "E", "ESE", "SE", "SSE",
                  "S", "SSW", "SW", "WSW", "W", "WNW", "NW", "NNW"]
    index = round(bearing / 22.5) % 16
    return directions[index]


def process_aircraft(aircraft):
    """Process aircraft within radius."""
    hex_code = aircraft.get("hex")
    lat = aircraft.get("lat")
    lon = aircraft.get("lon")

    if lat is None or lon is None:
        return

    distance = haversine_distance(CENTER_LAT, CENTER_LON, lat, lon)

    if distance > RADIUS_NM:
        return

    if hex_code in alerted:
        return

    flight = aircraft.get("flight", hex_code)
    bearing = bearing_to(CENTER_LAT, CENTER_LON, lat, lon)
    direction = compass_direction(bearing)

    print(f"Aircraft in range: {flight}")
    print(f"  Distance: {distance:.1f} nm {direction}")
    print(f"  Type: {aircraft.get('type', 'Unknown')}")
    print(f"  Altitude: {aircraft.get('alt', 0):,} ft")
    print(f"  Speed: {aircraft.get('gs', 0)} kts")
    print()

    alerted.add(hex_code)


def main():
    print(f"Radius filter connected to {SKYSPY_URL}...")
    print(f"Monitoring {RADIUS_NM}nm radius around {CENTER_LAT}, {CENTER_LON}")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    for aircraft in data.get("aircraft", []):
                        process_aircraft(aircraft)

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
const CENTER_LAT = parseFloat(process.env.CENTER_LAT || '40.7128');
const CENTER_LON = parseFloat(process.env.CENTER_LON || '-74.0060');
const RADIUS_NM = parseFloat(process.env.RADIUS_NM || '25');

const EARTH_RADIUS_NM = 3440.065;
const alerted = new Set();

function haversine(lat1, lon1, lat2, lon2) {
  const toRad = x => x * Math.PI / 180;
  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);

  const a = Math.sin(dLat / 2) ** 2 +
            Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) * Math.sin(dLon / 2) ** 2;
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return EARTH_RADIUS_NM * c;
}

console.log(`Monitoring ${RADIUS_NM}nm radius around ${CENTER_LAT}, ${CENTER_LON}`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;

    const { lat, lon } = aircraft;
    if (lat == null || lon == null) continue;

    const distance = haversine(CENTER_LAT, CENTER_LON, lat, lon);

    if (distance <= RADIUS_NM) {
      console.log(`Aircraft: ${aircraft.flight || aircraft.hex} at ${distance.toFixed(1)}nm`);
      alerted.add(aircraft.hex);
    }
  }
});
```

```json Response Example
{
  "hex": "A12345",
  "flight": "UAL123",
  "distance": 12.4,
  "bearing": 270,
  "direction": "W",
  "in_range": true
}
```

## Haversine Formula

<!-- python@23-35 -->

The Haversine formula calculates the great-circle distance between two points on a sphere. This accounts for Earth's curvature, giving accurate distances.

## Add Bearing/Direction

<!-- python@38-53 -->

Calculate which direction aircraft are from your location using bearing (0-360°) converted to compass directions (N, NE, E, etc.).

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `CENTER_LAT` | Your latitude | `40.7128` |
| `CENTER_LON` | Your longitude | `-74.0060` |
| `RADIUS_NM` | Radius in nautical miles | `25` |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Unit Conversions

| Unit | Multiplier to nm |
|------|------------------|
| Nautical miles | 1 |
| Statute miles | 0.868976 |
| Kilometers | 0.539957 |

```python
# Convert km to nm
radius_km = 50
RADIUS_NM = radius_km * 0.539957

# Convert statute miles to nm
radius_mi = 30
RADIUS_NM = radius_mi * 0.868976
```

### Create Alert Rule

Use the SkySpy API with distance condition:

```shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Nearby Aircraft",
    "enabled": true,
    "conditions": {
      "operator": "AND",
      "conditions": [
        {"field": "distance", "operator": "lt", "value": 25}
      ]
    },
    "notification_enabled": true
  }'
```

**Note:** The `distance` field uses the receiver location configured in SkySpy.

### Distance Bands

Alert at different priorities for different distances:

```python
def get_priority(distance):
    if distance < 5:
        return "critical"  # Very close
    elif distance < 15:
        return "high"      # Nearby
    elif distance < 25:
        return "medium"    # In range
    else:
        return "low"       # Distant
```

## Finding Your Coordinates

### Google Maps

1. Right-click on your location
2. Click the coordinates to copy
3. Format: `lat, lon` (e.g., `40.7128, -74.0060`)

### Command Line (macOS)

```shell
# Get coordinates from macOS Location Services
osascript -e 'tell application "System Events" to get system attribute "com.apple.locationd.location"'
```

### IP-based (approximate)

```shell
curl -s ipinfo.io | jq '.loc'
```

## Testing & Verification

1. **Set center to your location**
2. **Start with large radius** (50nm) to verify data
3. **Reduce radius** to desired range
4. **Check bearing accuracy** with known aircraft

### Visual Test

```python
# Print a simple radar display
for aircraft in aircraft_list:
    dist = haversine_distance(CENTER_LAT, CENTER_LON, aircraft['lat'], aircraft['lon'])
    if dist <= RADIUS_NM:
        direction = compass_direction(bearing_to(CENTER_LAT, CENTER_LON, aircraft['lat'], aircraft['lon']))
        bar = '█' * int(dist / RADIUS_NM * 10)
        print(f"{direction:>3} {bar} {aircraft.get('flight', aircraft['hex'])}")
```

## Troubleshooting

### Distances Seem Wrong

**Problem:** Calculated distances don't match expectations
**Solution:**
- Verify lat/lon are in correct order
- Check for coordinate sign errors
- Confirm aircraft have valid position data

### No Aircraft in Range

**Problem:** Nothing matching despite visible aircraft
**Solution:**
- Increase radius temporarily
- Verify center coordinates are correct
- Check SkySpy is receiving data

### Aircraft Appearing at Wrong Locations

**Problem:** Aircraft bearings seem backwards
**Solution:**
- Check you're using `(YOUR_LAT, YOUR_LON, AIRCRAFT_LAT, AIRCRAFT_LON)` order
- Verify longitude sign (Western = negative)

## Related Recipes

- [Bounding Box Filter](/docs/bounding-box-filter) - Rectangular area
- [Geofence Alerts](/docs/geofence-alerts) - Enter/exit zones
- [Airport Proximity](/docs/airport-proximity) - Track arrivals
- [Altitude Bands](/docs/altitude-bands) - Vertical filtering
