---
title: Microsoft Teams
description: Send aircraft alerts to Microsoft Teams channels.
hidden: false
recipe:
  color: '#5558AF'
  icon: 💼
---
```shell Shell
curl -X POST $TEAMS_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d '{
    "@type": "MessageCard",
    "summary": "Aircraft Alert",
    "themeColor": "0076D7",
    "title": "Military Aircraft Detected",
    "sections": [{
      "facts": [
        {"name": "Callsign", "value": "RCH419"},
        {"name": "Type", "value": "C-17"},
        {"name": "Altitude", "value": "35,000 ft"}
      ]
    }]
  }'
```

```go Go
package main

import (
	"bytes"
	"encoding/json"
	"net/http"
	"os"
)

type TeamsCard struct {
	Type       string    `json:"@type"`
	Summary    string    `json:"summary"`
	ThemeColor string    `json:"themeColor"`
	Title      string    `json:"title"`
	Sections   []Section `json:"sections"`
}

type Section struct {
	Facts []Fact `json:"facts"`
}

type Fact struct {
	Name  string `json:"name"`
	Value string `json:"value"`
}

func sendTeamsAlert(aircraft map[string]interface{}) {
	webhookURL := os.Getenv("TEAMS_WEBHOOK_URL")

	card := TeamsCard{
		Type:       "MessageCard",
		Summary:    "Aircraft Alert",
		ThemeColor: "0076D7",
		Title:      "Military Aircraft Detected",
		Sections: []Section{{
			Facts: []Fact{
				{Name: "Callsign", Value: aircraft["flight"].(string)},
				{Name: "Type", Value: aircraft["type"].(string)},
				{Name: "Altitude", Value: "35,000 ft"},
			},
		}},
	}

	body, _ := json.Marshal(card)
	http.Post(webhookURL, "application/json", bytes.NewReader(body))
}
```

```python Python
import os
import requests

TEAMS_WEBHOOK_URL = os.getenv("TEAMS_WEBHOOK_URL")

def send_teams_alert(aircraft):
    card = {
        "@type": "MessageCard",
        "summary": "Aircraft Alert",
        "themeColor": "0076D7",
        "title": f"Military Aircraft: {aircraft.get('flight', 'Unknown')}",
        "sections": [{
            "facts": [
                {"name": "Callsign", "value": aircraft.get("flight", "N/A")},
                {"name": "Type", "value": aircraft.get("type", "Unknown")},
                {"name": "Altitude", "value": f"{aircraft.get('alt', 0):,} ft"},
                {"name": "Speed", "value": f"{aircraft.get('gs', 0)} kts"},
            ]
        }]
    }
    requests.post(TEAMS_WEBHOOK_URL, json=card)

# Example usage
send_teams_alert({"flight": "RCH419", "type": "C-17", "alt": 35000, "gs": 450})
```

```javascript JavaScript
const TEAMS_WEBHOOK_URL = process.env.TEAMS_WEBHOOK_URL;

async function sendTeamsAlert(aircraft) {
    const card = {
        '@type': 'MessageCard',
        'summary': 'Aircraft Alert',
        'themeColor': '0076D7',
        'title': `Military Aircraft: ${aircraft.flight || 'Unknown'}`,
        'sections': [{
            'facts': [
                { name: 'Callsign', value: aircraft.flight || 'N/A' },
                { name: 'Type', value: aircraft.type || 'Unknown' },
                { name: 'Altitude', value: `${aircraft.alt?.toLocaleString() || 0} ft` },
                { name: 'Speed', value: `${aircraft.gs || 0} kts` },
            ]
        }]
    };

    await fetch(TEAMS_WEBHOOK_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(card),
    });
}

// Example usage
sendTeamsAlert({ flight: 'RCH419', type: 'C-17', alt: 35000, gs: 450 });
```

```json Response Example
1
```

# Configure Teams Webhook

<!-- shell@1 -->
<!-- go@1-9 -->
<!-- python@1-4 -->
<!-- javascript@1 -->

Create an Incoming Webhook in your Teams channel. Go to channel settings > Connectors > Incoming Webhook.

# Build Message Card

<!-- shell@3-15 -->
<!-- go@27-41 -->
<!-- python@7-20 -->
<!-- javascript@4-17 -->

Use the MessageCard format with facts for structured data display. Set themeColor to match alert priority.

# Send to Teams

<!-- go@43-44 -->
<!-- python@21 -->
<!-- javascript@19-23 -->

POST the card JSON to your webhook URL. Teams will render it as a rich card in the channel.
