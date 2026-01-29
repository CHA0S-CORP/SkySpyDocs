---
title: Philips Hue Alerts
excerpt: Flash smart lights on aircraft events.
hidden: false
recipe:
  color: '#FFB800'
  icon: 💡
difficulty: beginner
tags: [iot, smart-home, hue, lights]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Philips Hue Bridge on your network
- Hue Bridge username/API key

## What You'll Build

Flash your Philips Hue lights when aircraft are detected overhead. Different colors indicate aircraft type: red for military, blue for emergency squawks, and green for normal aircraft.

```shell Shell
pip install sseclient-py requests
export HUE_BRIDGE_IP="192.168.1.100"
export HUE_USERNAME="your-hue-api-username"
python hue_alerts.py
```

```go Go
package main

import (
    "bufio"
    "bytes"
    "crypto/tls"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "strings"
    "time"
)

func main() {
    bridgeIP := os.Getenv("HUE_BRIDGE_IP")
    username := os.Getenv("HUE_USERNAME")
    skyspyURL := os.Getenv("SKYSPY_URL")
    lightID := os.Getenv("HUE_LIGHT_ID")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }
    if lightID == "" {
        lightID = "1"
    }

    if bridgeIP == "" || username == "" {
        fmt.Println("Error: HUE_BRIDGE_IP and HUE_USERNAME required")
        os.Exit(1)
    }

    tr := &http.Transport{TLSClientConfig: &tls.Config{InsecureSkipVerify: true}}
    client := &http.Client{Transport: tr}
    baseURL := fmt.Sprintf("https://%s/api/%s", bridgeIP, username)

    seenAircraft := make(map[string]bool)

    for {
        resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
        if err != nil {
            fmt.Println("Connection error, retrying...")
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
                        hex := ac["hex"].(string)

                        if seenAircraft[hex] {
                            continue
                        }
                        seenAircraft[hex] = true

                        var hue int
                        if ac["military"] == true {
                            hue = 0 // Red
                        } else if squawk, ok := ac["squawk"].(string); ok && isEmergency(squawk) {
                            hue = 46920 // Blue
                        } else {
                            hue = 25500 // Green
                        }

                        flashLight(client, baseURL, lightID, hue)
                    }
                }
            }
        }
        resp.Body.Close()
    }
}

func isEmergency(squawk string) bool {
    return squawk == "7500" || squawk == "7600" || squawk == "7700"
}

func flashLight(client *http.Client, baseURL, lightID string, hue int) {
    url := fmt.Sprintf("%s/lights/%s/state", baseURL, lightID)

    // Flash on
    body, _ := json.Marshal(map[string]interface{}{
        "on": true, "bri": 254, "hue": hue, "sat": 254, "alert": "select",
    })
    req, _ := http.NewRequest("PUT", url, bytes.NewReader(body))
    req.Header.Set("Content-Type", "application/json")
    client.Do(req)
}
```

```python Python
import json
import os
import sys
import time
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
HUE_BRIDGE_IP = os.getenv("HUE_BRIDGE_IP")
HUE_USERNAME = os.getenv("HUE_USERNAME")
HUE_LIGHT_ID = os.getenv("HUE_LIGHT_ID", "1")

if not HUE_BRIDGE_IP or not HUE_USERNAME:
    print("Error: HUE_BRIDGE_IP and HUE_USERNAME required")
    sys.exit(1)

BASE_URL = f"https://{HUE_BRIDGE_IP}/api/{HUE_USERNAME}"
session = requests.Session()
session.verify = False  # Hue Bridge uses self-signed cert

# Color definitions (Hue values 0-65535)
COLORS = {
    "military": {"hue": 0, "sat": 254, "bri": 254},         # Red
    "emergency": {"hue": 46920, "sat": 254, "bri": 254},    # Blue
    "normal": {"hue": 25500, "sat": 254, "bri": 254},       # Green
}


def flash_light(light_id, color_type="normal"):
    """Flash a Hue light with the specified color."""
    url = f"{BASE_URL}/lights/{light_id}/state"
    color = COLORS.get(color_type, COLORS["normal"])

    try:
        # Flash the light
        session.put(url, json={
            "on": True,
            **color,
            "alert": "select"  # Single flash
        })
        return True
    except Exception as e:
        print(f"Failed to flash light: {e}")
        return False


def flash_group(group_id, color_type="normal"):
    """Flash a Hue group/room with the specified color."""
    url = f"{BASE_URL}/groups/{group_id}/action"
    color = COLORS.get(color_type, COLORS["normal"])

    try:
        session.put(url, json={
            "on": True,
            **color,
            "alert": "select"
        })
        return True
    except Exception as e:
        print(f"Failed to flash group: {e}")
        return False


def is_emergency_squawk(squawk):
    """Check if squawk code indicates emergency."""
    return squawk in ["7500", "7600", "7700"]


def get_aircraft_type(aircraft):
    """Determine aircraft type for color selection."""
    if aircraft.get("military"):
        return "military"
    if is_emergency_squawk(aircraft.get("squawk", "")):
        return "emergency"
    return "normal"


seen_aircraft = set()


def process_aircraft(aircraft_list):
    """Process aircraft and flash lights for new detections."""
    global seen_aircraft

    for aircraft in aircraft_list:
        hex_code = aircraft.get("hex")
        if hex_code in seen_aircraft:
            continue

        seen_aircraft.add(hex_code)
        aircraft_type = get_aircraft_type(aircraft)
        flight = aircraft.get("flight", hex_code)

        print(f"New aircraft: {flight} ({aircraft_type})")
        flash_light(HUE_LIGHT_ID, aircraft_type)


def main():
    print(f"Hue Alerts connected to {SKYSPY_URL}...")
    print(f"Using Hue Bridge at {HUE_BRIDGE_IP}")

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
                    aircraft_list = data.get("aircraft", [])
                    if aircraft_list:
                        process_aircraft(aircraft_list)

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const fetch = require('node-fetch');
const https = require('https');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const HUE_BRIDGE_IP = process.env.HUE_BRIDGE_IP;
const HUE_USERNAME = process.env.HUE_USERNAME;
const HUE_LIGHT_ID = process.env.HUE_LIGHT_ID || '1';

if (!HUE_BRIDGE_IP || !HUE_USERNAME) {
  console.error('Error: HUE_BRIDGE_IP and HUE_USERNAME required');
  process.exit(1);
}

const BASE_URL = `https://${HUE_BRIDGE_IP}/api/${HUE_USERNAME}`;

// Allow self-signed certs for Hue Bridge
const agent = new https.Agent({ rejectUnauthorized: false });

const COLORS = {
  military: { hue: 0, sat: 254, bri: 254 },        // Red
  emergency: { hue: 46920, sat: 254, bri: 254 },   // Blue
  normal: { hue: 25500, sat: 254, bri: 254 }       // Green
};

async function flashLight(lightId, colorType = 'normal') {
  const color = COLORS[colorType] || COLORS.normal;
  const url = `${BASE_URL}/lights/${lightId}/state`;

  await fetch(url, {
    method: 'PUT',
    agent,
    body: JSON.stringify({
      on: true,
      ...color,
      alert: 'select'
    })
  });
}

function isEmergencySquawk(squawk) {
  return ['7500', '7600', '7700'].includes(squawk);
}

function getAircraftType(aircraft) {
  if (aircraft.military) return 'military';
  if (isEmergencySquawk(aircraft.squawk)) return 'emergency';
  return 'normal';
}

const seenAircraft = new Set();

console.log(`Hue Alerts connected to ${SKYSPY_URL}...`);
console.log(`Using Hue Bridge at ${HUE_BRIDGE_IP}`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);
  const aircraft = data.aircraft || [];

  for (const ac of aircraft) {
    if (seenAircraft.has(ac.hex)) continue;

    seenAircraft.add(ac.hex);
    const acType = getAircraftType(ac);
    const flight = ac.flight || ac.hex;

    console.log(`New aircraft: ${flight} (${acType})`);
    await flashLight(HUE_LIGHT_ID, acType);
  }
});

es.onerror = (err) => {
  console.error('SSE error:', err);
};
```

## Get Hue Bridge Username

1. **Find your Bridge IP**:
   - Check the Hue app under Settings > Hue Bridges
   - Or visit https://discovery.meethue.com

2. **Create a username**:
   ```shell
   # Press the link button on your Hue Bridge, then run:
   curl -X POST https://YOUR_BRIDGE_IP/api \
     -d '{"devicetype":"skyspy#alerts"}' \
     --insecure
   ```

3. **Copy the username** from the response:
   ```json
   [{"success":{"username":"abc123..."}}]
   ```

## Flash Lights on Detection

<!-- python@30-45 -->

The script flashes lights with different colors based on aircraft type:
- **Red** - Military aircraft
- **Blue** - Emergency squawk codes (7500, 7600, 7700)
- **Green** - Normal civilian aircraft

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `HUE_BRIDGE_IP` | IP address of your Hue Bridge | Required |
| `HUE_USERNAME` | Hue API username/key | Required |
| `HUE_LIGHT_ID` | Light ID to flash | `1` |
| `HUE_GROUP_ID` | Group/room ID to flash | None |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Find Light IDs

```shell
curl https://$HUE_BRIDGE_IP/api/$HUE_USERNAME/lights --insecure | jq
```

### Find Group IDs

```shell
curl https://$HUE_BRIDGE_IP/api/$HUE_USERNAME/groups --insecure | jq
```

## Testing & Verification

1. **Verify Hue Bridge access**:
   ```shell
   curl https://$HUE_BRIDGE_IP/api/$HUE_USERNAME/lights --insecure
   ```
2. **Test light flash manually**:
   ```shell
   curl -X PUT https://$HUE_BRIDGE_IP/api/$HUE_USERNAME/lights/1/state \
     -d '{"on":true,"alert":"select"}' --insecure
   ```
3. **Start the script** and watch for "New aircraft" messages
4. **Check lights flash** when aircraft are detected

## Troubleshooting

### Unauthorized Error

**Problem:** API returns unauthorized or link button not pressed
**Solution:**
- Press the link button on your Hue Bridge
- Create a new username within 30 seconds of pressing
- Verify username is correct

### Light Not Flashing

**Problem:** Script runs but lights don't flash
**Solution:**
- Verify light ID exists with `/lights` endpoint
- Check light is reachable (not powered off)
- Test with manual curl command
- Try a different light ID

### SSL Certificate Errors

**Problem:** Connection fails with SSL/certificate error
**Solution:**
- Use `--insecure` flag with curl
- Set `verify=False` in Python requests
- Use `rejectUnauthorized: false` in Node.js

## Related Recipes

- [Home Assistant Integration](/docs/home-assistant) - Full HA integration
- [MQTT Publisher](/docs/mqtt-publisher) - MQTT integration
- [Discord Alert Bot](/docs/discord-alert-bot) - Alert notifications
- [Telegram Bot](/docs/telegram-bot) - Mobile alerts
