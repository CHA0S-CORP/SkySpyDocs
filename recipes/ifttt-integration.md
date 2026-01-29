---
title: IFTTT Applets
excerpt: Trigger IFTTT automations from aircraft events.
hidden: false
recipe:
  color: '#000000'
  icon: 🔗
difficulty: beginner
tags: [notifications, ifttt, automation, webhooks]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- IFTTT account (free tier works)
- IFTTT Webhooks service enabled

## What You'll Build

An integration that triggers IFTTT applets when aircraft events occur, connecting SkySpy to thousands of apps and smart devices.

```shell Shell
pip install sseclient-py requests
export IFTTT_KEY="your_ifttt_webhooks_key"
python ifttt_trigger.py
```

```python Python
import json
import os
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
IFTTT_KEY = os.getenv("IFTTT_KEY")
IFTTT_EVENT = os.getenv("IFTTT_EVENT", "aircraft_spotted")

if not IFTTT_KEY:
    print("Error: IFTTT_KEY environment variable required")
    sys.exit(1)


def trigger_ifttt(event_name, value1="", value2="", value3=""):
    """Trigger an IFTTT Webhooks event."""
    url = f"https://maker.ifttt.com/trigger/{event_name}/with/key/{IFTTT_KEY}"

    payload = {
        "value1": value1,
        "value2": value2,
        "value3": value3
    }

    response = requests.post(url, json=payload, timeout=10)
    return response.status_code == 200


def format_aircraft_message(aircraft):
    """Format aircraft data for IFTTT."""
    flight = aircraft.get("flight", "Unknown").strip()
    hex_code = aircraft.get("hex", "Unknown")
    aircraft_type = aircraft.get("type", "Unknown")
    altitude = aircraft.get("alt", 0)
    distance = aircraft.get("distance", 0)
    is_military = aircraft.get("military", False)
    squawk = aircraft.get("squawk", "")

    # value1: Main message
    if squawk in ["7700", "7600", "7500"]:
        value1 = f"🚨 EMERGENCY: {flight} squawking {squawk}"
    elif is_military:
        value1 = f"🎖️ Military: {flight} ({aircraft_type})"
    else:
        value1 = f"✈️ Aircraft: {flight} ({aircraft_type})"

    # value2: Details
    value2 = f"Alt: {altitude:,}ft, Dist: {distance:.1f}nm"

    # value3: Tracking link
    value3 = f"https://globe.adsbexchange.com/?icao={hex_code}"

    return value1, value2, value3


def main():
    print(f"IFTTT Integration connected to {SKYSPY_URL}")
    print(f"Triggering event: {IFTTT_EVENT}")
    alerted = set()

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    for aircraft in data.get("aircraft", []):
                        hex_code = aircraft.get("hex")
                        squawk = aircraft.get("squawk", "")
                        is_military = aircraft.get("military", False)

                        # Alert on military or emergency
                        should_alert = is_military or squawk in ["7700", "7600", "7500"]

                        if should_alert and hex_code not in alerted:
                            v1, v2, v3 = format_aircraft_message(aircraft)

                            # Trigger different events based on type
                            if squawk in ["7700", "7600", "7500"]:
                                event_name = "aircraft_emergency"
                            elif is_military:
                                event_name = "aircraft_military"
                            else:
                                event_name = IFTTT_EVENT

                            if trigger_ifttt(event_name, v1, v2, v3):
                                flight = aircraft.get("flight", hex_code)
                                print(f"✅ IFTTT triggered: {event_name} for {flight}")
                                alerted.add(hex_code)

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
	"bytes"
	"encoding/json"
	"fmt"
	"net/http"
	"os"
	"strings"
)

var alerted = make(map[string]bool)

func triggerIFTTT(key, event, v1, v2, v3 string) bool {
	url := fmt.Sprintf("https://maker.ifttt.com/trigger/%s/with/key/%s", event, key)

	payload := map[string]string{
		"value1": v1,
		"value2": v2,
		"value3": v3,
	}

	body, _ := json.Marshal(payload)
	resp, err := http.Post(url, "application/json", bytes.NewBuffer(body))
	if err != nil {
		return false
	}
	defer resp.Body.Close()
	return resp.StatusCode == 200
}

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	iftttKey := os.Getenv("IFTTT_KEY")
	if iftttKey == "" {
		fmt.Println("Error: IFTTT_KEY required")
		os.Exit(1)
	}

	defaultEvent := os.Getenv("IFTTT_EVENT")
	if defaultEvent == "" {
		defaultEvent = "aircraft_spotted"
	}

	fmt.Printf("IFTTT Integration connected to %s\n", skyspyURL)

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
						squawk := fmt.Sprint(ac["squawk"])
						isMilitary := ac["military"] == true
						flight := strings.TrimSpace(fmt.Sprint(ac["flight"]))

						shouldAlert := isMilitary || squawk == "7700" || squawk == "7600" || squawk == "7500"

						if shouldAlert && !alerted[hex] {
							var eventName, v1 string
							switch {
							case squawk == "7700" || squawk == "7600" || squawk == "7500":
								eventName = "aircraft_emergency"
								v1 = fmt.Sprintf("🚨 EMERGENCY: %s squawking %s", flight, squawk)
							case isMilitary:
								eventName = "aircraft_military"
								v1 = fmt.Sprintf("🎖️ Military: %s", flight)
							default:
								eventName = defaultEvent
								v1 = fmt.Sprintf("✈️ Aircraft: %s", flight)
							}

							v2 := fmt.Sprintf("Alt: %.0fft, Dist: %.1fnm", ac["alt"], ac["distance"])
							v3 := fmt.Sprintf("https://globe.adsbexchange.com/?icao=%s", hex)

							if triggerIFTTT(iftttKey, eventName, v1, v2, v3) {
								fmt.Printf("✅ IFTTT triggered: %s for %s\n", eventName, flight)
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
```

```javascript JavaScript
const EventSource = require('eventsource');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const IFTTT_KEY = process.env.IFTTT_KEY;
const IFTTT_EVENT = process.env.IFTTT_EVENT || 'aircraft_spotted';

if (!IFTTT_KEY) {
  console.error('Error: IFTTT_KEY environment variable required');
  process.exit(1);
}

const alerted = new Set();

async function triggerIFTTT(eventName, value1, value2, value3) {
  const url = `https://maker.ifttt.com/trigger/${eventName}/with/key/${IFTTT_KEY}`;

  const response = await fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ value1, value2, value3 })
  });

  return response.ok;
}

console.log(`IFTTT Integration connected to ${SKYSPY_URL}`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;
    const squawk = aircraft.squawk || '';
    const isMilitary = aircraft.military || false;
    const flight = (aircraft.flight || hex).trim();

    const shouldAlert = isMilitary || ['7700', '7600', '7500'].includes(squawk);

    if (shouldAlert && !alerted.has(hex)) {
      let eventName, v1;

      if (['7700', '7600', '7500'].includes(squawk)) {
        eventName = 'aircraft_emergency';
        v1 = `🚨 EMERGENCY: ${flight} squawking ${squawk}`;
      } else if (isMilitary) {
        eventName = 'aircraft_military';
        v1 = `🎖️ Military: ${flight}`;
      } else {
        eventName = IFTTT_EVENT;
        v1 = `✈️ Aircraft: ${flight}`;
      }

      const v2 = `Alt: ${(aircraft.alt || 0).toLocaleString()}ft, Dist: ${(aircraft.distance || 0).toFixed(1)}nm`;
      const v3 = `https://globe.adsbexchange.com/?icao=${hex}`;

      if (await triggerIFTTT(eventName, v1, v2, v3)) {
        console.log(`✅ IFTTT triggered: ${eventName} for ${flight}`);
        alerted.add(hex);
      }
    }
  }
});

es.onerror = (err) => console.error('SSE error:', err);
```

## Setting Up IFTTT Webhooks

### 1. Get Your Webhooks Key

1. Go to [IFTTT Webhooks](https://ifttt.com/maker_webhooks)
2. Click "Documentation"
3. Copy your key from the URL shown

### 2. Create Applets

Create applets with Webhooks as the trigger:

**Example: Phone Notification**
- **If**: Webhooks - Receive a web request
- **Event Name**: `aircraft_military`
- **Then**: Notifications - Send a notification
- **Message**: `{{Value1}} - {{Value2}}`

**Example: Smart Light Flash**
- **If**: Webhooks - Receive a web request
- **Event Name**: `aircraft_emergency`
- **Then**: Philips Hue - Flash lights

**Example: Log to Google Sheet**
- **If**: Webhooks - Receive a web request
- **Event Name**: `aircraft_spotted`
- **Then**: Google Sheets - Add row
- **Columns**: `{{OccurredAt}}`, `{{Value1}}`, `{{Value2}}`, `{{Value3}}`

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `IFTTT_KEY` | IFTTT Webhooks key | Required |
| `IFTTT_EVENT` | Default event name | `aircraft_spotted` |

## Event Names

The integration uses different event names based on aircraft type:

| Event | Trigger |
|-------|---------|
| `aircraft_emergency` | Squawk 7700, 7600, or 7500 |
| `aircraft_military` | Military aircraft detected |
| `aircraft_spotted` | Default for other aircraft |

Create applets for each event name you want to handle.

## Value Fields

IFTTT Webhooks support three value fields:

| Field | Content |
|-------|---------|
| `value1` | Main message with aircraft info |
| `value2` | Altitude and distance details |
| `value3` | Tracking URL |

## Popular Applet Ideas

| Applet | Description |
|--------|-------------|
| **Phone notification** | Push alert to mobile device |
| **Smart lights** | Flash Hue/LIFX lights |
| **Google Sheet log** | Append to spreadsheet |
| **Tweet** | Post to Twitter |
| **Email** | Send Gmail/Outlook email |
| **Slack/Discord** | Post to channel |
| **Calendar event** | Create Google Calendar entry |
| **Alexa announcement** | Speak via Echo device |

## Testing & Verification

1. **Create a test applet** with phone notification
2. **Start the integration**
3. **Wait for military aircraft** or modify filter
4. **Check IFTTT Activity** page for triggers
5. **Verify notification** arrives on phone

### Test Trigger

```shell
# Test your webhooks key
curl -X POST "https://maker.ifttt.com/trigger/test/with/key/$IFTTT_KEY" \
  -H "Content-Type: application/json" \
  -d '{"value1": "Test", "value2": "Message", "value3": "Link"}'
```

## Troubleshooting

### Applet Not Triggering

**Problem:** IFTTT shows "Never run"
**Solution:**
- Verify webhook key is correct
- Check event name matches exactly
- Ensure applet is connected and enabled
- Check IFTTT Activity for errors

### Rate Limits

**Problem:** Some events not triggering
**Solution:**
- IFTTT free tier has rate limits
- Add cooldown between triggers
- Consider IFTTT Pro for higher limits

### Delayed Notifications

**Problem:** Notifications arrive late
**Solution:**
- IFTTT free tier can have delays up to 15 minutes
- IFTTT Pro offers faster execution
- Consider alternative services for time-critical alerts

## Alternative Services

For faster/more reliable automations:
- [Zapier](/docs/zapier-integration) - More powerful, paid
- [n8n](/docs/n8n-workflow) - Self-hosted
- [Webhook Notifications](/docs/webhook-notifications) - Direct webhooks

## Related Recipes

- [Webhook Notifications](/docs/webhook-notifications) - Generic webhooks
- [Home Assistant](/docs/home-assistant) - Local automation
- [Discord Alert Bot](/docs/discord-alert-bot) - Direct notifications
- [Email Alerts](/docs/email-alerts) - SMTP email
