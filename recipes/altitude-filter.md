---
title: Altitude Band Filter
excerpt: Filter aircraft by altitude ranges for specific monitoring.
hidden: false
recipe:
  color: '#6366F1'
  icon: 📏
difficulty: beginner
tags: [filtering, altitude, geographic, monitoring]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Python: `pip install sseclient-py requests`

## What You'll Build

A filter that monitors aircraft in specific altitude bands - useful for tracking low-flying aircraft, high-altitude traffic, or approach/departure patterns.

```shell Shell
pip install sseclient-py requests
export LOW_ALT=0
export HIGH_ALT=5000
python altitude_filter.py
```

```python Python
import json
import os
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
LOW_ALT = int(os.getenv("LOW_ALT", "0"))
HIGH_ALT = int(os.getenv("HIGH_ALT", "10000"))
ALERT_LOW = os.getenv("ALERT_LOW", "true").lower() == "true"

alerted = set()


def check_altitude(aircraft):
    """Check if aircraft is in the specified altitude band."""
    altitude = aircraft.get("alt")
    if altitude is None:
        return False, None

    in_band = LOW_ALT <= altitude <= HIGH_ALT
    return in_band, altitude


def format_altitude_category(altitude):
    """Categorize altitude for display."""
    if altitude < 1000:
        return "Very Low", "🔴"
    elif altitude < 5000:
        return "Low", "🟠"
    elif altitude < 18000:
        return "Medium", "🟡"
    elif altitude < 35000:
        return "High", "🟢"
    else:
        return "Very High", "🔵"


def main():
    print(f"Altitude Filter connected to {SKYSPY_URL}")
    print(f"Monitoring altitude band: {LOW_ALT:,} - {HIGH_ALT:,} ft")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)

                    for aircraft in data.get("aircraft", []):
                        hex_code = aircraft.get("hex")
                        in_band, altitude = check_altitude(aircraft)

                        if in_band and hex_code not in alerted:
                            flight = aircraft.get("flight", "Unknown").strip() or hex_code
                            aircraft_type = aircraft.get("type", "Unknown")
                            distance = aircraft.get("distance", 0)
                            category, emoji = format_altitude_category(altitude)

                            print(f"\n{emoji} Aircraft in altitude band!")
                            print(f"   Flight: {flight}")
                            print(f"   Type: {aircraft_type}")
                            print(f"   Altitude: {altitude:,} ft ({category})")
                            print(f"   Distance: {distance:.1f} nm")

                            alerted.add(hex_code)

                        # Remove from alerted if left the band
                        elif not in_band and hex_code in alerted:
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

func formatCategory(alt float64) (string, string) {
	switch {
	case alt < 1000:
		return "Very Low", "🔴"
	case alt < 5000:
		return "Low", "🟠"
	case alt < 18000:
		return "Medium", "🟡"
	case alt < 35000:
		return "High", "🟢"
	default:
		return "Very High", "🔵"
	}
}

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	lowAlt := float64(getEnvInt("LOW_ALT", 0))
	highAlt := float64(getEnvInt("HIGH_ALT", 10000))

	fmt.Printf("Altitude Filter connected to %s\n", skyspyURL)
	fmt.Printf("Monitoring altitude band: %.0f - %.0f ft\n", lowAlt, highAlt)

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

						alt, hasAlt := ac["alt"].(float64)
						if !hasAlt {
							continue
						}

						inBand := alt >= lowAlt && alt <= highAlt

						if inBand && !alerted[hex] {
							flight := strings.TrimSpace(fmt.Sprint(ac["flight"]))
							if flight == "<nil>" {
								flight = hex
							}
							category, emoji := formatCategory(alt)

							fmt.Printf("\n%s Aircraft in altitude band!\n", emoji)
							fmt.Printf("   Flight: %s\n", flight)
							fmt.Printf("   Type: %v\n", ac["type"])
							fmt.Printf("   Altitude: %.0f ft (%s)\n", alt, category)
							fmt.Printf("   Distance: %.1f nm\n", ac["distance"])

							alerted[hex] = true
						} else if !inBand && alerted[hex] {
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
const LOW_ALT = parseInt(process.env.LOW_ALT || '0');
const HIGH_ALT = parseInt(process.env.HIGH_ALT || '10000');

const alerted = new Set();

function formatCategory(alt) {
  if (alt < 1000) return ['Very Low', '🔴'];
  if (alt < 5000) return ['Low', '🟠'];
  if (alt < 18000) return ['Medium', '🟡'];
  if (alt < 35000) return ['High', '🟢'];
  return ['Very High', '🔵'];
}

console.log(`Altitude Filter connected to ${SKYSPY_URL}`);
console.log(`Monitoring altitude band: ${LOW_ALT.toLocaleString()} - ${HIGH_ALT.toLocaleString()} ft`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;
    const alt = aircraft.alt;

    if (alt == null) continue;

    const inBand = alt >= LOW_ALT && alt <= HIGH_ALT;

    if (inBand && !alerted.has(hex)) {
      const flight = (aircraft.flight || hex).trim();
      const [category, emoji] = formatCategory(alt);

      console.log(`\n${emoji} Aircraft in altitude band!`);
      console.log(`   Flight: ${flight}`);
      console.log(`   Type: ${aircraft.type || 'Unknown'}`);
      console.log(`   Altitude: ${alt.toLocaleString()} ft (${category})`);
      console.log(`   Distance: ${(aircraft.distance || 0).toFixed(1)} nm`);

      alerted.add(hex);
    } else if (!inBand && alerted.has(hex)) {
      alerted.delete(hex);
    }
  }
});

es.onerror = (err) => console.error('SSE error:', err);
```

## Altitude Bands Reference

| Band | Altitude | Typical Traffic |
|------|----------|-----------------|
| Very Low | 0 - 1,000 ft | Helicopters, small aircraft, takeoff/landing |
| Low | 1,000 - 5,000 ft | Training, pattern work, departures |
| Medium | 5,000 - 18,000 ft | Regional flights, climbing/descending |
| High | 18,000 - 35,000 ft | Domestic cruising |
| Very High | 35,000+ ft | Long-haul cruising |

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `LOW_ALT` | Lower altitude bound (ft) | `0` |
| `HIGH_ALT` | Upper altitude bound (ft) | `10000` |

## Use Cases

### Low-Flying Aircraft Monitor

```shell
LOW_ALT=0 HIGH_ALT=1000 python altitude_filter.py
```

Catches:
- Helicopters
- Small aircraft
- Landing/takeoff traffic
- Unusual low-altitude operations

### High Altitude Only

```shell
LOW_ALT=35000 HIGH_ALT=50000 python altitude_filter.py
```

Catches:
- Long-haul international flights
- High-altitude business jets
- Military high-altitude operations

### Approach/Departure Corridor

```shell
LOW_ALT=2000 HIGH_ALT=8000 python altitude_filter.py
```

Catches:
- Aircraft in approach phase
- Departing traffic
- Pattern work

## Multiple Bands

Monitor multiple altitude bands with notifications:

```python
BANDS = [
    {"name": "Ground Level", "low": 0, "high": 500, "priority": "high"},
    {"name": "Low Altitude", "low": 500, "high": 2000, "priority": "medium"},
    {"name": "Approach", "low": 2000, "high": 5000, "priority": "low"},
]

def check_bands(altitude):
    """Check which bands the aircraft is in."""
    matches = []
    for band in BANDS:
        if band["low"] <= altitude <= band["high"]:
            matches.append(band)
    return matches
```

## Add Webhook Notifications

```python
import requests

WEBHOOK_URL = os.getenv("WEBHOOK_URL")

def send_alert(aircraft, altitude, category):
    if not WEBHOOK_URL:
        return

    payload = {
        "text": f"{category} aircraft alert",
        "aircraft": {
            "hex": aircraft.get("hex"),
            "flight": aircraft.get("flight"),
            "altitude": altitude,
            "type": aircraft.get("type"),
        }
    }

    requests.post(WEBHOOK_URL, json=payload, timeout=5)
```

## Create Alert Rule

Use SkySpy's built-in alerts for altitude filtering:

```shell
# Alert on very low aircraft
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Low Flying Aircraft",
    "enabled": true,
    "priority": "high",
    "conditions": {
      "operator": "AND",
      "conditions": [
        {"field": "alt", "operator": "lt", "value": 1000},
        {"field": "alt", "operator": "gt", "value": 0}
      ]
    },
    "notification_enabled": true
  }'
```

## Testing & Verification

1. **Set a wide altitude band** initially (0-50000 ft)
2. **Run the filter** and verify aircraft are detected
3. **Narrow the band** to your target range
4. **Check alerts trigger** at band boundaries

### Simulate Low Aircraft

Aircraft in your area may not always be at low altitudes. For testing:
- Watch during airport approach times
- Look for helicopter traffic
- Check near training airports

## Troubleshooting

### No Aircraft Detected

**Problem:** Filter running but no matches
**Solution:**
- Widen altitude band
- Verify aircraft have altitude data
- Check receiver coverage

### Too Many Alerts

**Problem:** Constant notifications
**Solution:**
- Narrow altitude band
- Add distance filter
- Increase deduplication time

### Missing Low Aircraft

**Problem:** Low-flying aircraft not detected
**Solution:**
- Receiver may not see very low aircraft
- Check antenna line-of-sight
- Low altitude has shorter range

## Related Recipes

- [Radius Filter](/docs/radius-filter) - Distance-based filtering
- [Bounding Box Filter](/docs/bounding-box-filter) - Geographic filtering
- [Track Specific Aircraft](/docs/track-aircraft) - Watchlist filtering
- [Emergency Monitor](/docs/emergency-monitor) - Squawk filtering
