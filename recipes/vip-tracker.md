---
title: VIP Aircraft Tracker
excerpt: Track known VIP and celebrity aircraft by tail number.
hidden: false
recipe:
  color: '#8B5CF6'
  icon: ⭐
difficulty: beginner
tags: [tracking, vip, alerts, specialty]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- List of VIP aircraft ICAO hex codes

## What You'll Build

A watchlist monitor that alerts when known VIP, celebrity, or interesting aircraft appear in your coverage area.

```shell Shell
pip install sseclient-py requests
python vip_tracker.py
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

// VIP Aircraft Database
var vipAircraft = map[string]string{
    // US Government
    "AE01CF": "Air Force One (VC-25A)",
    "AE01D0": "Air Force One (VC-25A)",
    "AE041C": "Air Force Two (C-32A)",

    // Business Leaders
    "A835AF": "Trump 757 (N757AF)",
    "A00F69": "Bezos G650 (N271DV)",

    // Entertainment
    "A326CA": "Swift Falcon 7X (N898TS)",
    "A5E75B": "Travolta 707 (N707JT)",

    // Historic
    "A2A2A2": "Fifi B-29",
    "A8A0E7": "Doc B-29",
}

func main() {
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    alerted := make(map[string]bool)

    fmt.Printf("Tracking %d VIP aircraft...\n", len(vipAircraft))

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
                        hex := strings.ToUpper(fmt.Sprint(ac["hex"]))

                        if name, isVIP := vipAircraft[hex]; isVIP && !alerted[hex] {
                            fmt.Printf("⭐ VIP SPOTTED: %s\n", name)
                            fmt.Printf("   Callsign: %s\n", ac["flight"])
                            fmt.Printf("   Altitude: %v ft\n", ac["alt"])
                            fmt.Printf("   Position: %v, %v\n", ac["lat"], ac["lon"])
                            alerted[hex] = true
                        }
                    }
                }
            }
        }
        resp.Body.Close()
    }
}
```

```python Python
import json
import os
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

# VIP Aircraft Database
VIP_AIRCRAFT = {
    # US Government
    "AE01CF": {"name": "Air Force One", "type": "VC-25A", "category": "government"},
    "AE01D0": {"name": "Air Force One", "type": "VC-25A", "category": "government"},
    "AE041C": {"name": "Air Force Two", "type": "C-32A", "category": "government"},
    "AE0416": {"name": "E-4B Nightwatch", "type": "E-4B", "category": "government"},

    # Business Leaders
    "A835AF": {"name": "Trump Force One", "type": "757", "category": "celebrity", "reg": "N757AF"},
    "A00F69": {"name": "Jeff Bezos", "type": "G650", "category": "celebrity"},
    "A59E45": {"name": "Elon Musk", "type": "G650ER", "category": "celebrity"},
    "A2B48F": {"name": "Bill Gates", "type": "G650ER", "category": "celebrity"},

    # Entertainment
    "A326CA": {"name": "Taylor Swift", "type": "Falcon 7X", "category": "celebrity", "reg": "N898TS"},
    "A5E75B": {"name": "John Travolta", "type": "707", "category": "celebrity", "reg": "N707JT"},
    "A7A8E7": {"name": "Kylie Jenner", "type": "Global 7500", "category": "celebrity"},

    # Historic Aircraft
    "A2A2A2": {"name": "Fifi", "type": "B-29", "category": "historic"},
    "A8A0E7": {"name": "Doc", "type": "B-29", "category": "historic"},
    "A45E85": {"name": "Aluminum Overcast", "type": "B-17", "category": "historic"},
    "A5CD8F": {"name": "Collings B-24", "type": "B-24", "category": "historic"},

    # Sports Teams (examples)
    "A22222": {"name": "New England Patriots", "type": "767", "category": "sports"},
}

alerted = set()


def on_vip_spotted(hex_code, aircraft, vip_info):
    """Handle VIP aircraft detection."""
    print(f"\n⭐ VIP AIRCRAFT SPOTTED ⭐")
    print(f"   {vip_info['name']} ({vip_info['type']})")
    print(f"   Category: {vip_info['category'].title()}")
    print(f"   Callsign: {aircraft.get('flight', 'Unknown')}")
    print(f"   Altitude: {aircraft.get('alt', 0):,} ft")
    print(f"   Speed: {aircraft.get('gs', 0)} kts")
    print(f"   Distance: {aircraft.get('distance', 0):.1f} nm")
    if "reg" in vip_info:
        print(f"   Registration: {vip_info['reg']}")
    print()

    # Here you could add notifications (Discord, Telegram, etc.)


def main():
    print(f"VIP Tracker connected to {SKYSPY_URL}")
    print(f"Watching for {len(VIP_AIRCRAFT)} VIP aircraft...\n")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    for aircraft in data.get("aircraft", []):
                        hex_code = aircraft.get("hex", "").upper()

                        if hex_code in VIP_AIRCRAFT and hex_code not in alerted:
                            on_vip_spotted(hex_code, aircraft, VIP_AIRCRAFT[hex_code])
                            alerted.add(hex_code)

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

const VIP_AIRCRAFT = {
  // US Government
  'AE01CF': { name: 'Air Force One', type: 'VC-25A' },
  'AE01D0': { name: 'Air Force One', type: 'VC-25A' },
  'AE041C': { name: 'Air Force Two', type: 'C-32A' },

  // Celebrities
  'A835AF': { name: 'Trump 757', type: '757', reg: 'N757AF' },
  'A326CA': { name: 'Taylor Swift', type: 'Falcon 7X' },
  'A5E75B': { name: 'John Travolta', type: '707' },

  // Historic
  'A2A2A2': { name: 'Fifi B-29', type: 'B-29' },
  'A8A0E7': { name: 'Doc B-29', type: 'B-29' },
};

const alerted = new Set();

console.log(`Tracking ${Object.keys(VIP_AIRCRAFT).length} VIP aircraft...`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const hex = (aircraft.hex || '').toUpperCase();

    if (VIP_AIRCRAFT[hex] && !alerted.has(hex)) {
      const vip = VIP_AIRCRAFT[hex];
      console.log(`\n⭐ VIP SPOTTED: ${vip.name} (${vip.type})`);
      console.log(`   Callsign: ${aircraft.flight || 'Unknown'}`);
      console.log(`   Altitude: ${aircraft.alt || 0} ft`);
      console.log(`   Distance: ${(aircraft.distance || 0).toFixed(1)} nm`);
      alerted.add(hex);
    }
  }
});
```

```json VIP Database Format
{
  "AE01CF": {
    "name": "Air Force One",
    "type": "VC-25A",
    "category": "government",
    "notes": "Primary presidential aircraft"
  },
  "A835AF": {
    "name": "Trump Force One",
    "type": "757-200",
    "category": "celebrity",
    "reg": "N757AF",
    "notes": "Personal aircraft"
  }
}
```

## VIP Database

The database maps ICAO hex codes to aircraft information:

| Category | Examples |
|----------|----------|
| Government | Air Force One, E-4B Nightwatch |
| Celebrity | Private jets of public figures |
| Historic | Warbirds, vintage aircraft |
| Sports | Team aircraft |
| Corporate | Notable company planes |

### Finding ICAO Hex Codes

1. **ADS-B Exchange** - Search by registration and note the hex
2. **FlightAware** - Shows hex code in aircraft details
3. **FAA Registry** - Convert N-numbers to hex

## Add Notifications

<!-- python@50-65 -->

Integrate with other recipes to send alerts:

```python
from discord_webhook import DiscordWebhook

def on_vip_spotted(hex_code, aircraft, vip_info):
    # Console output
    print(f"⭐ {vip_info['name']} spotted!")

    # Discord notification
    webhook = DiscordWebhook(url=os.getenv("DISCORD_WEBHOOK"))
    webhook.set_content(f"⭐ **VIP Alert**: {vip_info['name']} ({vip_info['type']})")
    webhook.execute()
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### External Database

Load VIP list from a file:

```python
import json

with open("vip_aircraft.json") as f:
    VIP_AIRCRAFT = json.load(f)

# Or from URL
response = requests.get("https://example.com/vip-aircraft.json")
VIP_AIRCRAFT = response.json()
```

### Categories

Filter by category:

```python
# Only track government aircraft
CATEGORIES = {"government"}

if hex_code in VIP_AIRCRAFT:
    if VIP_AIRCRAFT[hex_code]["category"] in CATEGORIES:
        on_vip_spotted(...)
```

## Create Alert Rule

Use SkySpy's built-in alerts:

```shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Air Force One",
    "enabled": true,
    "priority": "critical",
    "conditions": {
      "operator": "OR",
      "conditions": [
        {"field": "hex", "operator": "eq", "value": "AE01CF"},
        {"field": "hex", "operator": "eq", "value": "AE01D0"}
      ]
    },
    "notification_enabled": true
  }'
```

## Testing & Verification

1. **Start the tracker** and verify connection
2. **Check database loads** - see count of tracked aircraft
3. **Wait for VIP flights** - check FlightAware for active VIP aircraft
4. **Test with common aircraft** - temporarily add a frequent flyer

### Test Mode

Add a common aircraft for testing:

```python
# Add any frequent aircraft for testing
VIP_AIRCRAFT["A12345"] = {"name": "Test Aircraft", "type": "Test", "category": "test"}
```

## Troubleshooting

### No VIP Alerts

**Problem:** Known VIP flew over but no alert
**Solution:**
- Verify hex code is correct (uppercase)
- Check aircraft was in ADS-B coverage
- Confirm aircraft was transmitting position

### Wrong Aircraft

**Problem:** Alerts for different aircraft than expected
**Solution:**
- Hex codes can be reassigned
- Verify current registration matches hex
- Update database regularly

### Duplicate Alerts

**Problem:** Same aircraft alerting multiple times
**Solution:**
- Check the `alerted` set is working
- Consider adding time-based expiry:
  ```python
  alerted[hex_code] = time.time()
  # Expire after 24 hours
  if time.time() - alerted[hex_code] > 86400:
      del alerted[hex_code]
  ```

## Disclaimer

Tracking VIP aircraft is legal as ADS-B data is publicly broadcast. However:
- Respect privacy
- Don't share real-time location data publicly
- Be aware some aircraft may use anonymous modes

## Related Recipes

- [Track Specific Aircraft](/docs/track-aircraft) - Track by ICAO hex
- [Military Aircraft Spotter](/docs/military-spotter) - Military tracking
- [Discord Alert Bot](/docs/discord-alert-bot) - Send alerts
- [Rare Aircraft Spotter](/docs/rare-spotter) - Unusual types
