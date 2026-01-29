---
title: Bounding Box Filter
excerpt: Alert on aircraft within specific latitude/longitude bounds.
hidden: false
recipe:
  color: '#10B981'
  icon: 📍
difficulty: beginner
tags: [geographic, filtering, location, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Coordinates for your area of interest

## What You'll Build

A geographic filter that alerts only when aircraft are within a defined rectangular area. Perfect for monitoring specific airports, cities, or regions.

```shell Shell
pip install sseclient-py requests
python bounding_box.py
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
)

// Define your bounding box
var (
    minLat = 40.4   // Southern boundary
    maxLat = 40.9   // Northern boundary
    minLon = -74.3  // Western boundary
    maxLon = -73.7  // Eastern boundary
)

func main() {
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

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

                        if latOk && lonOk && inBoundingBox(lat, lon) {
                            fmt.Printf("Aircraft in zone: %s (%s) at %.4f, %.4f\n",
                                ac["flight"], ac["type"], lat, lon)
                            alerted[hex] = true
                        }
                    }
                }
            }
        }
        resp.Body.Close()
    }
}

func inBoundingBox(lat, lon float64) bool {
    return lat >= minLat && lat <= maxLat && lon >= minLon && lon <= maxLon
}
```

```python Python
import json
import os
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

# Define your bounding box (example: NYC area)
BOUNDS = {
    "min_lat": float(os.getenv("MIN_LAT", "40.4")),   # Southern boundary
    "max_lat": float(os.getenv("MAX_LAT", "40.9")),   # Northern boundary
    "min_lon": float(os.getenv("MIN_LON", "-74.3")),  # Western boundary
    "max_lon": float(os.getenv("MAX_LON", "-73.7")),  # Eastern boundary
}

alerted = set()


def in_bounding_box(lat, lon):
    """Check if coordinates are within the bounding box."""
    return (
        BOUNDS["min_lat"] <= lat <= BOUNDS["max_lat"] and
        BOUNDS["min_lon"] <= lon <= BOUNDS["max_lon"]
    )


def get_zone_name(lat, lon):
    """Optional: Return zone name based on location."""
    # Subdivide the bounding box into named zones
    mid_lat = (BOUNDS["min_lat"] + BOUNDS["max_lat"]) / 2
    mid_lon = (BOUNDS["min_lon"] + BOUNDS["max_lon"]) / 2

    ns = "North" if lat > mid_lat else "South"
    ew = "East" if lon > mid_lon else "West"
    return f"{ns}{ew}"


def process_aircraft(aircraft):
    """Process aircraft in the bounding box."""
    hex_code = aircraft.get("hex")
    lat = aircraft.get("lat")
    lon = aircraft.get("lon")

    if lat is None or lon is None:
        return

    if not in_bounding_box(lat, lon):
        return

    if hex_code in alerted:
        return

    flight = aircraft.get("flight", hex_code)
    zone = get_zone_name(lat, lon)

    print(f"Aircraft in {zone}: {flight} ({aircraft.get('type', 'Unknown')})")
    print(f"  Position: {lat:.4f}, {lon:.4f}")
    print(f"  Altitude: {aircraft.get('alt', 0):,} ft")
    print(f"  Speed: {aircraft.get('gs', 0)} kts")
    print()

    alerted.add(hex_code)


def main():
    print(f"Bounding box filter connected to {SKYSPY_URL}...")
    print(f"Monitoring area: {BOUNDS['min_lat']},{BOUNDS['min_lon']} to {BOUNDS['max_lat']},{BOUNDS['max_lon']}")

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

// Define your bounding box
const BOUNDS = {
  minLat: parseFloat(process.env.MIN_LAT || '40.4'),
  maxLat: parseFloat(process.env.MAX_LAT || '40.9'),
  minLon: parseFloat(process.env.MIN_LON || '-74.3'),
  maxLon: parseFloat(process.env.MAX_LON || '-73.7')
};

const alerted = new Set();

function inBoundingBox(lat, lon) {
  return lat >= BOUNDS.minLat && lat <= BOUNDS.maxLat &&
         lon >= BOUNDS.minLon && lon <= BOUNDS.maxLon;
}

console.log(`Monitoring: ${BOUNDS.minLat},${BOUNDS.minLon} to ${BOUNDS.maxLat},${BOUNDS.maxLon}`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;

    const { lat, lon } = aircraft;
    if (lat == null || lon == null) continue;

    if (inBoundingBox(lat, lon)) {
      console.log(`Aircraft in zone: ${aircraft.flight || aircraft.hex} at ${lat.toFixed(4)}, ${lon.toFixed(4)}`);
      alerted.add(aircraft.hex);
    }
  }
});
```

```json Response Example
{
  "hex": "A12345",
  "flight": "UAL123",
  "lat": 40.7128,
  "lon": -74.0060,
  "alt": 3500,
  "in_zone": true,
  "zone_name": "SouthWest"
}
```

## Define Bounding Box

<!-- python@10-16 -->

Define four coordinates:
- `min_lat`: Southern boundary
- `max_lat`: Northern boundary
- `min_lon`: Western boundary
- `max_lon`: Eastern boundary

### Common Bounding Boxes

| Location | min_lat | max_lat | min_lon | max_lon |
|----------|---------|---------|---------|---------|
| NYC Area | 40.4 | 41.0 | -74.3 | -73.7 |
| LA Basin | 33.7 | 34.3 | -118.6 | -117.8 |
| London | 51.3 | 51.7 | -0.5 | 0.3 |
| Sydney | -34.1 | -33.6 | 150.9 | 151.4 |

## Check Coordinates

<!-- python@20-25 -->

The `in_bounding_box` function checks if aircraft coordinates fall within your defined area.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `MIN_LAT` | Southern boundary | `40.4` |
| `MAX_LAT` | Northern boundary | `40.9` |
| `MIN_LON` | Western boundary | `-74.3` |
| `MAX_LON` | Eastern boundary | `-73.7` |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Create Alert Rule

Use the SkySpy API to create a persistent bounding box rule:

```shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "NYC Area Monitor",
    "enabled": true,
    "conditions": {
      "operator": "AND",
      "conditions": [
        {"field": "lat", "operator": "gte", "value": 40.4},
        {"field": "lat", "operator": "lte", "value": 40.9},
        {"field": "lon", "operator": "gte", "value": -74.3},
        {"field": "lon", "operator": "lte", "value": -73.7}
      ]
    },
    "notification_enabled": true
  }'
```

### Sub-zones

Divide your area into named zones:

```python
ZONES = {
    "Airport": {"min_lat": 40.62, "max_lat": 40.66, "min_lon": -73.82, "max_lon": -73.76},
    "Manhattan": {"min_lat": 40.70, "max_lat": 40.82, "min_lon": -74.02, "max_lon": -73.93},
    "Brooklyn": {"min_lat": 40.57, "max_lat": 40.70, "min_lon": -74.04, "max_lon": -73.83},
}

def get_zone(lat, lon):
    for name, bounds in ZONES.items():
        if in_bounds(lat, lon, bounds):
            return name
    return "Unknown"
```

## Finding Coordinates

### Using Google Maps

1. Right-click on a location
2. Click coordinates to copy
3. First number is latitude, second is longitude

### Using bboxfinder.com

1. Go to [bboxfinder.com](http://bboxfinder.com)
2. Draw a rectangle on the map
3. Copy coordinates from "Box" field

## Testing & Verification

1. **Set bounds** for an area with frequent traffic
2. **Start the script** and verify connection
3. **Check output** for aircraft entering the zone
4. **Adjust bounds** if needed

### Visualize Your Box

```python
print(f"""
Bounding Box:
  NW: {BOUNDS['max_lat']}, {BOUNDS['min_lon']}  ----  NE: {BOUNDS['max_lat']}, {BOUNDS['max_lon']}
     |                                                    |
     |                                                    |
  SW: {BOUNDS['min_lat']}, {BOUNDS['min_lon']}  ----  SE: {BOUNDS['min_lat']}, {BOUNDS['max_lon']}
""")
```

## Troubleshooting

### No Aircraft Detected

**Problem:** Script runs but no aircraft match
**Solution:**
- Verify aircraft have lat/lon data (some don't)
- Check bounds are in correct format (lat before lon)
- Ensure bounds cover an area with ADS-B coverage

### Too Many Alerts

**Problem:** Overwhelming number of matches
**Solution:**
- Make the bounding box smaller
- Add altitude filters
- Combine with other conditions (military only, etc.)

### Wrong Hemisphere

**Problem:** Getting aircraft from opposite side of world
**Solution:**
- Check sign of coordinates
- Western longitudes are negative
- Southern latitudes are negative

## Related Recipes

- [Radius Filter](/docs/radius-filter) - Circular geographic filter
- [Geofence Alerts](/docs/geofence-alerts) - Enter/exit notifications
- [Airport Proximity](/docs/airport-proximity) - Track arrivals/departures
- [Geofence Alerts](/docs/geofence-alerts) - Multiple areas and enter/exit notifications
