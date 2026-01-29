---
title: Zapier Webhooks
excerpt: Connect SkySpy to 5000+ apps via Zapier.
hidden: false
recipe:
  color: '#FF4A00'
  icon: ⚡
difficulty: beginner
tags: [notifications, zapier, automation, webhooks]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Zapier account (free tier works for testing)
- Zapier Webhooks by Zapier app enabled

## What You'll Build

An integration that triggers Zapier Zaps when aircraft events occur, connecting SkySpy to 5000+ apps including Google Sheets, Slack, Gmail, and more.

```shell Shell
pip install sseclient-py requests
export ZAPIER_WEBHOOK_URL="https://hooks.zapier.com/hooks/catch/123456/abcdef/"
python zapier_trigger.py
```

```python Python
import json
import os
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
ZAPIER_WEBHOOK_URL = os.getenv("ZAPIER_WEBHOOK_URL")

if not ZAPIER_WEBHOOK_URL:
    print("Error: ZAPIER_WEBHOOK_URL environment variable required")
    sys.exit(1)


def trigger_zapier(payload):
    """Send data to Zapier webhook."""
    response = requests.post(ZAPIER_WEBHOOK_URL, json=payload, timeout=10)
    return response.status_code == 200


def format_aircraft_payload(aircraft, event_type):
    """Format aircraft data for Zapier."""
    flight = aircraft.get("flight", "Unknown").strip()
    hex_code = aircraft.get("hex", "Unknown")
    aircraft_type = aircraft.get("type", "Unknown")
    altitude = aircraft.get("alt", 0)
    distance = aircraft.get("distance", 0)
    squawk = aircraft.get("squawk", "")
    latitude = aircraft.get("lat", 0)
    longitude = aircraft.get("lon", 0)

    return {
        "event_type": event_type,
        "flight": flight,
        "hex": hex_code,
        "aircraft_type": aircraft_type,
        "altitude": altitude,
        "distance": distance,
        "squawk": squawk,
        "latitude": latitude,
        "longitude": longitude,
        "is_military": aircraft.get("military", False),
        "tracking_url": f"https://globe.adsbexchange.com/?icao={hex_code}"
    }


def main():
    print(f"Zapier Integration connected to {SKYSPY_URL}")
    print(f"Webhook URL: {ZAPIER_WEBHOOK_URL[:50]}...")
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

                        # Determine event type
                        if squawk in ["7700", "7600", "7500"]:
                            event_type = "emergency"
                        elif is_military:
                            event_type = "military"
                        else:
                            continue  # Skip non-interesting aircraft

                        if hex_code not in alerted:
                            payload = format_aircraft_payload(aircraft, event_type)

                            if trigger_zapier(payload):
                                flight = aircraft.get("flight", hex_code)
                                print(f"Zapier triggered: {event_type} for {flight}")
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

type AircraftPayload struct {
	EventType    string  `json:"event_type"`
	Flight       string  `json:"flight"`
	Hex          string  `json:"hex"`
	AircraftType string  `json:"aircraft_type"`
	Altitude     float64 `json:"altitude"`
	Distance     float64 `json:"distance"`
	Squawk       string  `json:"squawk"`
	Latitude     float64 `json:"latitude"`
	Longitude    float64 `json:"longitude"`
	IsMilitary   bool    `json:"is_military"`
	TrackingURL  string  `json:"tracking_url"`
}

func triggerZapier(webhookURL string, payload AircraftPayload) bool {
	body, _ := json.Marshal(payload)
	resp, err := http.Post(webhookURL, "application/json", bytes.NewBuffer(body))
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

	zapierWebhookURL := os.Getenv("ZAPIER_WEBHOOK_URL")
	if zapierWebhookURL == "" {
		fmt.Println("Error: ZAPIER_WEBHOOK_URL required")
		os.Exit(1)
	}

	fmt.Printf("Zapier Integration connected to %s\n", skyspyURL)

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

						var eventType string
						switch {
						case squawk == "7700" || squawk == "7600" || squawk == "7500":
							eventType = "emergency"
						case isMilitary:
							eventType = "military"
						default:
							continue
						}

						if !alerted[hex] {
							payload := AircraftPayload{
								EventType:    eventType,
								Flight:       flight,
								Hex:          hex,
								AircraftType: fmt.Sprint(ac["type"]),
								Altitude:     ac["alt"].(float64),
								Distance:     ac["distance"].(float64),
								Squawk:       squawk,
								Latitude:     ac["lat"].(float64),
								Longitude:    ac["lon"].(float64),
								IsMilitary:   isMilitary,
								TrackingURL:  fmt.Sprintf("https://globe.adsbexchange.com/?icao=%s", hex),
							}

							if triggerZapier(zapierWebhookURL, payload) {
								fmt.Printf("Zapier triggered: %s for %s\n", eventType, flight)
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
const ZAPIER_WEBHOOK_URL = process.env.ZAPIER_WEBHOOK_URL;

if (!ZAPIER_WEBHOOK_URL) {
  console.error('Error: ZAPIER_WEBHOOK_URL environment variable required');
  process.exit(1);
}

const alerted = new Set();

async function triggerZapier(payload) {
  const response = await fetch(ZAPIER_WEBHOOK_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  });

  return response.ok;
}

function formatAircraftPayload(aircraft, eventType) {
  const hex = aircraft.hex;
  return {
    event_type: eventType,
    flight: (aircraft.flight || hex).trim(),
    hex: hex,
    aircraft_type: aircraft.type || 'Unknown',
    altitude: aircraft.alt || 0,
    distance: aircraft.distance || 0,
    squawk: aircraft.squawk || '',
    latitude: aircraft.lat || 0,
    longitude: aircraft.lon || 0,
    is_military: aircraft.military || false,
    tracking_url: `https://globe.adsbexchange.com/?icao=${hex}`
  };
}

console.log(`Zapier Integration connected to ${SKYSPY_URL}`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;
    const squawk = aircraft.squawk || '';
    const isMilitary = aircraft.military || false;

    let eventType;
    if (['7700', '7600', '7500'].includes(squawk)) {
      eventType = 'emergency';
    } else if (isMilitary) {
      eventType = 'military';
    } else {
      continue;
    }

    if (!alerted.has(hex)) {
      const payload = formatAircraftPayload(aircraft, eventType);

      if (await triggerZapier(payload)) {
        console.log(`Zapier triggered: ${eventType} for ${payload.flight}`);
        alerted.add(hex);
      }
    }
  }
});

es.onerror = (err) => console.error('SSE error:', err);
```

## Setting Up Zapier Webhooks

### 1. Create a New Zap

1. Go to [Zapier](https://zapier.com) and click "Create Zap"
2. For the trigger, search for "Webhooks by Zapier"
3. Select "Catch Hook" as the trigger event
4. Copy the webhook URL provided (looks like `https://hooks.zapier.com/hooks/catch/...`)

### 2. Configure Your Environment

Set the webhook URL as an environment variable:

```shell
export ZAPIER_WEBHOOK_URL="https://hooks.zapier.com/hooks/catch/123456/abcdef/"
```

### 3. Test the Connection

Run your script and trigger a test event. Zapier will capture the data and show you the available fields.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `ZAPIER_WEBHOOK_URL` | Zapier Webhooks catch URL | Required |

## Webhook Payload Fields

The integration sends these fields to Zapier:

| Field | Type | Description |
|-------|------|-------------|
| `event_type` | string | `emergency`, `military`, or `aircraft` |
| `flight` | string | Flight number or callsign |
| `hex` | string | ICAO hex code |
| `aircraft_type` | string | Aircraft type code |
| `altitude` | number | Altitude in feet |
| `distance` | number | Distance in nautical miles |
| `squawk` | string | Transponder squawk code |
| `latitude` | number | Current latitude |
| `longitude` | number | Current longitude |
| `is_military` | boolean | Military aircraft flag |
| `tracking_url` | string | ADS-B Exchange tracking link |

## Example Zaps

### Google Sheets Logging

Log all aircraft events to a spreadsheet:

1. **Trigger**: Webhooks by Zapier - Catch Hook
2. **Action**: Google Sheets - Create Spreadsheet Row
3. **Spreadsheet**: "Aircraft Sightings"
4. **Columns**:
   - Date: `{{zap_meta_human_now}}`
   - Event: `{{event_type}}`
   - Flight: `{{flight}}`
   - Type: `{{aircraft_type}}`
   - Altitude: `{{altitude}}`
   - Link: `{{tracking_url}}`

### Email Alerts

Send email notifications for emergencies:

1. **Trigger**: Webhooks by Zapier - Catch Hook
2. **Filter**: Only continue if `event_type` equals `emergency`
3. **Action**: Gmail - Send Email
4. **To**: your-email@example.com
5. **Subject**: `Aircraft Emergency: {{flight}}`
6. **Body**:
   ```
   Emergency aircraft detected!

   Flight: {{flight}}
   Squawk: {{squawk}}
   Altitude: {{altitude}} ft

   Track: {{tracking_url}}
   ```

### Slack Channel Notification

Post military aircraft to a Slack channel:

1. **Trigger**: Webhooks by Zapier - Catch Hook
2. **Filter**: Only continue if `is_military` is `true`
3. **Action**: Slack - Send Channel Message
4. **Channel**: #aircraft-alerts
5. **Message**:
   ```
   Military aircraft spotted!
   Flight: {{flight}}
   Type: {{aircraft_type}}
   Altitude: {{altitude}} ft
   Distance: {{distance}} nm
   <{{tracking_url}}|Track on ADS-B Exchange>
   ```

### Multi-Step Zap Example

Combine multiple actions:

1. **Trigger**: Webhooks by Zapier - Catch Hook
2. **Filter**: Only continue if `event_type` equals `emergency`
3. **Action 1**: Slack - Send Channel Message
4. **Action 2**: Gmail - Send Email
5. **Action 3**: Google Sheets - Create Spreadsheet Row
6. **Action 4**: Twilio - Send SMS

## Testing & Verification

1. **Start the integration** with your webhook URL
2. **Check Zapier** for "Test trigger" option
3. **Send a test request** manually:

```shell
curl -X POST "$ZAPIER_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "event_type": "military",
    "flight": "TEST123",
    "hex": "ABC123",
    "aircraft_type": "F16",
    "altitude": 25000,
    "distance": 5.2,
    "is_military": true,
    "tracking_url": "https://globe.adsbexchange.com/?icao=ABC123"
  }'
```

4. **Verify Zap executes** in Zapier Task History
5. **Check destination app** (Sheets, email, Slack, etc.)

## Troubleshooting

### Webhook Not Triggering

**Problem:** Zapier shows no incoming data
**Solution:**
- Verify the webhook URL is correct and complete
- Check the URL hasn't expired (recreate if needed)
- Ensure the Zap is turned on
- Check Zapier Task History for errors

### Data Not Mapping Correctly

**Problem:** Fields show as empty in Zapier
**Solution:**
- Send a test request to refresh available fields
- Re-map fields in the Zap editor
- Check field names match exactly (case-sensitive)

### Rate Limits

**Problem:** Some events not processing
**Solution:**
- Zapier free tier: 100 tasks/month
- Add filtering to reduce triggers
- Consider Zapier paid plans for higher limits
- Implement cooldown in your script

### Zap Errors

**Problem:** Task shows "Errored" status
**Solution:**
- Check Task History for error details
- Verify connected app credentials
- Test each action step individually
- Check app-specific rate limits (Gmail, Slack, etc.)

## Zapier vs IFTTT

| Feature | Zapier | IFTTT |
|---------|--------|-------|
| Apps | 5000+ | 700+ |
| Multi-step | Yes | Pro only |
| Filters | Yes | Limited |
| Custom fields | Unlimited | 3 values |
| Free tier | 100 tasks/mo | Limited |
| Speed | Fast | Can be delayed |

## Related Recipes

- [IFTTT Applets](/docs/ifttt-integration) - Simpler automation alternative
- [Webhook Notifications](/docs/webhook-notifications) - Generic webhooks
- [Slack Integration](/docs/slack-integration) - Direct Slack integration
- [Email Alerts](/docs/email-alerts) - Direct SMTP email
