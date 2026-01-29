---
title: Speed Filtering
excerpt: Detect fast or slow aircraft by ground speed.
hidden: false
recipe:
  color: '#F59E0B'
  icon: ⚡
difficulty: beginner
tags: [filtering, speed, geographic, monitoring]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Python: `pip install sseclient-py requests`

## What You'll Build

A filter that monitors aircraft by ground speed - useful for detecting supersonic military jets, slow-moving drones, stationary aircraft on the ground, or unusual speed patterns.

```shell Shell
pip install sseclient-py requests
export MIN_SPEED=0
export MAX_SPEED=100
python speed_filter.py
```

```python Python
import json
import os
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
MIN_SPEED = int(os.getenv("MIN_SPEED", "0"))
MAX_SPEED = int(os.getenv("MAX_SPEED", "100"))

alerted = set()


def check_speed(aircraft):
    """Check if aircraft is in the specified speed range."""
    speed = aircraft.get("gs")  # ground speed in knots
    if speed is None:
        return False, None

    in_range = MIN_SPEED <= speed <= MAX_SPEED
    return in_range, speed


def format_speed_category(speed):
    """Categorize speed for display."""
    if speed < 10:
        return "Stationary", "🛑"
    elif speed < 100:
        return "Slow", "🐢"
    elif speed < 250:
        return "Normal", "✈️"
    elif speed < 500:
        return "Fast", "🚀"
    else:
        return "Very Fast", "⚡"


def main():
    print(f"Speed Filter connected to {SKYSPY_URL}")
    print(f"Monitoring speed range: {MIN_SPEED:,} - {MAX_SPEED:,} kts")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)

                    for aircraft in data.get("aircraft", []):
                        hex_code = aircraft.get("hex")
                        in_range, speed = check_speed(aircraft)

                        if in_range and hex_code not in alerted:
                            flight = aircraft.get("flight", "Unknown").strip() or hex_code
                            aircraft_type = aircraft.get("type", "Unknown")
                            altitude = aircraft.get("alt", 0)
                            distance = aircraft.get("distance", 0)
                            category, emoji = format_speed_category(speed)

                            print(f"\n{emoji} Aircraft in speed range!")
                            print(f"   Flight: {flight}")
                            print(f"   Type: {aircraft_type}")
                            print(f"   Speed: {speed:,.0f} kts ({category})")
                            print(f"   Altitude: {altitude:,} ft")
                            print(f"   Distance: {distance:.1f} nm")

                            alerted.add(hex_code)

                        # Remove from alerted if left the range
                        elif not in_range and hex_code in alerted:
                            alerted.discard(hex_code)

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            import time
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```go Go
package main

import (
	"bufio"
	"encoding/json"
	"fmt"
	"net/http"
	"os"
	"strconv"
	"strings"
)

var alerted = make(map[string]bool)

func getEnvInt(key string, def int) int {
	if v := os.Getenv(key); v != "" {
		if i, err := strconv.Atoi(v); err == nil {
			return i
		}
	}
	return def
}

func formatCategory(speed float64) (string, string) {
	switch {
	case speed < 10:
		return "Stationary", "🛑"
	case speed < 100:
		return "Slow", "🐢"
	case speed < 250:
		return "Normal", "✈️"
	case speed < 500:
		return "Fast", "🚀"
	default:
		return "Very Fast", "⚡"
	}
}

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	minSpeed := float64(getEnvInt("MIN_SPEED", 0))
	maxSpeed := float64(getEnvInt("MAX_SPEED", 100))

	fmt.Printf("Speed Filter connected to %s\n", skyspyURL)
	fmt.Printf("Monitoring speed range: %.0f - %.0f kts\n", minSpeed, maxSpeed)

	for {
		resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
		if err != nil {
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
						hex := fmt.Sprint(ac["hex"])

						speed, hasSpeed := ac["gs"].(float64)
						if !hasSpeed {
							continue
						}

						inRange := speed >= minSpeed && speed <= maxSpeed

						if inRange && !alerted[hex] {
							flight := strings.TrimSpace(fmt.Sprint(ac["flight"]))
							if flight == "<nil>" {
								flight = hex
							}
							category, emoji := formatCategory(speed)

							fmt.Printf("\n%s Aircraft in speed range!\n", emoji)
							fmt.Printf("   Flight: %s\n", flight)
							fmt.Printf("   Type: %v\n", ac["type"])
							fmt.Printf("   Speed: %.0f kts (%s)\n", speed, category)
							fmt.Printf("   Altitude: %v ft\n", ac["alt"])
							fmt.Printf("   Distance: %.1f nm\n", ac["distance"])

							alerted[hex] = true
						} else if !inRange && alerted[hex] {
							delete(alerted, hex)
						}
					}
				}
			}
		}
		resp.Body.Close()
	}
}
```

```javascript JavaScript
const EventSource = require('eventsource');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const MIN_SPEED = parseInt(process.env.MIN_SPEED || '0');
const MAX_SPEED = parseInt(process.env.MAX_SPEED || '100');

const alerted = new Set();

function formatCategory(speed) {
  if (speed < 10) return ['Stationary', '🛑'];
  if (speed < 100) return ['Slow', '🐢'];
  if (speed < 250) return ['Normal', '✈️'];
  if (speed < 500) return ['Fast', '🚀'];
  return ['Very Fast', '⚡'];
}

console.log(`Speed Filter connected to ${SKYSPY_URL}`);
console.log(`Monitoring speed range: ${MIN_SPEED.toLocaleString()} - ${MAX_SPEED.toLocaleString()} kts`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;
    const speed = aircraft.gs;

    if (speed == null) continue;

    const inRange = speed >= MIN_SPEED && speed <= MAX_SPEED;

    if (inRange && !alerted.has(hex)) {
      const flight = (aircraft.flight || hex).trim();
      const [category, emoji] = formatCategory(speed);

      console.log(`\n${emoji} Aircraft in speed range!`);
      console.log(`   Flight: ${flight}`);
      console.log(`   Type: ${aircraft.type || 'Unknown'}`);
      console.log(`   Speed: ${speed.toLocaleString()} kts (${category})`);
      console.log(`   Altitude: ${(aircraft.alt || 0).toLocaleString()} ft`);
      console.log(`   Distance: ${(aircraft.distance || 0).toFixed(1)} nm`);

      alerted.add(hex);
    } else if (!inRange && alerted.has(hex)) {
      alerted.delete(hex);
    }
  }
});

es.onerror = (err) => console.error('SSE error:', err);
```

## Speed Categories Reference

| Category | Speed Range | Typical Traffic |
|----------|-------------|-----------------|
| Stationary | 0 - 10 kts | Parked aircraft, ground operations |
| Slow | 10 - 100 kts | Helicopters, small props, drones |
| Normal | 100 - 250 kts | Regional aircraft, GA traffic |
| Fast | 250 - 500 kts | Commercial jets, business aviation |
| Very Fast | 500+ kts | Military jets, supersonic aircraft |

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `MIN_SPEED` | Minimum speed threshold (kts) | `0` |
| `MAX_SPEED` | Maximum speed threshold (kts) | `100` |

## Use Cases

### Slow Mover Detection (Drones, Helicopters)

```shell
MIN_SPEED=0 MAX_SPEED=100 python speed_filter.py
```

Catches:
- Drones and UAVs
- Helicopters
- Small propeller aircraft
- Blimps and airships
- Unusual slow traffic

### Fast Mover Detection (Military Jets)

```shell
MIN_SPEED=500 MAX_SPEED=2000 python speed_filter.py
```

Catches:
- Military fighter jets
- Supersonic aircraft
- High-speed interceptors
- Test flights

### Stationary Aircraft Monitor

```shell
MIN_SPEED=0 MAX_SPEED=10 python speed_filter.py
```

Catches:
- Parked aircraft with transponders on
- Ground operations
- Aircraft holding position
- Engine run-ups

### Normal Traffic Filter

```shell
MIN_SPEED=100 MAX_SPEED=300 python speed_filter.py
```

Catches:
- Regional airline traffic
- General aviation
- Training flights
- Approach/departure traffic

## Multiple Speed Bands

Monitor multiple speed bands with priority alerts:

```python
BANDS = [
    {"name": "Stationary", "min": 0, "max": 10, "priority": "low"},
    {"name": "Slow Mover", "min": 10, "max": 100, "priority": "medium"},
    {"name": "Fast Mover", "min": 500, "max": 2000, "priority": "high"},
]

def check_bands(speed):
    """Check which bands the aircraft is in."""
    matches = []
    for band in BANDS:
        if band["min"] <= speed <= band["max"]:
            matches.append(band)
    return matches
```

## Add Webhook Notifications

```python
import requests

WEBHOOK_URL = os.getenv("WEBHOOK_URL")

def send_alert(aircraft, speed, category):
    if not WEBHOOK_URL:
        return

    payload = {
        "text": f"{category} aircraft alert",
        "aircraft": {
            "hex": aircraft.get("hex"),
            "flight": aircraft.get("flight"),
            "speed": speed,
            "type": aircraft.get("type"),
        }
    }

    requests.post(WEBHOOK_URL, json=payload, timeout=5)
```

## Create Alert Rule

Use SkySpy's built-in alerts for speed filtering:

```shell
# Alert on fast movers (military jets)
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Fast Mover Alert",
    "enabled": true,
    "priority": "high",
    "conditions": {
      "operator": "AND",
      "conditions": [
        {"field": "gs", "operator": "gt", "value": 500}
      ]
    },
    "notification_enabled": true
  }'
```

```shell
# Alert on slow movers (drones/helicopters)
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Slow Mover Alert",
    "enabled": true,
    "priority": "medium",
    "conditions": {
      "operator": "AND",
      "conditions": [
        {"field": "gs", "operator": "lt", "value": 100},
        {"field": "gs", "operator": "gt", "value": 0}
      ]
    },
    "notification_enabled": true
  }'
```

## Testing & Verification

1. **Set a wide speed range** initially (0-600 kts)
2. **Run the filter** and verify aircraft are detected
3. **Narrow the range** to your target speeds
4. **Check alerts trigger** at speed boundaries

### Test Different Speed Ranges

Aircraft speeds vary by type and phase of flight:
- **Near airports**: Expect slower speeds (approach/departure)
- **En route traffic**: Faster cruise speeds
- **Helicopters**: Look near hospitals, police stations
- **Military**: Near air bases or training areas

## Troubleshooting

### No Aircraft Detected

**Problem:** Filter running but no matches
**Solution:**
- Widen speed range
- Verify aircraft have ground speed data (`gs` field)
- Check receiver coverage
- Some aircraft may not report ground speed

### Too Many Alerts

**Problem:** Constant notifications
**Solution:**
- Narrow speed range
- Add altitude filter to exclude ground traffic
- Add distance filter
- Increase deduplication time

### Speed Data Missing

**Problem:** Aircraft visible but no speed alerts
**Solution:**
- Not all aircraft report ground speed
- ADS-B coverage may be incomplete
- Check if `gs` field exists in raw data
- Mode S aircraft may lack speed data

### False Positives on Ground

**Problem:** Alerts for taxiing aircraft
**Solution:**
- Add minimum altitude filter
- Filter out aircraft below 100 ft
- Combine with `on_ground` field if available

```python
# Filter out ground traffic
if aircraft.get("alt", 0) < 100:
    continue
```

## Related Recipes

- [Altitude Band Filter](/docs/altitude-filter) - Altitude-based filtering
- [Radius Filter](/docs/radius-filter) - Distance-based filtering
- [Track Specific Aircraft](/docs/track-aircraft) - Watchlist filtering
- [Emergency Monitor](/docs/emergency-monitor) - Squawk filtering
