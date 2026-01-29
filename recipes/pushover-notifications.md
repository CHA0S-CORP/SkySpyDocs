---
title: Pushover Notifications
excerpt: Send mobile push notifications via Pushover.
hidden: false
recipe:
  color: '#249DF1'
  icon: 📲
difficulty: beginner
tags: [notifications, push, mobile, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Pushover account ($5 one-time purchase)
- Pushover app on your phone/tablet
- API token and user key from Pushover

## What You'll Build

Instant push notifications to your mobile device with priority levels, sounds, and optional images.

```shell Shell
pip install sseclient-py requests
export PUSHOVER_TOKEN="your-api-token"
export PUSHOVER_USER="your-user-key"
python pushover_alerts.py
```

```go Go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "net/http"
    "net/url"
    "os"
    "strings"
)

func main() {
    token := os.Getenv("PUSHOVER_TOKEN")
    user := os.Getenv("PUSHOVER_USER")
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    if token == "" || user == "" {
        fmt.Println("Error: PUSHOVER_TOKEN and PUSHOVER_USER required")
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

                        http.PostForm("https://api.pushover.net/1/messages.json", url.Values{
                            "token":    {token},
                            "user":     {user},
                            "title":    {fmt.Sprintf("🎖️ Military: %s", ac["flight"])},
                            "message":  {fmt.Sprintf("Type: %s\nAltitude: %v ft\nSpeed: %v kts", ac["type"], ac["alt"], ac["gs"])},
                            "priority": {"1"},
                            "sound":    {"alien"},
                            "url":      {fmt.Sprintf("https://globe.adsbexchange.com/?icao=%s", hex)},
                        })
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
PUSHOVER_TOKEN = os.getenv("PUSHOVER_TOKEN")
PUSHOVER_USER = os.getenv("PUSHOVER_USER")

if not PUSHOVER_TOKEN or not PUSHOVER_USER:
    print("Error: PUSHOVER_TOKEN and PUSHOVER_USER required")
    sys.exit(1)

alerted = set()


def send_pushover(aircraft, priority=1):
    hex_code = aircraft.get("hex")
    flight = aircraft.get("flight", hex_code)

    data = {
        "token": PUSHOVER_TOKEN,
        "user": PUSHOVER_USER,
        "title": f"🎖️ Military: {flight}",
        "message": (
            f"Type: {aircraft.get('type', 'Unknown')}\n"
            f"Altitude: {aircraft.get('alt', 0):,} ft\n"
            f"Speed: {aircraft.get('gs', 0)} kts\n"
            f"Distance: {aircraft.get('distance', 0):.1f} nm"
        ),
        "priority": priority,
        "sound": "alien",
        "url": f"https://globe.adsbexchange.com/?icao={hex_code}",
        "url_title": "Track on ADS-B Exchange"
    }

    # For emergency priority (2), require acknowledgment
    if priority == 2:
        data["retry"] = 60    # Retry every 60 seconds
        data["expire"] = 3600  # Expire after 1 hour

    response = requests.post("https://api.pushover.net/1/messages.json", data=data)

    if response.status_code == 200:
        print(f"Notified: {flight} ({hex_code})")
    else:
        print(f"Failed: {response.text}")


def main():
    print(f"Pushover alerts connected to {SKYSPY_URL}...")

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

                        # Emergency squawks get highest priority
                        squawk = aircraft.get("squawk", "")
                        if squawk in ["7700", "7600", "7500"]:
                            send_pushover(aircraft, priority=2)
                            alerted.add(hex_code)
                        elif aircraft.get("military"):
                            send_pushover(aircraft, priority=1)
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
const FormData = require('form-data');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const PUSHOVER_TOKEN = process.env.PUSHOVER_TOKEN;
const PUSHOVER_USER = process.env.PUSHOVER_USER;

if (!PUSHOVER_TOKEN || !PUSHOVER_USER) {
  console.error('Error: PUSHOVER_TOKEN and PUSHOVER_USER required');
  process.exit(1);
}

const alerted = new Set();

async function sendPushover(aircraft, priority = 1) {
  const hex = aircraft.hex;
  const flight = aircraft.flight || hex;

  const form = new FormData();
  form.append('token', PUSHOVER_TOKEN);
  form.append('user', PUSHOVER_USER);
  form.append('title', `🎖️ Military: ${flight}`);
  form.append('message', `Type: ${aircraft.type || 'Unknown'}\nAltitude: ${aircraft.alt || 0} ft`);
  form.append('priority', priority.toString());
  form.append('sound', 'alien');
  form.append('url', `https://globe.adsbexchange.com/?icao=${hex}`);

  await fetch('https://api.pushover.net/1/messages.json', {
    method: 'POST',
    body: form
  });

  console.log(`Notified: ${flight}`);
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;
    if (aircraft.military) {
      await sendPushover(aircraft);
      alerted.add(aircraft.hex);
    }
  }
});
```

```json Response Example
{
  "status": 1,
  "request": "abc123-def456"
}
```

## Get Pushover Credentials

1. Create account at [pushover.net](https://pushover.net)
2. Note your **User Key** on the dashboard
3. Create an Application to get an **API Token**
4. Install the Pushover app on your device

## Priority Levels

| Priority | Effect | Use Case |
|----------|--------|----------|
| -2 | No notification | Logging only |
| -1 | Quiet | Low priority |
| 0 | Normal | Standard alerts |
| 1 | High | Bypass quiet hours |
| 2 | Emergency | Repeat until acknowledged |

<!-- python@45-55 -->

Emergency priority (2) requires acknowledgment and keeps notifying until the user responds.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `PUSHOVER_TOKEN` | Application API token | Required |
| `PUSHOVER_USER` | Your user key | Required |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Custom Sounds

Available sounds: `pushover`, `bike`, `bugle`, `cashregister`, `classical`, `cosmic`, `falling`, `gamelan`, `incoming`, `intermission`, `magic`, `mechanical`, `pianobar`, `siren`, `spacealarm`, `tugboat`, `alien`, `climb`, `persistent`, `echo`, `updown`, `vibrate`, `none`

### Device Targeting

Send to specific devices only:

```python
data["device"] = "iphone,ipad"  # Comma-separated device names
```

### Attach Images

Include aircraft photos:

```python
photo_url = f"https://api.planespotters.net/pub/photos/hex/{hex_code}"
photo_response = requests.get(photo_url)
if photo_response.status_code == 200:
    files = {"attachment": ("aircraft.jpg", photo_response.content, "image/jpeg")}
    requests.post("https://api.pushover.net/1/messages.json", data=data, files=files)
```

## Testing & Verification

1. **Test Pushover API**:
   ```shell
   curl -s --form-string "token=$PUSHOVER_TOKEN" \
        --form-string "user=$PUSHOVER_USER" \
        --form-string "message=Test from SkySpy" \
        https://api.pushover.net/1/messages.json
   ```
2. **Check your device** for the test notification
3. **Start the script** and wait for matching aircraft
4. **Verify sound and priority** work as expected

## Troubleshooting

### Invalid Token/User

**Problem:** `user identifier is invalid` or similar
**Solution:**
- Verify keys are copied correctly (no extra spaces)
- Regenerate the API token if needed
- Ensure user key is for your account

### Notifications Not Arriving

**Problem:** API returns success but no notification
**Solution:**
- Check device is registered in Pushover app
- Verify the app isn't in Do Not Disturb mode
- Try priority 1 to bypass quiet hours

### Rate Limiting

**Problem:** `429` response or throttling
**Solution:** Pushover allows 7,500 messages/month per app. Add throttling:

```python
from datetime import datetime, timedelta

last_sent = {}

def can_send(hex_code, cooldown_minutes=5):
    if hex_code in last_sent:
        if datetime.now() - last_sent[hex_code] < timedelta(minutes=cooldown_minutes):
            return False
    last_sent[hex_code] = datetime.now()
    return True
```

## Related Recipes

- [ntfy.sh Integration](/docs/ntfy-integration) - Free alternative
- [Gotify Notifications](/docs/gotify-notifications) - Self-hosted
- [Telegram Bot](/docs/telegram-bot) - Free mobile notifications
- [Discord Alert Bot](/docs/discord-alert-bot) - Discord webhooks
