---
title: Slack Integration
excerpt: Send rich aircraft alerts to Slack channels with Block Kit formatting.
hidden: false
recipe:
  color: '#4A154B'
  icon: 💬
difficulty: beginner
tags: [notifications, slack, webhooks, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Slack workspace with permission to add apps
- Incoming Webhook URL from Slack
- Python: `pip install sseclient-py requests`

## What You'll Build

Rich Slack notifications with Block Kit formatting, including aircraft details, action buttons, and visual formatting.

```shell Shell
pip install sseclient-py requests
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/T.../B.../..."
python slack_bot.py
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
    webhookURL := os.Getenv("SLACK_WEBHOOK_URL")
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    if webhookURL == "" {
        fmt.Println("Error: SLACK_WEBHOOK_URL required")
        os.Exit(1)
    }

    alerted := make(map[string]bool)

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

                        payload := map[string]interface{}{
                            "blocks": []map[string]interface{}{
                                {
                                    "type": "header",
                                    "text": map[string]string{
                                        "type": "plain_text",
                                        "text": fmt.Sprintf("🎖️ Military: %s", ac["flight"]),
                                    },
                                },
                                {
                                    "type": "section",
                                    "fields": []map[string]string{
                                        {"type": "mrkdwn", "text": fmt.Sprintf("*Type:*\n%s", ac["type"])},
                                        {"type": "mrkdwn", "text": fmt.Sprintf("*Altitude:*\n%v ft", ac["alt"])},
                                        {"type": "mrkdwn", "text": fmt.Sprintf("*Speed:*\n%v kts", ac["gs"])},
                                        {"type": "mrkdwn", "text": fmt.Sprintf("*ICAO:*\n%s", hex)},
                                    },
                                },
                            },
                        }

                        body, _ := json.Marshal(payload)
                        http.Post(webhookURL, "application/json", bytes.NewReader(body))
                        alerted[hex] = true
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
WEBHOOK_URL = os.getenv("SLACK_WEBHOOK_URL")

if not WEBHOOK_URL:
    print("Error: SLACK_WEBHOOK_URL required")
    sys.exit(1)

alerted = set()


def send_slack_alert(aircraft):
    hex_code = aircraft.get("hex")
    flight = aircraft.get("flight", hex_code)

    blocks = [
        {
            "type": "header",
            "text": {
                "type": "plain_text",
                "text": f"🎖️ Military Aircraft: {flight}",
                "emoji": True
            }
        },
        {
            "type": "section",
            "fields": [
                {"type": "mrkdwn", "text": f"*Type:*\n{aircraft.get('type', 'Unknown')}"},
                {"type": "mrkdwn", "text": f"*Altitude:*\n{aircraft.get('alt', 0):,} ft"},
                {"type": "mrkdwn", "text": f"*Speed:*\n{aircraft.get('gs', 0)} kts"},
                {"type": "mrkdwn", "text": f"*Distance:*\n{aircraft.get('distance', 0):.1f} nm"},
            ]
        },
        {
            "type": "context",
            "elements": [
                {"type": "mrkdwn", "text": f"ICAO: `{hex_code}`"}
            ]
        },
        {
            "type": "actions",
            "elements": [
                {
                    "type": "button",
                    "text": {"type": "plain_text", "text": "FlightRadar24"},
                    "url": f"https://www.flightradar24.com/{hex_code}"
                },
                {
                    "type": "button",
                    "text": {"type": "plain_text", "text": "ADS-B Exchange"},
                    "url": f"https://globe.adsbexchange.com/?icao={hex_code}"
                }
            ]
        },
        {"type": "divider"}
    ]

    response = requests.post(WEBHOOK_URL, json={"blocks": blocks})
    if response.status_code == 200:
        print(f"Alerted: {flight} ({hex_code})")
    else:
        print(f"Failed to send: {response.text}")


def main():
    print(f"Slack bot connected to {SKYSPY_URL}...")

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
                        if aircraft.get("military"):
                            send_slack_alert(aircraft)
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

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const WEBHOOK_URL = process.env.SLACK_WEBHOOK_URL;

if (!WEBHOOK_URL) {
  console.error('Error: SLACK_WEBHOOK_URL required');
  process.exit(1);
}

const alerted = new Set();

async function sendSlackAlert(aircraft) {
  const blocks = [
    {
      type: 'header',
      text: {
        type: 'plain_text',
        text: `🎖️ Military Aircraft: ${aircraft.flight || aircraft.hex}`,
        emoji: true
      }
    },
    {
      type: 'section',
      fields: [
        { type: 'mrkdwn', text: `*Type:*\n${aircraft.type || 'Unknown'}` },
        { type: 'mrkdwn', text: `*Altitude:*\n${aircraft.alt?.toLocaleString() || 0} ft` },
        { type: 'mrkdwn', text: `*Speed:*\n${aircraft.gs || 0} kts` },
        { type: 'mrkdwn', text: `*Distance:*\n${aircraft.distance?.toFixed(1) || 0} nm` }
      ]
    },
    {
      type: 'context',
      elements: [
        { type: 'mrkdwn', text: `ICAO: \`${aircraft.hex}\`` }
      ]
    },
    {
      type: 'actions',
      elements: [
        {
          type: 'button',
          text: { type: 'plain_text', text: 'FlightRadar24' },
          url: `https://www.flightradar24.com/${aircraft.hex}`
        },
        {
          type: 'button',
          text: { type: 'plain_text', text: 'ADS-B Exchange' },
          url: `https://globe.adsbexchange.com/?icao=${aircraft.hex}`
        }
      ]
    },
    { type: 'divider' }
  ];

  await fetch(WEBHOOK_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ blocks })
  });

  console.log(`Alerted: ${aircraft.flight} (${aircraft.hex})`);
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.onopen = () => console.log('Slack bot connected...');

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;

    if (aircraft.military) {
      await sendSlackAlert(aircraft);
      alerted.add(aircraft.hex);
    }
  }
});
```

```json Response Example
{
  "blocks": [
    {
      "type": "header",
      "text": {"type": "plain_text", "text": "🎖️ Military Aircraft: RCH419"}
    },
    {
      "type": "section",
      "fields": [
        {"type": "mrkdwn", "text": "*Type:*\nC17"},
        {"type": "mrkdwn", "text": "*Altitude:*\n28,000 ft"}
      ]
    }
  ]
}
```

## Create Slack Webhook

1. Go to [Slack Apps](https://api.slack.com/apps)
2. Create New App → From scratch
3. Add "Incoming Webhooks" feature
4. Activate and add to your channel
5. Copy the Webhook URL

## Send Block Kit Messages

<!-- python@20-55 -->
<!-- javascript@15-50 -->

Block Kit provides rich formatting with headers, fields, buttons, and dividers. Messages look professional and are easy to scan.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SLACK_WEBHOOK_URL` | Incoming webhook URL (required) | None |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Custom Channels

Create multiple webhooks for different channels:

```python
CHANNELS = {
    "military": "https://hooks.slack.com/...",
    "emergencies": "https://hooks.slack.com/...",
    "all": "https://hooks.slack.com/..."
}

def send_to_channel(channel, blocks):
    requests.post(CHANNELS[channel], json={"blocks": blocks})
```

### Add Attachments

Include color-coded attachments:

```python
{
    "attachments": [{
        "color": "#ff0000",  # Red for emergencies
        "blocks": blocks
    }]
}
```

## Testing & Verification

1. **Test webhook** directly:
   ```shell
   curl -X POST $SLACK_WEBHOOK_URL -H 'Content-Type: application/json' \
     -d '{"text": "Test from SkySpy"}'
   ```
2. **Start the bot** and verify "connected" message
3. **Wait for aircraft** matching your criteria
4. **Check Slack** for formatted messages with buttons

## Troubleshooting

### Webhook Returns Error

**Problem:** `invalid_payload` or similar errors
**Solution:**
- Validate JSON structure with [Block Kit Builder](https://app.slack.com/block-kit-builder)
- Ensure all required fields are present
- Check for special characters in text that need escaping

### Messages Not Appearing

**Problem:** Bot runs but no messages in Slack
**Solution:**
- Verify webhook URL is correct and active
- Check the webhook is assigned to the right channel
- Test webhook manually with curl

### Rate Limiting

**Problem:** `rate_limited` response
**Solution:** Add delays between messages:

```python
import time
time.sleep(1)  # 1 message per second max
```

## Related Recipes

- [Discord Alert Bot](/docs/discord-alert-bot) - Send to Discord
- [Telegram Bot](/docs/telegram-bot) - Send to Telegram
- [Microsoft Teams](/docs/teams-integration) - Send to Teams
- [Webhook Notifications](/docs/webhook-notifications) - Generic webhooks
