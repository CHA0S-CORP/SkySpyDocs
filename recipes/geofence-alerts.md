---
title: Geofence Alerts
excerpt: Enter/exit zone notifications for aircraft.
hidden: false
recipe:
  color: '#8B5CF6'
  icon: "\U0001F532"
difficulty: intermediate
tags: [geographic, geofence, alerts, filtering]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Zone coordinates (polygon vertices or circle center + radius)
- Basic understanding of geographic coordinate systems

## What You'll Build

A geofence monitoring system that tracks aircraft entering and exiting defined zones. Supports both polygon zones (for complex shapes like airspace boundaries) and circular zones (for simple radius-based monitoring). Get notified when aircraft cross zone boundaries in either direction.

```shell Shell
pip install sseclient-py requests
export SKYSPY_URL="http://localhost:5000"
python geofence_alerts.py
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
	"strings"
	"time"
)

const earthRadiusNm = 3440.065

// Zone types
type CircleZone struct {
	Name      string
	CenterLat float64
	CenterLon float64
	RadiusNm  float64
}

type PolygonZone struct {
	Name   string
	Points [][2]float64 // [lat, lon] pairs
}

// Define your zones
var circleZones = []CircleZone{
	{Name: "Airport", CenterLat: 40.6413, CenterLon: -73.7781, RadiusNm: 5},
	{Name: "Downtown", CenterLat: 40.7128, CenterLon: -74.0060, RadiusNm: 3},
}

var polygonZones = []PolygonZone{
	{
		Name: "Restricted Airspace",
		Points: [][2]float64{
			{40.75, -74.00}, {40.80, -73.95},
			{40.78, -73.90}, {40.72, -73.92},
		},
	},
}

// Track aircraft zone states
var aircraftZones = make(map[string]map[string]bool)

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	fmt.Printf("Geofence monitor connected to %s\n", skyspyURL)
	fmt.Printf("Monitoring %d circle zones, %d polygon zones\n",
		len(circleZones), len(polygonZones))

	for {
		resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
		if err != nil {
			fmt.Printf("Connection error: %v, retrying...\n", err)
			time.Sleep(5 * time.Second)
			continue
		}

		scanner := bufio.NewScanner(resp.Body)
		for scanner.Scan() {
			line := scanner.Text()
			if strings.HasPrefix(line, "data:") {
				var data map[string]interface{}
				json.Unmarshal([]byte(line[5:]), &data)

				if aircraft, ok := data["aircraft"].([]interface{}); ok {
					for _, a := range aircraft {
						ac := a.(map[string]interface{})
						processAircraft(ac)
					}
				}
			}
		}
		resp.Body.Close()
	}
}

func processAircraft(ac map[string]interface{}) {
	hex := fmt.Sprint(ac["hex"])
	lat, latOk := ac["lat"].(float64)
	lon, lonOk := ac["lon"].(float64)

	if !latOk || !lonOk {
		return
	}

	if aircraftZones[hex] == nil {
		aircraftZones[hex] = make(map[string]bool)
	}

	flight := fmt.Sprint(ac["flight"])
	if flight == "<nil>" {
		flight = hex
	}

	// Check circle zones
	for _, zone := range circleZones {
		inZone := haversine(zone.CenterLat, zone.CenterLon, lat, lon) <= zone.RadiusNm
		wasInZone := aircraftZones[hex][zone.Name]

		if inZone && !wasInZone {
			fmt.Printf("ENTER: %s entered %s\n", flight, zone.Name)
			aircraftZones[hex][zone.Name] = true
		} else if !inZone && wasInZone {
			fmt.Printf("EXIT: %s exited %s\n", flight, zone.Name)
			aircraftZones[hex][zone.Name] = false
		}
	}

	// Check polygon zones
	for _, zone := range polygonZones {
		inZone := pointInPolygon(lat, lon, zone.Points)
		wasInZone := aircraftZones[hex][zone.Name]

		if inZone && !wasInZone {
			fmt.Printf("ENTER: %s entered %s\n", flight, zone.Name)
			aircraftZones[hex][zone.Name] = true
		} else if !inZone && wasInZone {
			fmt.Printf("EXIT: %s exited %s\n", flight, zone.Name)
			aircraftZones[hex][zone.Name] = false
		}
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

func pointInPolygon(lat, lon float64, polygon [][2]float64) bool {
	n := len(polygon)
	inside := false

	j := n - 1
	for i := 0; i < n; i++ {
		yi, xi := polygon[i][0], polygon[i][1]
		yj, xj := polygon[j][0], polygon[j][1]

		if ((yi > lat) != (yj > lat)) &&
			(lon < (xj-xi)*(lat-yi)/(yj-yi)+xi) {
			inside = !inside
		}
		j = i
	}
	return inside
}
```

```python Python
import json
import math
import os
import time
from dataclasses import dataclass
from typing import Optional
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
EARTH_RADIUS_NM = 3440.065


@dataclass
class CircleZone:
    """A circular geofence zone."""
    name: str
    center_lat: float
    center_lon: float
    radius_nm: float


@dataclass
class PolygonZone:
    """A polygon geofence zone."""
    name: str
    points: list  # List of (lat, lon) tuples


# Define your zones
CIRCLE_ZONES = [
    CircleZone("JFK Airport", 40.6413, -73.7781, 5),
    CircleZone("Downtown NYC", 40.7128, -74.0060, 3),
    CircleZone("Newark Airport", 40.6895, -74.1745, 5),
]

POLYGON_ZONES = [
    PolygonZone("Restricted Airspace", [
        (40.75, -74.00),
        (40.80, -73.95),
        (40.78, -73.90),
        (40.72, -73.92),
    ]),
    PolygonZone("Hudson River Corridor", [
        (40.70, -74.02),
        (40.85, -73.97),
        (40.85, -73.95),
        (40.70, -74.00),
    ]),
]

# Track zone states per aircraft
aircraft_zones: dict[str, dict[str, bool]] = {}


def haversine_distance(lat1: float, lon1: float, lat2: float, lon2: float) -> float:
    """Calculate distance between two points using Haversine formula."""
    lat1_rad = math.radians(lat1)
    lat2_rad = math.radians(lat2)
    delta_lat = math.radians(lat2 - lat1)
    delta_lon = math.radians(lon2 - lon1)

    a = (math.sin(delta_lat / 2) ** 2 +
         math.cos(lat1_rad) * math.cos(lat2_rad) * math.sin(delta_lon / 2) ** 2)
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))

    return EARTH_RADIUS_NM * c


def point_in_polygon(lat: float, lon: float, polygon: list) -> bool:
    """
    Check if a point is inside a polygon using ray casting algorithm.
    polygon: list of (lat, lon) tuples defining vertices
    """
    n = len(polygon)
    inside = False

    j = n - 1
    for i in range(n):
        yi, xi = polygon[i]
        yj, xj = polygon[j]

        if ((yi > lat) != (yj > lat)) and \
           (lon < (xj - xi) * (lat - yi) / (yj - yi) + xi):
            inside = not inside
        j = i

    return inside


def check_circle_zones(hex_code: str, lat: float, lon: float, flight: str):
    """Check aircraft against all circle zones."""
    for zone in CIRCLE_ZONES:
        distance = haversine_distance(zone.center_lat, zone.center_lon, lat, lon)
        in_zone = distance <= zone.radius_nm
        was_in_zone = aircraft_zones.get(hex_code, {}).get(zone.name, False)

        if in_zone and not was_in_zone:
            print(f"ENTER: {flight} entered {zone.name} ({distance:.1f}nm from center)")
            aircraft_zones.setdefault(hex_code, {})[zone.name] = True

        elif not in_zone and was_in_zone:
            print(f"EXIT: {flight} exited {zone.name}")
            aircraft_zones[hex_code][zone.name] = False


def check_polygon_zones(hex_code: str, lat: float, lon: float, flight: str):
    """Check aircraft against all polygon zones."""
    for zone in POLYGON_ZONES:
        in_zone = point_in_polygon(lat, lon, zone.points)
        was_in_zone = aircraft_zones.get(hex_code, {}).get(zone.name, False)

        if in_zone and not was_in_zone:
            print(f"ENTER: {flight} entered {zone.name}")
            aircraft_zones.setdefault(hex_code, {})[zone.name] = True

        elif not in_zone and was_in_zone:
            print(f"EXIT: {flight} exited {zone.name}")
            aircraft_zones[hex_code][zone.name] = False


def process_aircraft(aircraft: dict):
    """Process a single aircraft for geofence crossings."""
    hex_code = aircraft.get("hex")
    lat = aircraft.get("lat")
    lon = aircraft.get("lon")

    if lat is None or lon is None:
        return

    flight = aircraft.get("flight", hex_code)

    check_circle_zones(hex_code, lat, lon, flight)
    check_polygon_zones(hex_code, lat, lon, flight)


def main():
    print(f"Geofence monitor connected to {SKYSPY_URL}")
    print(f"Monitoring {len(CIRCLE_ZONES)} circle zones, {len(POLYGON_ZONES)} polygon zones")
    print()

    while True:
        try:
            response = requests.get(
                f"{SKYSPY_URL}/api/v1/map/sse",
                stream=True,
                timeout=30
            )
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    for aircraft in data.get("aircraft", []):
                        process_aircraft(aircraft)

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const EARTH_RADIUS_NM = 3440.065;

// Define circle zones
const circleZones = [
  { name: 'JFK Airport', centerLat: 40.6413, centerLon: -73.7781, radiusNm: 5 },
  { name: 'Downtown NYC', centerLat: 40.7128, centerLon: -74.0060, radiusNm: 3 },
  { name: 'Newark Airport', centerLat: 40.6895, centerLon: -74.1745, radiusNm: 5 },
];

// Define polygon zones (array of [lat, lon] vertices)
const polygonZones = [
  {
    name: 'Restricted Airspace',
    points: [
      [40.75, -74.00], [40.80, -73.95],
      [40.78, -73.90], [40.72, -73.92],
    ],
  },
  {
    name: 'Hudson River Corridor',
    points: [
      [40.70, -74.02], [40.85, -73.97],
      [40.85, -73.95], [40.70, -74.00],
    ],
  },
];

// Track aircraft zone states: { hex: { zoneName: boolean } }
const aircraftZones = new Map();

function haversine(lat1, lon1, lat2, lon2) {
  const toRad = x => x * Math.PI / 180;
  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);

  const a = Math.sin(dLat / 2) ** 2 +
            Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) * Math.sin(dLon / 2) ** 2;
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return EARTH_RADIUS_NM * c;
}

function pointInPolygon(lat, lon, polygon) {
  const n = polygon.length;
  let inside = false;

  let j = n - 1;
  for (let i = 0; i < n; i++) {
    const [yi, xi] = polygon[i];
    const [yj, xj] = polygon[j];

    if (((yi > lat) !== (yj > lat)) &&
        (lon < (xj - xi) * (lat - yi) / (yj - yi) + xi)) {
      inside = !inside;
    }
    j = i;
  }
  return inside;
}

function processAircraft(aircraft) {
  const { hex, lat, lon } = aircraft;
  if (lat == null || lon == null) return;

  const flight = aircraft.flight || hex;

  if (!aircraftZones.has(hex)) {
    aircraftZones.set(hex, {});
  }
  const zones = aircraftZones.get(hex);

  // Check circle zones
  for (const zone of circleZones) {
    const distance = haversine(zone.centerLat, zone.centerLon, lat, lon);
    const inZone = distance <= zone.radiusNm;
    const wasInZone = zones[zone.name] || false;

    if (inZone && !wasInZone) {
      console.log(`ENTER: ${flight} entered ${zone.name} (${distance.toFixed(1)}nm from center)`);
      zones[zone.name] = true;
    } else if (!inZone && wasInZone) {
      console.log(`EXIT: ${flight} exited ${zone.name}`);
      zones[zone.name] = false;
    }
  }

  // Check polygon zones
  for (const zone of polygonZones) {
    const inZone = pointInPolygon(lat, lon, zone.points);
    const wasInZone = zones[zone.name] || false;

    if (inZone && !wasInZone) {
      console.log(`ENTER: ${flight} entered ${zone.name}`);
      zones[zone.name] = true;
    } else if (!inZone && wasInZone) {
      console.log(`EXIT: ${flight} exited ${zone.name}`);
      zones[zone.name] = false;
    }
  }
}

console.log(`Geofence monitor connected to ${SKYSPY_URL}`);
console.log(`Monitoring ${circleZones.length} circle zones, ${polygonZones.length} polygon zones`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);
  for (const aircraft of data.aircraft || []) {
    processAircraft(aircraft);
  }
});

es.addEventListener('error', (e) => {
  console.error('Connection error, reconnecting...');
});
```

```json Response Example
{
  "event": "zone_crossing",
  "type": "enter",
  "aircraft": {
    "hex": "A12345",
    "flight": "UAL123",
    "lat": 40.6500,
    "lon": -73.7800,
    "alt": 2500
  },
  "zone": {
    "name": "JFK Airport",
    "type": "circle",
    "distance_from_center": 1.2
  }
}
```

## Defining Geofence Zones

### Circle Zones

Circle zones are defined by a center point and radius. Best for:
- Airport monitoring
- Point-of-interest tracking
- Simple "around me" scenarios

<!-- python@20-25 -->

### Polygon Zones

Polygon zones are defined by a series of vertices forming a closed shape. Best for:
- Irregular airspace boundaries
- City/region boundaries
- Flight corridors

<!-- python@27-38 -->

## Point-in-Polygon Algorithm

<!-- python@65-82 -->

The ray casting algorithm determines if a point is inside a polygon by counting how many times a ray from the point crosses the polygon boundary. An odd count means inside, even means outside.

### How It Works

1. Cast a horizontal ray from the test point
2. Count intersections with polygon edges
3. Odd intersections = inside, even = outside

## Haversine Distance for Circles

<!-- python@52-62 -->

The Haversine formula calculates great-circle distance between two points, accounting for Earth's curvature. Essential for accurate radius-based geofencing.

## Multiple Zone Support

The system tracks each aircraft's state for every zone independently:

```python
# Track zone states per aircraft
aircraft_zones = {
    "A12345": {
        "JFK Airport": True,      # Currently inside
        "Downtown NYC": False,    # Currently outside
        "Restricted Airspace": True,
    },
    "B67890": {
        "JFK Airport": False,
        "Downtown NYC": True,
    }
}
```

This allows:
- Simultaneous monitoring of overlapping zones
- Correct enter/exit detection for each zone
- Memory-efficient per-aircraft tracking

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Zone Configuration

| Zone Property | Type | Description |
|---------------|------|-------------|
| `name` | string | Unique zone identifier |
| `center_lat` | float | Circle center latitude |
| `center_lon` | float | Circle center longitude |
| `radius_nm` | float | Circle radius in nautical miles |
| `points` | array | Polygon vertices as (lat, lon) pairs |

### Zone Definition Examples

```python
# Airport approach zone (circle)
CircleZone("LAX Approach", 33.9425, -118.4081, 10)

# State boundary (polygon)
PolygonZone("Nevada Airspace", [
    (42.0, -120.0),  # NW corner
    (42.0, -114.0),  # NE corner
    (35.0, -114.0),  # SE corner
    (35.0, -120.0),  # SW corner
])
```

### Creating Zones via API

```shell
# Create a circle zone alert rule
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "JFK Airport Zone",
    "enabled": true,
    "conditions": {
      "field": "distance_from_point",
      "operator": "lt",
      "value": 5,
      "params": {
        "lat": 40.6413,
        "lon": -73.7781
      }
    },
    "notification_enabled": true
  }'
```

## Testing & Verification

### 1. Verify Zone Definitions

```python
# Print zone boundaries
for zone in CIRCLE_ZONES:
    print(f"Circle: {zone.name}")
    print(f"  Center: {zone.center_lat}, {zone.center_lon}")
    print(f"  Radius: {zone.radius_nm}nm")

for zone in POLYGON_ZONES:
    print(f"Polygon: {zone.name}")
    print(f"  Vertices: {len(zone.points)}")
    for i, (lat, lon) in enumerate(zone.points):
        print(f"    {i+1}: {lat}, {lon}")
```

### 2. Test Point-in-Polygon

```python
# Test with known coordinates
test_point = (40.76, -73.97)  # Should be inside "Restricted Airspace"
result = point_in_polygon(*test_point, POLYGON_ZONES[0].points)
print(f"Point {test_point} in zone: {result}")
```

### 3. Test Circle Distance

```python
# Test distance calculation
test_lat, test_lon = 40.65, -73.78
for zone in CIRCLE_ZONES:
    dist = haversine_distance(zone.center_lat, zone.center_lon, test_lat, test_lon)
    in_zone = "INSIDE" if dist <= zone.radius_nm else "OUTSIDE"
    print(f"{zone.name}: {dist:.2f}nm - {in_zone}")
```

### 4. Monitor Output

Expected output when running:

```
Geofence monitor connected to http://localhost:5000
Monitoring 3 circle zones, 2 polygon zones

ENTER: UAL123 entered JFK Airport (2.3nm from center)
ENTER: DAL456 entered Downtown NYC (1.1nm from center)
EXIT: UAL123 exited JFK Airport
ENTER: AAL789 entered Restricted Airspace
EXIT: DAL456 exited Downtown NYC
```

## Troubleshooting

### No Enter/Exit Events

**Problem:** Aircraft are visible but no zone crossings detected
**Solution:**
- Verify zone coordinates are correct (lat/lon order)
- Check that zones cover areas with ADS-B coverage
- Ensure aircraft have valid position data
- Print zone definitions to verify they're loaded

### False Enter/Exit Events

**Problem:** Aircraft trigger multiple enter/exit events rapidly
**Solution:**
- Add hysteresis (buffer zone) to prevent boundary flickering:

```python
HYSTERESIS_NM = 0.1  # 0.1nm buffer

def check_with_hysteresis(distance, radius, was_inside):
    if was_inside:
        return distance <= radius + HYSTERESIS_NM
    else:
        return distance <= radius - HYSTERESIS_NM
```

### Polygon Zone Not Working

**Problem:** Aircraft inside polygon not detected
**Solution:**
- Ensure polygon vertices are in correct order (clockwise or counter-clockwise)
- Verify polygon is closed (algorithm handles this automatically)
- Check for self-intersecting polygons (not supported)
- Test with known coordinates inside the polygon

### Memory Growing Over Time

**Problem:** Script uses increasing memory
**Solution:**
- Implement cleanup for aircraft that haven't been seen recently:

```python
import time

CLEANUP_INTERVAL = 300  # 5 minutes
aircraft_last_seen = {}

def cleanup_stale_aircraft():
    now = time.time()
    stale = [hex for hex, ts in aircraft_last_seen.items()
             if now - ts > CLEANUP_INTERVAL]
    for hex in stale:
        del aircraft_zones[hex]
        del aircraft_last_seen[hex]
```

### Coordinate System Issues

**Problem:** Zones appear in wrong location
**Solution:**
- Western longitudes are negative (e.g., -74.0060 for NYC)
- Southern latitudes are negative (e.g., -33.8688 for Sydney)
- Verify coordinate format matches (decimal degrees, not DMS)

## Related Recipes

- [Bounding Box Filter](/docs/bounding-box-filter) - Rectangular area monitoring
- [Radius Filter](/docs/radius-filter) - Simple circular filter
- [Airport Proximity](/docs/airport-proximity) - Track arrivals/departures
- [Multi-Zone Monitoring](/docs/multi-zone-monitoring) - Advanced zone management
