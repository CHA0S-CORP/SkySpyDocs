---
title: Webhook Notifications
excerpt: Send aircraft alerts to any webhook endpoint.
hidden: false
recipe:
  color: '#6366F1'
  icon: 🔗
difficulty: beginner
tags: [notifications, webhook, integration, api]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- A webhook endpoint (your own server, Zapier, n8n, etc.)

## What You'll Build

A generic webhook sender that posts aircraft data to any HTTP endpoint. Perfect for integrating with custom systems, automation platforms, or building your own notification pipeline.

```shell Shell
export WEBHOOK_URL="https://your-server.com/webhook"
python webhook_sender.py
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
    "time"
)

type WebhookPayload struct {
    Event     string                 `json:"event"`
    Timestamp string                 `json:"timestamp"`
    Aircraft  map[string]interface{} `json:"aircraft"`
}

func main() {
    webhookURL := os.Getenv("WEBHOOK_URL")
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    if webhookURL == "" {
        fmt.Println("Error: WEBHOOK_URL required")
        os.Exit(1)
    }

    alerted := make(map[string]bool)
    client := &http.Client{Timeout: 10 * time.Second}

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
                        hex := fmt.Sprint(ac["hex"])

                        if alerted[hex] || ac["military"] != true {
                            continue
                        }

                        payload := WebhookPayload{
                            Event:     "military_aircraft",
                            Timestamp: time.Now().UTC().Format(time.RFC3339),
                            Aircraft:  ac,
                        }

                        body, _ := json.Marshal(payload)
                        req, _ := http.NewRequest("POST", webhookURL, bytes.NewReader(body))
                        req.Header.Set("Content-Type", "application/json")
                        req.Header.Set("X-SkySpy-Event", "military_aircraft")

                        client.Do(req)
                        alerted[hex] = true
                        fmt.Printf("Sent webhook: %s\n", ac["flight"])
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
from datetime import datetime
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
WEBHOOK_URL = os.getenv("WEBHOOK_URL")
WEBHOOK_SECRET = os.getenv("WEBHOOK_SECRET", "")

if not WEBHOOK_URL:
    print("Error: WEBHOOK_URL required")
    sys.exit(1)

alerted = set()


def send_webhook(event_type, aircraft):
    hex_code = aircraft.get("hex")

    payload = {
        "event": event_type,
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "aircraft": {
            "hex": hex_code,
            "flight": aircraft.get("flight"),
            "type": aircraft.get("type"),
            "altitude": aircraft.get("alt"),
            "speed": aircraft.get("gs"),
            "track": aircraft.get("track"),
            "latitude": aircraft.get("lat"),
            "longitude": aircraft.get("lon"),
            "distance": aircraft.get("distance"),
            "military": aircraft.get("military", False),
            "squawk": aircraft.get("squawk"),
            "category": aircraft.get("category"),
        }
    }

    headers = {
        "Content-Type": "application/json",
        "X-SkySpy-Event": event_type,
        "X-SkySpy-Timestamp": payload["timestamp"],
    }

    if WEBHOOK_SECRET:
        import hmac
        import hashlib
        signature = hmac.new(
            WEBHOOK_SECRET.encode(),
            json.dumps(payload).encode(),
            hashlib.sha256
        ).hexdigest()
        headers["X-SkySpy-Signature"] = f"sha256={signature}"

    try:
        response = requests.post(
            WEBHOOK_URL,
            json=payload,
            headers=headers,
            timeout=10
        )
        if response.status_code in [200, 201, 202, 204]:
            print(f"Webhook sent: {event_type} - {aircraft.get('flight', hex_code)}")
        else:
            print(f"Webhook failed: {response.status_code}")
    except Exception as e:
        print(f"Webhook error: {e}")


def main():
    print(f"Webhook sender connected to {SKYSPY_URL}...")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    for aircraft in data.get("aircraft", []):
                        hex_code = aircraft.get("hex")
                        if hex_code in alerted:
                            continue

                        squawk = aircraft.get("squawk", "")

                        # Emergency squawks
                        if squawk in ["7700", "7600", "7500"]:
                            send_webhook("emergency_squawk", aircraft)
                            alerted.add(hex_code)
                        # Military aircraft
                        elif aircraft.get("military"):
                            send_webhook("military_aircraft", aircraft)
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
const fetch = require('node-fetch');
const crypto = require('crypto');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const WEBHOOK_URL = process.env.WEBHOOK_URL;
const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET || '';

if (!WEBHOOK_URL) {
  console.error('Error: WEBHOOK_URL required');
  process.exit(1);
}

const alerted = new Set();

async function sendWebhook(eventType, aircraft) {
  const payload = {
    event: eventType,
    timestamp: new Date().toISOString(),
    aircraft: {
      hex: aircraft.hex,
      flight: aircraft.flight,
      type: aircraft.type,
      altitude: aircraft.alt,
      speed: aircraft.gs,
      track: aircraft.track,
      latitude: aircraft.lat,
      longitude: aircraft.lon,
      distance: aircraft.distance,
      military: aircraft.military || false,
      squawk: aircraft.squawk
    }
  };

  const headers = {
    'Content-Type': 'application/json',
    'X-SkySpy-Event': eventType,
    'X-SkySpy-Timestamp': payload.timestamp
  };

  if (WEBHOOK_SECRET) {
    const signature = crypto
      .createHmac('sha256', WEBHOOK_SECRET)
      .update(JSON.stringify(payload))
      .digest('hex');
    headers['X-SkySpy-Signature'] = `sha256=${signature}`;
  }

  try {
    const response = await fetch(WEBHOOK_URL, {
      method: 'POST',
      headers,
      body: JSON.stringify(payload)
    });
    console.log(`Webhook sent: ${eventType} - ${aircraft.flight || aircraft.hex}`);
  } catch (err) {
    console.error(`Webhook error: ${err.message}`);
  }
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;

    if (aircraft.military) {
      await sendWebhook('military_aircraft', aircraft);
      alerted.add(aircraft.hex);
    }
  }
});
```

```json Response Example
{
  "event": "military_aircraft",
  "timestamp": "2024-01-15T14:32:18.000Z",
  "aircraft": {
    "hex": "AE1234",
    "flight": "RCH419",
    "type": "C17",
    "altitude": 28000,
    "speed": 380,
    "track": 270,
    "latitude": 40.7128,
    "longitude": -74.006,
    "distance": 12.4,
    "military": true,
    "squawk": null
  }
}
```

## Webhook Payload Structure

Every webhook includes:
- `event`: Event type (e.g., `military_aircraft`, `emergency_squawk`)
- `timestamp`: ISO 8601 timestamp
- `aircraft`: Full aircraft data object

## Security with HMAC Signatures

<!-- python@43-53 -->
<!-- javascript@35-45 -->

Sign webhooks for verification. The receiving server can validate:

```python
import hmac
import hashlib

def verify_signature(payload, signature, secret):
    expected = hmac.new(
        secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f"sha256={expected}", signature)
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `WEBHOOK_URL` | Target webhook URL (required) | None |
| `WEBHOOK_SECRET` | HMAC signing secret | None (optional) |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Event Types

| Event | Description |
|-------|-------------|
| `military_aircraft` | Military aircraft detected |
| `emergency_squawk` | 7700/7600/7500 squawk |
| `aircraft_new` | New aircraft in range |
| `aircraft_lost` | Aircraft left coverage |

### Custom Event Filters

```python
# Only send for specific conditions
def should_send(aircraft):
    return (
        aircraft.get("military") or
        aircraft.get("squawk") in ["7700", "7600", "7500"] or
        aircraft.get("alt", 99999) < 1000 or
        aircraft.get("distance", 999) < 5
    )
```

### Retry Logic

Add exponential backoff for reliability:

```python
import time

def send_with_retry(url, payload, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.post(url, json=payload, timeout=10)
            if response.status_code < 500:
                return response
        except requests.RequestException:
            pass
        time.sleep(2 ** attempt)
    return None
```

## Receiving Webhooks

Example Express.js receiver:

```javascript
const express = require('express');
const crypto = require('crypto');

const app = express();
app.use(express.json());

app.post('/webhook', (req, res) => {
  // Verify signature
  const signature = req.headers['x-skyspy-signature'];
  if (signature) {
    const expected = crypto
      .createHmac('sha256', process.env.WEBHOOK_SECRET)
      .update(JSON.stringify(req.body))
      .digest('hex');
    if (!crypto.timingSafeEqual(
      Buffer.from(signature),
      Buffer.from(`sha256=${expected}`)
    )) {
      return res.status(401).send('Invalid signature');
    }
  }

  // Process the webhook
  console.log('Received:', req.body);
  res.status(200).send('OK');
});

app.listen(3000);
```

## Testing & Verification

1. **Use webhook.site** for testing:
   ```shell
   export WEBHOOK_URL="https://webhook.site/your-unique-id"
   ```
2. **Start the sender** and wait for aircraft
3. **Check webhook.site** for incoming requests
4. **Verify payload structure** matches your expectations

## Troubleshooting

### Webhooks Not Arriving

**Problem:** No requests at destination
**Solution:**
- Check URL is correct and reachable
- Verify firewall isn't blocking outbound requests
- Check for HTTPS certificate issues

### Timeout Errors

**Problem:** `Connection timed out`
**Solution:**
- Increase timeout value
- Check destination server is responding
- Implement retry logic

### Signature Mismatch

**Problem:** Signature validation fails
**Solution:**
- Verify secret matches on both ends
- Ensure payload JSON serialization is identical
- Check for encoding issues (UTF-8)

## Related Recipes

- [IFTTT Applets](/docs/ifttt-integration) - IFTTT webhooks
- [Zapier Webhooks](/docs/zapier-integration) - Zapier automation
