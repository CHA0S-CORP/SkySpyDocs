---
title: Home Assistant Integration
excerpt: Create HA sensors and automations from aircraft data.
hidden: false
recipe:
  color: '#41BDF5'
  icon: 🏠
difficulty: intermediate
tags: [smart-home, home-assistant, sensors, automation]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Home Assistant instance with API access
- Long-lived access token from HA

## What You'll Build

Home Assistant sensors that update in real-time with aircraft data, enabling automations like flashing lights when military aircraft are overhead or announcing aircraft on speakers.

```shell Shell
pip install sseclient-py requests
export HA_URL="http://homeassistant.local:8123"
export HA_TOKEN="your-long-lived-access-token"
python ha_integration.py
```

```go Go
package main

import (
    "bufio"
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "strings"
)

func main() {
    haURL := os.Getenv("HA_URL")
    haToken := os.Getenv("HA_TOKEN")
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    if haURL == "" || haToken == "" {
        fmt.Println("Error: HA_URL and HA_TOKEN required")
        os.Exit(1)
    }

    client := &http.Client{}

    for {
        resp, _ := http.Get(skyspyURL + "/api/v1/map/sse")
        scanner := bufio.NewScanner(resp.Body)

        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data map[string]interface{}
                json.Unmarshal([]byte(line[5:]), &data)

                if aircraft, ok := data["aircraft"].([]interface{}); ok {
                    // Update aircraft count sensor
                    updateSensor(client, haURL, haToken, "sensor.skyspy_aircraft_count", map[string]interface{}{
                        "state": len(aircraft),
                        "attributes": map[string]interface{}{
                            "friendly_name": "SkySpy Aircraft Count",
                            "icon":          "mdi:airplane",
                        },
                    })

                    // Count military aircraft
                    military := 0
                    for _, a := range aircraft {
                        ac := a.(map[string]interface{})
                        if ac["military"] == true {
                            military++
                        }
                    }

                    updateSensor(client, haURL, haToken, "sensor.skyspy_military_count", map[string]interface{}{
                        "state": military,
                        "attributes": map[string]interface{}{
                            "friendly_name": "Military Aircraft Nearby",
                            "icon":          "mdi:shield-airplane",
                        },
                    })
                }
            }
        }
        resp.Body.Close()
    }
}

func updateSensor(client *http.Client, haURL, token, entityID string, data map[string]interface{}) {
    body, _ := json.Marshal(data)
    req, _ := http.NewRequest("POST", haURL+"/api/states/"+entityID, bytes.NewReader(body))
    req.Header.Set("Authorization", "Bearer "+token)
    req.Header.Set("Content-Type", "application/json")
    client.Do(req)
}
```

```python Python
import json
import os
import sys
from datetime import datetime
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
HA_URL = os.getenv("HA_URL", "http://homeassistant.local:8123")
HA_TOKEN = os.getenv("HA_TOKEN")

if not HA_TOKEN:
    print("Error: HA_TOKEN required")
    sys.exit(1)

session = requests.Session()
session.headers.update({
    "Authorization": f"Bearer {HA_TOKEN}",
    "Content-Type": "application/json"
})


def update_sensor(entity_id, state, attributes=None):
    """Update a Home Assistant sensor."""
    data = {
        "state": state,
        "attributes": attributes or {}
    }
    try:
        response = session.post(f"{HA_URL}/api/states/{entity_id}", json=data)
        return response.status_code in [200, 201]
    except Exception as e:
        print(f"Failed to update {entity_id}: {e}")
        return False


def fire_event(event_type, data):
    """Fire a Home Assistant event."""
    try:
        session.post(f"{HA_URL}/api/events/{event_type}", json=data)
    except Exception:
        pass


def process_aircraft(aircraft_list):
    """Process aircraft list and update HA sensors."""
    total = len(aircraft_list)
    military = [a for a in aircraft_list if a.get("military")]
    closest = min(aircraft_list, key=lambda a: a.get("distance", 999), default=None)

    # Update count sensors
    update_sensor("sensor.skyspy_aircraft_count", total, {
        "friendly_name": "Aircraft Overhead",
        "icon": "mdi:airplane",
        "unit_of_measurement": "aircraft"
    })

    update_sensor("sensor.skyspy_military_count", len(military), {
        "friendly_name": "Military Aircraft",
        "icon": "mdi:shield-airplane",
        "unit_of_measurement": "aircraft"
    })

    # Update closest aircraft sensor
    if closest:
        update_sensor("sensor.skyspy_closest_aircraft", closest.get("distance", 0), {
            "friendly_name": "Closest Aircraft",
            "icon": "mdi:airplane-landing",
            "unit_of_measurement": "nm",
            "flight": closest.get("flight", closest.get("hex")),
            "type": closest.get("type", "Unknown"),
            "altitude": closest.get("alt", 0),
            "speed": closest.get("gs", 0),
            "hex": closest.get("hex")
        })

    # Binary sensor for military presence
    update_sensor("binary_sensor.skyspy_military_nearby", "on" if military else "off", {
        "friendly_name": "Military Aircraft Nearby",
        "device_class": "presence",
        "icon": "mdi:shield-airplane"
    })

    # Fire events for new military aircraft
    global seen_military
    for aircraft in military:
        hex_code = aircraft.get("hex")
        if hex_code not in seen_military:
            fire_event("skyspy_military_aircraft", {
                "hex": hex_code,
                "flight": aircraft.get("flight"),
                "type": aircraft.get("type"),
                "altitude": aircraft.get("alt"),
                "distance": aircraft.get("distance")
            })
            seen_military.add(hex_code)


seen_military = set()


def main():
    print(f"Home Assistant integration connected to {SKYSPY_URL}...")
    print(f"Updating sensors at {HA_URL}")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    aircraft_list = data.get("aircraft", [])
                    if aircraft_list:
                        process_aircraft(aircraft_list)

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            import time
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const fetch = require('node-fetch');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const HA_URL = process.env.HA_URL || 'http://homeassistant.local:8123';
const HA_TOKEN = process.env.HA_TOKEN;

if (!HA_TOKEN) {
  console.error('Error: HA_TOKEN required');
  process.exit(1);
}

const headers = {
  'Authorization': `Bearer ${HA_TOKEN}`,
  'Content-Type': 'application/json'
};

async function updateSensor(entityId, state, attributes = {}) {
  await fetch(`${HA_URL}/api/states/${entityId}`, {
    method: 'POST',
    headers,
    body: JSON.stringify({ state, attributes })
  });
}

async function fireEvent(eventType, data) {
  await fetch(`${HA_URL}/api/events/${eventType}`, {
    method: 'POST',
    headers,
    body: JSON.stringify(data)
  });
}

const seenMilitary = new Set();

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);
  const aircraft = data.aircraft || [];

  await updateSensor('sensor.skyspy_aircraft_count', aircraft.length, {
    friendly_name: 'Aircraft Overhead',
    icon: 'mdi:airplane'
  });

  const military = aircraft.filter(a => a.military);
  await updateSensor('sensor.skyspy_military_count', military.length, {
    friendly_name: 'Military Aircraft',
    icon: 'mdi:shield-airplane'
  });

  for (const ac of military) {
    if (!seenMilitary.has(ac.hex)) {
      await fireEvent('skyspy_military_aircraft', ac);
      seenMilitary.add(ac.hex);
    }
  }
});
```

```yaml Home Assistant Automation Example
# configuration.yaml or automations.yaml

automation:
  - alias: "Flash lights when military aircraft nearby"
    trigger:
      - platform: state
        entity_id: binary_sensor.skyspy_military_nearby
        to: "on"
    action:
      - service: light.turn_on
        target:
          entity_id: light.living_room
        data:
          flash: short

  - alias: "Announce military aircraft"
    trigger:
      - platform: event
        event_type: skyspy_military_aircraft
    action:
      - service: tts.speak
        target:
          entity_id: tts.google_en
        data:
          message: >
            Military aircraft {{ trigger.event.data.flight }}
            at {{ trigger.event.data.altitude }} feet

  - alias: "Notify on emergency squawk"
    trigger:
      - platform: event
        event_type: skyspy_emergency
    action:
      - service: notify.mobile_app
        data:
          title: "Emergency Squawk!"
          message: >
            Aircraft {{ trigger.event.data.flight }} squawking
            {{ trigger.event.data.squawk }}
```

## Get HA Long-Lived Token

1. Go to your HA profile (click your username)
2. Scroll to "Long-Lived Access Tokens"
3. Click "Create Token"
4. Copy and save the token securely

## Create Sensors

<!-- python@25-65 -->

The integration creates these sensors:
- `sensor.skyspy_aircraft_count` - Total aircraft in range
- `sensor.skyspy_military_count` - Military aircraft count
- `sensor.skyspy_closest_aircraft` - Distance to closest aircraft
- `binary_sensor.skyspy_military_nearby` - On when military present

## Fire Events

<!-- python@38-45 -->

Events enable automations. The script fires `skyspy_military_aircraft` with full aircraft data when new military aircraft are detected.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `HA_URL` | Home Assistant URL | `http://homeassistant.local:8123` |
| `HA_TOKEN` | Long-lived access token | Required |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Lovelace Cards

Display aircraft data on your dashboard:

```yaml
type: entities
entities:
  - entity: sensor.skyspy_aircraft_count
  - entity: sensor.skyspy_military_count
  - entity: sensor.skyspy_closest_aircraft
  - entity: binary_sensor.skyspy_military_nearby
```

### Gauge Card

```yaml
type: gauge
entity: sensor.skyspy_aircraft_count
min: 0
max: 50
severity:
  green: 0
  yellow: 20
  red: 40
```

## Testing & Verification

1. **Verify HA API access**:
   ```shell
   curl -H "Authorization: Bearer $HA_TOKEN" $HA_URL/api/
   ```
2. **Start the integration** and check for "connected" message
3. **Check HA Developer Tools → States** for new sensors
4. **Test automations** when military aircraft appear

## Troubleshooting

### 401 Unauthorized

**Problem:** API returns unauthorized
**Solution:**
- Verify token is correct and not expired
- Check HA URL is correct (include port)
- Regenerate token if needed

### Sensors Not Appearing

**Problem:** Sensors don't show in HA
**Solution:**
- Check HA logs for API errors
- Verify entity_ids are valid (lowercase, underscores)
- Restart HA after first run

### Automations Not Triggering

**Problem:** Events fire but automations don't run
**Solution:**
- Check automation is enabled
- Verify event data structure matches trigger
- Check HA logs for automation traces

## Related Recipes

- [Node-RED Flows](/docs/node-red-flows) - Visual automation
- [MQTT Publisher](/docs/mqtt-publisher) - MQTT integration
- [Philips Hue Alerts](/docs/hue-alerts) - Light automation
- [Alexa Announcements](/docs/alexa-announcements) - Voice alerts
