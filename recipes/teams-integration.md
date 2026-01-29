---
title: Microsoft Teams Integration
excerpt: Send aircraft alerts to Microsoft Teams channels.
hidden: false
recipe:
  color: '#6264A7'
  icon: 💼
difficulty: beginner
tags: [notifications, teams, microsoft, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Microsoft Teams workspace with permission to add connectors
- Teams Incoming Webhook URL

## What You'll Build

A notification system that sends rich aircraft alerts to Microsoft Teams channels using Adaptive Cards.

```shell Shell
pip install sseclient-py requests
export TEAMS_WEBHOOK="https://outlook.office.com/webhook/..."
python teams_alert.py
```

```python Python
import json
import os
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
TEAMS_WEBHOOK = os.getenv("TEAMS_WEBHOOK")

if not TEAMS_WEBHOOK:
    print("Error: TEAMS_WEBHOOK environment variable required")
    sys.exit(1)


def send_teams_alert(aircraft):
    """Send an aircraft alert to Microsoft Teams."""
    flight = aircraft.get("flight", "Unknown").strip()
    hex_code = aircraft.get("hex", "Unknown")
    aircraft_type = aircraft.get("type", "Unknown")
    altitude = aircraft.get("alt", 0)
    speed = aircraft.get("gs", 0)
    distance = aircraft.get("distance", 0)
    lat = aircraft.get("lat", 0)
    lon = aircraft.get("lon", 0)
    is_military = aircraft.get("military", False)
    squawk = aircraft.get("squawk", "")

    # Determine theme color
    if squawk in ["7700", "7600", "7500"]:
        theme_color = "FF0000"  # Red for emergency
        title = f"🚨 Emergency Aircraft: {flight}"
    elif is_military:
        theme_color = "FFA500"  # Orange for military
        title = f"🎖️ Military Aircraft: {flight}"
    else:
        theme_color = "0078D4"  # Blue for normal
        title = f"✈️ Aircraft Alert: {flight}"

    # Build Adaptive Card
    card = {
        "@type": "MessageCard",
        "@context": "http://schema.org/extensions",
        "themeColor": theme_color,
        "summary": title,
        "sections": [{
            "activityTitle": title,
            "facts": [
                {"name": "Callsign", "value": flight},
                {"name": "ICAO Hex", "value": hex_code},
                {"name": "Type", "value": aircraft_type},
                {"name": "Altitude", "value": f"{altitude:,} ft"},
                {"name": "Speed", "value": f"{speed} kts"},
                {"name": "Distance", "value": f"{distance:.1f} nm"},
                {"name": "Position", "value": f"{lat:.4f}, {lon:.4f}"},
            ],
            "markdown": True
        }],
        "potentialAction": [{
            "@type": "OpenUri",
            "name": "View on Map",
            "targets": [{
                "os": "default",
                "uri": f"https://globe.adsbexchange.com/?icao={hex_code}"
            }]
        }, {
            "@type": "OpenUri",
            "name": "FlightAware",
            "targets": [{
                "os": "default",
                "uri": f"https://flightaware.com/live/modes/{hex_code}"
            }]
        }]
    }

    if squawk:
        card["sections"][0]["facts"].append({"name": "Squawk", "value": squawk})

    if is_military:
        card["sections"][0]["facts"].append({"name": "Military", "value": "Yes"})

    response = requests.post(TEAMS_WEBHOOK, json=card, timeout=10)
    return response.status_code == 200


def main():
    print(f"Teams Alert Bot connected to {SKYSPY_URL}")
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
                            if send_teams_alert(aircraft):
                                flight = aircraft.get("flight", hex_code)
                                print(f"✅ Teams alert sent: {flight}")
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

type TeamsCard struct {
	Type       string         `json:"@type"`
	Context    string         `json:"@context"`
	ThemeColor string         `json:"themeColor"`
	Summary    string         `json:"summary"`
	Sections   []TeamsSection `json:"sections"`
	Actions    []TeamsAction  `json:"potentialAction,omitempty"`
}

type TeamsSection struct {
	ActivityTitle string      `json:"activityTitle"`
	Facts         []TeamsFact `json:"facts"`
	Markdown      bool        `json:"markdown"`
}

type TeamsFact struct {
	Name  string `json:"name"`
	Value string `json:"value"`
}

type TeamsAction struct {
	Type    string        `json:"@type"`
	Name    string        `json:"name"`
	Targets []TeamsTarget `json:"targets"`
}

type TeamsTarget struct {
	OS  string `json:"os"`
	URI string `json:"uri"`
}

var alerted = make(map[string]bool)

func sendTeamsAlert(webhook string, aircraft map[string]interface{}) bool {
	hex := fmt.Sprint(aircraft["hex"])
	flight := strings.TrimSpace(fmt.Sprint(aircraft["flight"]))
	if flight == "<nil>" {
		flight = hex
	}

	isMilitary := aircraft["military"] == true
	squawk := fmt.Sprint(aircraft["squawk"])

	var themeColor, title string
	switch {
	case squawk == "7700" || squawk == "7600" || squawk == "7500":
		themeColor = "FF0000"
		title = fmt.Sprintf("🚨 Emergency: %s", flight)
	case isMilitary:
		themeColor = "FFA500"
		title = fmt.Sprintf("🎖️ Military: %s", flight)
	default:
		themeColor = "0078D4"
		title = fmt.Sprintf("✈️ Aircraft: %s", flight)
	}

	card := TeamsCard{
		Type:       "MessageCard",
		Context:    "http://schema.org/extensions",
		ThemeColor: themeColor,
		Summary:    title,
		Sections: []TeamsSection{{
			ActivityTitle: title,
			Facts: []TeamsFact{
				{Name: "Callsign", Value: flight},
				{Name: "ICAO Hex", Value: hex},
				{Name: "Type", Value: fmt.Sprint(aircraft["type"])},
				{Name: "Altitude", Value: fmt.Sprintf("%.0f ft", aircraft["alt"])},
				{Name: "Speed", Value: fmt.Sprintf("%.0f kts", aircraft["gs"])},
				{Name: "Distance", Value: fmt.Sprintf("%.1f nm", aircraft["distance"])},
			},
			Markdown: true,
		}},
		Actions: []TeamsAction{{
			Type: "OpenUri",
			Name: "View on Map",
			Targets: []TeamsTarget{{
				OS:  "default",
				URI: fmt.Sprintf("https://globe.adsbexchange.com/?icao=%s", hex),
			}},
		}},
	}

	body, _ := json.Marshal(card)
	resp, err := http.Post(webhook, "application/json", bytes.NewBuffer(body))
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

	webhook := os.Getenv("TEAMS_WEBHOOK")
	if webhook == "" {
		fmt.Println("Error: TEAMS_WEBHOOK required")
		os.Exit(1)
	}

	fmt.Printf("Teams Alert Bot connected to %s\n", skyspyURL)

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

						shouldAlert := isMilitary || squawk == "7700" || squawk == "7600" || squawk == "7500"

						if shouldAlert && !alerted[hex] {
							if sendTeamsAlert(webhook, ac) {
								fmt.Printf("✅ Teams alert: %s\n", hex)
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
const TEAMS_WEBHOOK = process.env.TEAMS_WEBHOOK;

if (!TEAMS_WEBHOOK) {
  console.error('Error: TEAMS_WEBHOOK environment variable required');
  process.exit(1);
}

const alerted = new Set();

async function sendTeamsAlert(aircraft) {
  const flight = (aircraft.flight || aircraft.hex).trim();
  const hex = aircraft.hex;
  const isMilitary = aircraft.military || false;
  const squawk = aircraft.squawk || '';

  let themeColor, title;
  if (['7700', '7600', '7500'].includes(squawk)) {
    themeColor = 'FF0000';
    title = `🚨 Emergency Aircraft: ${flight}`;
  } else if (isMilitary) {
    themeColor = 'FFA500';
    title = `🎖️ Military Aircraft: ${flight}`;
  } else {
    themeColor = '0078D4';
    title = `✈️ Aircraft Alert: ${flight}`;
  }

  const card = {
    '@type': 'MessageCard',
    '@context': 'http://schema.org/extensions',
    themeColor,
    summary: title,
    sections: [{
      activityTitle: title,
      facts: [
        { name: 'Callsign', value: flight },
        { name: 'ICAO Hex', value: hex },
        { name: 'Type', value: aircraft.type || 'Unknown' },
        { name: 'Altitude', value: `${(aircraft.alt || 0).toLocaleString()} ft` },
        { name: 'Speed', value: `${aircraft.gs || 0} kts` },
        { name: 'Distance', value: `${(aircraft.distance || 0).toFixed(1)} nm` },
      ],
      markdown: true
    }],
    potentialAction: [{
      '@type': 'OpenUri',
      name: 'View on Map',
      targets: [{
        os: 'default',
        uri: `https://globe.adsbexchange.com/?icao=${hex}`
      }]
    }]
  };

  const response = await fetch(TEAMS_WEBHOOK, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(card)
  });

  return response.ok;
}

console.log(`Teams Alert Bot connected to ${SKYSPY_URL}`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;
    const squawk = aircraft.squawk || '';
    const isMilitary = aircraft.military || false;

    const shouldAlert = isMilitary || ['7700', '7600', '7500'].includes(squawk);

    if (shouldAlert && !alerted.has(hex)) {
      if (await sendTeamsAlert(aircraft)) {
        console.log(`✅ Teams alert: ${aircraft.flight || hex}`);
        alerted.add(hex);
      }
    }
  }
});

es.onerror = (err) => console.error('SSE error:', err);
```

## Setting Up Teams Webhook

1. **Open Microsoft Teams**
2. **Navigate to your channel** → Click "..." → "Connectors"
3. **Find "Incoming Webhook"** → Click "Configure"
4. **Name your webhook** (e.g., "SkySpy Alerts")
5. **Upload an icon** (optional)
6. **Copy the webhook URL**

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `TEAMS_WEBHOOK` | Teams Incoming Webhook URL | Required |

## Adaptive Card Features

The alerts use MessageCard format which supports:

- **Theme colors** - Red for emergencies, orange for military, blue for normal
- **Facts list** - Structured data display
- **Action buttons** - Links to tracking sites
- **Markdown** - Formatted text in descriptions

## Testing & Verification

1. **Set up webhook** and export the URL
2. **Start the alert bot**
3. **Wait for military or emergency aircraft**
4. **Check Teams channel** for the alert card
5. **Click action buttons** to verify links work

### Test with Any Aircraft

```python
# Temporarily alert on all aircraft
should_alert = True  # Instead of military/emergency filter
```

## Troubleshooting

### No Alerts Appearing

**Problem:** Bot is running but no Teams messages
**Solution:**
- Verify webhook URL is correct
- Check channel permissions for connectors
- Test webhook with curl:
  ```shell
  curl -X POST "$TEAMS_WEBHOOK" \
    -H "Content-Type: application/json" \
    -d '{"text": "Test message"}'
  ```

### Rate Limiting

**Problem:** Some alerts not appearing
**Solution:**
- Teams has rate limits on webhooks
- Add delays between messages
- Batch multiple aircraft into single cards

### Card Not Rendering

**Problem:** Message appears as raw JSON
**Solution:**
- Verify card schema is correct
- Use MessageCard format (not Adaptive Card v2)
- Check for required fields

## Related Recipes

- [Slack Integration](/docs/slack-integration) - Slack Block Kit messages
- [Discord Alert Bot](/docs/discord-alert-bot) - Discord webhooks
- [Email Alerts](/docs/email-alerts) - SMTP notifications
- [Webhook Notifications](/docs/webhook-notifications) - Generic webhooks
