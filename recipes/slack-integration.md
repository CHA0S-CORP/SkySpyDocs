---
title: Slack Integration
description: Send rich aircraft alerts to Slack channels with Block Kit formatting.
hidden: false
recipe:
  color: '#4A154B'
  icon: 💬
---
```shell Shell
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/T00/B00/XXX"
curl -X POST $SLACK_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d '{"text": "🎖️ Military Aircraft Detected: RCH419"}'
```

```go Go
package main

import (
	"bytes"
	"encoding/json"
	"net/http"
	"os"
)

func main() {
	webhookURL := os.Getenv("SLACK_WEBHOOK_URL")

	payload := map[string]interface{}{
		"blocks": []map[string]interface{}{
			{
				"type": "header",
				"text": map[string]string{
					"type": "plain_text",
					"text": "🎖️ Military Aircraft Detected",
				},
			},
			{
				"type": "section",
				"fields": []map[string]string{
					{"type": "mrkdwn", "text": "*Callsign:*\nRCH419"},
					{"type": "mrkdwn", "text": "*Type:*\nC-17"},
					{"type": "mrkdwn", "text": "*Altitude:*\n35,000 ft"},
					{"type": "mrkdwn", "text": "*Speed:*\n450 kts"},
				},
			},
		},
	}

	body, _ := json.Marshal(payload)
	http.Post(webhookURL, "application/json", bytes.NewReader(body))
}
```

```python Python
import json
import os
import requests

SLACK_WEBHOOK_URL = os.getenv("SLACK_WEBHOOK_URL")

def send_slack_alert(aircraft):
    payload = {
        "blocks": [
            {
                "type": "header",
                "text": {"type": "plain_text", "text": "🎖️ Military Aircraft Detected"}
            },
            {
                "type": "section",
                "fields": [
                    {"type": "mrkdwn", "text": f"*Callsign:*\n{aircraft.get('flight', 'N/A')}"},
                    {"type": "mrkdwn", "text": f"*Type:*\n{aircraft.get('type', 'Unknown')}"},
                    {"type": "mrkdwn", "text": f"*Altitude:*\n{aircraft.get('alt', 0):,} ft"},
                    {"type": "mrkdwn", "text": f"*Speed:*\n{aircraft.get('gs', 0)} kts"},
                ]
            }
        ]
    }
    requests.post(SLACK_WEBHOOK_URL, json=payload)

# Example usage
send_slack_alert({"flight": "RCH419", "type": "C-17", "alt": 35000, "gs": 450})
```

```javascript JavaScript
const SLACK_WEBHOOK_URL = process.env.SLACK_WEBHOOK_URL;

async function sendSlackAlert(aircraft) {
    const payload = {
        blocks: [
            {
                type: 'header',
                text: { type: 'plain_text', text: '🎖️ Military Aircraft Detected' }
            },
            {
                type: 'section',
                fields: [
                    { type: 'mrkdwn', text: `*Callsign:*\n${aircraft.flight || 'N/A'}` },
                    { type: 'mrkdwn', text: `*Type:*\n${aircraft.type || 'Unknown'}` },
                    { type: 'mrkdwn', text: `*Altitude:*\n${aircraft.alt?.toLocaleString() || 0} ft` },
                    { type: 'mrkdwn', text: `*Speed:*\n${aircraft.gs || 0} kts` },
                ]
            }
        ]
    };

    await fetch(SLACK_WEBHOOK_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
    });
}

// Example usage
sendSlackAlert({ flight: 'RCH419', type: 'C-17', alt: 35000, gs: 450 });
```

```json Response Example
{"ok": true}
```

# Configure Slack Webhook

<!-- shell@1-2 -->
<!-- go@1-12 -->
<!-- python@1-6 -->
<!-- javascript@1-3 -->

Create an Incoming Webhook in your Slack workspace at [api.slack.com/apps](https://api.slack.com/apps). Add the webhook URL to your environment variables.

# Build Block Kit Message

<!-- go@14-35 -->
<!-- python@8-23 -->
<!-- javascript@5-20 -->

Slack's Block Kit provides rich formatting. Create a header block for the alert type and a section with fields for aircraft details like callsign, type, altitude, and speed.

# Send to Slack

<!-- shell@3-4 -->
<!-- go@37-38 -->
<!-- python@24-25 -->
<!-- javascript@22-26 -->

POST the JSON payload to your webhook URL. Slack will format and display the message in your configured channel.
