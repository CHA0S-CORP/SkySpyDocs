---
title: Webhook Notifications
description: Send aircraft alerts to any webhook endpoint.
hidden: false
recipe:
  color: '#6366F1'
  icon: 🔗
---
```shell Shell
curl -X POST https://your-webhook.example.com/alerts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $WEBHOOK_TOKEN" \
  -d '{
    "event": "aircraft_detected",
    "aircraft": {
      "hex": "A12345",
      "flight": "RCH419",
      "type": "C-17",
      "alt": 35000,
      "gs": 450,
      "military": true
    }
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

type WebhookPayload struct {
	Event    string                 `json:"event"`
	Aircraft map[string]interface{} `json:"aircraft"`
}

func main() {
	webhookURL := os.Getenv("WEBHOOK_URL")
	token := os.Getenv("WEBHOOK_TOKEN")

	payload := WebhookPayload{
		Event: "aircraft_detected",
		Aircraft: map[string]interface{}{
			"hex":      "A12345",
			"flight":   "RCH419",
			"type":     "C-17",
			"alt":      35000,
			"gs":       450,
			"military": true,
		},
	}

	body, _ := json.Marshal(payload)
	req, _ := http.NewRequest("POST", webhookURL, bytes.NewReader(body))
	req.Header.Set("Content-Type", "application/json")
	req.Header.Set("Authorization", "Bearer "+token)

	http.DefaultClient.Do(req)
}
```

```python Python
import os
import requests

WEBHOOK_URL = os.getenv("WEBHOOK_URL")
WEBHOOK_TOKEN = os.getenv("WEBHOOK_TOKEN")

def send_webhook(event, aircraft):
    payload = {
        "event": event,
        "aircraft": aircraft
    }
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {WEBHOOK_TOKEN}"
    }
    response = requests.post(WEBHOOK_URL, json=payload, headers=headers)
    return response.json()

# Example usage
result = send_webhook("aircraft_detected", {
    "hex": "A12345",
    "flight": "RCH419",
    "type": "C-17",
    "alt": 35000,
    "gs": 450,
    "military": True
})
```

```javascript JavaScript
const WEBHOOK_URL = process.env.WEBHOOK_URL;
const WEBHOOK_TOKEN = process.env.WEBHOOK_TOKEN;

async function sendWebhook(event, aircraft) {
    const response = await fetch(WEBHOOK_URL, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${WEBHOOK_TOKEN}`,
        },
        body: JSON.stringify({ event, aircraft }),
    });
    return response.json();
}

// Example usage
sendWebhook('aircraft_detected', {
    hex: 'A12345',
    flight: 'RCH419',
    type: 'C-17',
    alt: 35000,
    gs: 450,
    military: true,
});
```

```json Response Example
{"success": true, "id": "evt_123abc", "received_at": "2024-01-15T10:30:00Z"}
```

# Configure Webhook Endpoint

<!-- shell@1-3 -->
<!-- go@1-11 -->
<!-- python@1-5 -->
<!-- javascript@1-2 -->

Set your webhook URL and authentication token. Most webhook receivers expect a Bearer token or API key for authentication.

# Build Payload

<!-- shell@4-13 -->
<!-- go@13-28 -->
<!-- python@7-12 -->
<!-- javascript@4-12 -->

Structure the payload with an event type and aircraft data. Include all relevant fields: hex code, callsign, aircraft type, altitude, speed, and any flags like military status.

# Send POST Request

<!-- shell@14 -->
<!-- go@30-35 -->
<!-- python@13-17 -->
<!-- javascript@13-14 -->

POST the JSON payload with proper headers. Handle the response to confirm delivery or retry on failure.
