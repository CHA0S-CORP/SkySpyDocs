---
title: ntfy.sh Integration
excerpt: Self-hosted push notifications with ntfy.sh.
hidden: false
recipe:
  color: '#57A64E'
  icon: 🔔
difficulty: beginner
tags: [notifications, push, self-hosted, ntfy]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- ntfy.sh account (free) or self-hosted ntfy server
- ntfy app installed on your device

## What You'll Build

Push notifications using ntfy, a simple HTTP-based pub-sub notification service. Free tier includes unlimited notifications with no account required.

```shell Shell
# Send a test notification
curl -d "Test from SkySpy" ntfy.sh/your-topic

# With title and priority
curl -H "Title: Aircraft Alert" -H "Priority: high" \
     -d "Military aircraft detected" ntfy.sh/your-topic
```

```go Go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "strings"
)

func main() {
    ntfyTopic := os.Getenv("NTFY_TOPIC")
    ntfyServer := os.Getenv("NTFY_SERVER")
    if ntfyServer == "" {
        ntfyServer = "https://ntfy.sh"
    }
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    if ntfyTopic == "" {
        fmt.Println("Error: NTFY_TOPIC required")
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

                        msg := fmt.Sprintf("Type: %s | Alt: %v ft | Speed: %v kts",
                            ac["type"], ac["alt"], ac["gs"])

                        req, _ := http.NewRequest("POST", ntfyServer+"/"+ntfyTopic, strings.NewReader(msg))
                        req.Header.Set("Title", fmt.Sprintf("🎖️ Military: %s", ac["flight"]))
                        req.Header.Set("Priority", "high")
                        req.Header.Set("Tags", "airplane,military")
                        req.Header.Set("Click", fmt.Sprintf("https://globe.adsbexchange.com/?icao=%s", hex))

                        http.DefaultClient.Do(req)
                        alerted[hex] = true
                        fmt.Printf("Notified: %s\n", ac["flight"])
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
NTFY_TOPIC = os.getenv("NTFY_TOPIC")
NTFY_SERVER = os.getenv("NTFY_SERVER", "https://ntfy.sh")

if not NTFY_TOPIC:
    print("Error: NTFY_TOPIC required")
    sys.exit(1)

alerted = set()


def send_ntfy(aircraft, priority="high"):
    hex_code = aircraft.get("hex")
    flight = aircraft.get("flight", hex_code)

    headers = {
        "Title": f"🎖️ Military: {flight}",
        "Priority": priority,
        "Tags": "airplane,military",
        "Click": f"https://globe.adsbexchange.com/?icao={hex_code}",
        "Actions": f"view, Track on FR24, https://www.flightradar24.com/{hex_code}"
    }

    message = (
        f"Type: {aircraft.get('type', 'Unknown')}\n"
        f"Altitude: {aircraft.get('alt', 0):,} ft\n"
        f"Speed: {aircraft.get('gs', 0)} kts\n"
        f"Distance: {aircraft.get('distance', 0):.1f} nm"
    )

    response = requests.post(
        f"{NTFY_SERVER}/{NTFY_TOPIC}",
        data=message.encode('utf-8'),
        headers=headers
    )

    if response.status_code == 200:
        print(f"Notified: {flight} ({hex_code})")
    else:
        print(f"Failed: {response.text}")


def main():
    print(f"ntfy alerts connected to {SKYSPY_URL}...")

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
                        if squawk in ["7700", "7600", "7500"]:
                            send_ntfy(aircraft, priority="urgent")
                            alerted.add(hex_code)
                        elif aircraft.get("military"):
                            send_ntfy(aircraft)
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
const NTFY_TOPIC = process.env.NTFY_TOPIC;
const NTFY_SERVER = process.env.NTFY_SERVER || 'https://ntfy.sh';

if (!NTFY_TOPIC) {
  console.error('Error: NTFY_TOPIC required');
  process.exit(1);
}

const alerted = new Set();

async function sendNtfy(aircraft, priority = 'high') {
  const hex = aircraft.hex;
  const flight = aircraft.flight || hex;

  await fetch(`${NTFY_SERVER}/${NTFY_TOPIC}`, {
    method: 'POST',
    body: `Type: ${aircraft.type || 'Unknown'}\nAltitude: ${aircraft.alt || 0} ft\nSpeed: ${aircraft.gs || 0} kts`,
    headers: {
      'Title': `🎖️ Military: ${flight}`,
      'Priority': priority,
      'Tags': 'airplane,military',
      'Click': `https://globe.adsbexchange.com/?icao=${hex}`
    }
  });

  console.log(`Notified: ${flight}`);
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;
    if (aircraft.military) {
      await sendNtfy(aircraft);
      alerted.add(aircraft.hex);
    }
  }
});
```

```json Response Example
{
  "id": "abc123",
  "time": 1705330338,
  "event": "message",
  "topic": "skyspy-alerts",
  "title": "🎖️ Military: RCH419",
  "message": "Type: C17\nAltitude: 28,000 ft\nSpeed: 380 kts"
}
```

## Subscribe to Topic

1. Install ntfy app from [ntfy.sh](https://ntfy.sh)
2. Add subscription to your topic (e.g., `skyspy-alerts`)
3. Choose a unique topic name - anyone with the name can subscribe

For private topics, use a self-hosted server or ntfy.sh with authentication.

## Send Notifications with Headers

<!-- python@22-45 -->

ntfy uses HTTP headers for rich notifications:
- `Title`: Notification title
- `Priority`: `min`, `low`, `default`, `high`, `urgent`
- `Tags`: Emoji tags like `airplane,warning`
- `Click`: URL to open when tapped
- `Actions`: Action buttons

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `NTFY_TOPIC` | Topic name (required) | None |
| `NTFY_SERVER` | ntfy server URL | `https://ntfy.sh` |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Action Buttons

Add clickable buttons to notifications:

```python
headers["Actions"] = (
    "view, FlightRadar24, https://www.flightradar24.com/{hex}; "
    "view, ADS-B Exchange, https://globe.adsbexchange.com/?icao={hex}"
)
```

### Emoji Tags

Common aviation tags:
- `airplane` - ✈️
- `helicopter` - 🚁
- `warning` - ⚠️
- `rotating_light` - 🚨
- `military` - (no built-in, use custom)

### Attach Images

```python
headers["Attach"] = f"https://api.planespotters.net/pub/photos/hex/{hex_code}"
headers["Filename"] = "aircraft.jpg"
```

## Self-Hosted ntfy

Run your own ntfy server:

```shell
docker run -p 80:80 binwiederhier/ntfy serve
```

Then set `NTFY_SERVER=http://your-server:80`

## Testing & Verification

1. **Subscribe** to your topic in the ntfy app
2. **Send test notification**:
   ```shell
   curl -d "Test from SkySpy" ntfy.sh/your-topic
   ```
3. **Verify notification** appears on your device
4. **Start the script** and wait for aircraft

## Troubleshooting

### Notifications Not Arriving

**Problem:** No notifications on device
**Solution:**
- Verify topic name matches exactly
- Check ntfy app has notification permissions
- Try the public ntfy.sh server first

### Topic Already In Use

**Problem:** Getting notifications from others
**Solution:**
- Use a unique, hard-to-guess topic name
- Self-host ntfy for privacy
- Use ntfy.sh with authentication

### Rate Limiting

**Problem:** `429 Too Many Requests`
**Solution:** Public ntfy.sh has limits. Self-host or add delays:

```python
import time
time.sleep(0.5)  # Wait between notifications
```

## Related Recipes

- [Gotify Notifications](/docs/gotify-notifications) - Self-hosted alternative
- [Pushover Notifications](/docs/pushover-notifications) - Commercial option
- [Telegram Bot](/docs/telegram-bot) - Free mobile notifications
- [Webhook Notifications](/docs/webhook-notifications) - Generic webhooks
