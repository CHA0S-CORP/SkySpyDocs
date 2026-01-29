---
title: Gotify Notifications
excerpt: Self-hosted push notifications with Gotify
hidden: false
recipe:
  color: '#2196F3'
  icon: "🔔"
difficulty: beginner
tags: [notifications, gotify, self-hosted, push]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Gotify server running (self-hosted)
- Application token from your Gotify server

## What You'll Build

Self-hosted push notifications to your devices with customizable priority levels, no third-party dependencies, and complete control over your data.

```shell Shell
# Start Gotify with Docker
docker run -d \
  --name gotify \
  -p 8080:80 \
  -v gotify_data:/app/data \
  gotify/server

# Install dependencies
pip install sseclient-py requests

# Set environment variables
export GOTIFY_URL="http://localhost:8080"
export GOTIFY_TOKEN="your-app-token"

# Run the notification script
python gotify_alerts.py
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

type GotifyMessage struct {
	Title    string            `json:"title"`
	Message  string            `json:"message"`
	Priority int               `json:"priority"`
	Extras   map[string]interface{} `json:"extras,omitempty"`
}

func main() {
	token := os.Getenv("GOTIFY_TOKEN")
	gotifyURL := os.Getenv("GOTIFY_URL")
	skyspyURL := os.Getenv("SKYSPY_URL")

	if gotifyURL == "" {
		gotifyURL = "http://localhost:8080"
	}
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	if token == "" {
		fmt.Println("Error: GOTIFY_TOKEN required")
		os.Exit(1)
	}

	alerted := make(map[string]bool)

	for {
		resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
		if err != nil {
			fmt.Printf("Connection error: %v, retrying...\n", err)
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

						if alerted[hex] || ac["military"] != true {
							continue
						}

						sendGotify(gotifyURL, token, ac)
						alerted[hex] = true
						fmt.Printf("Notified: %s\n", ac["flight"])
					}
				}
			}
		}
		resp.Body.Close()
	}
}

func sendGotify(gotifyURL, token string, aircraft map[string]interface{}) {
	msg := GotifyMessage{
		Title:    fmt.Sprintf("Military: %s", aircraft["flight"]),
		Message:  fmt.Sprintf("Type: %s\nAltitude: %v ft\nSpeed: %v kts", aircraft["type"], aircraft["alt"], aircraft["gs"]),
		Priority: 7,
		Extras: map[string]interface{}{
			"client::notification": map[string]interface{}{
				"click": map[string]string{
					"url": fmt.Sprintf("https://globe.adsbexchange.com/?icao=%s", aircraft["hex"]),
				},
			},
		},
	}

	body, _ := json.Marshal(msg)
	req, _ := http.NewRequest("POST", gotifyURL+"/message", bytes.NewBuffer(body))
	req.Header.Set("Content-Type", "application/json")
	req.Header.Set("X-Gotify-Key", token)

	http.DefaultClient.Do(req)
}
```

```python Python
import json
import os
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
GOTIFY_URL = os.getenv("GOTIFY_URL", "http://localhost:8080")
GOTIFY_TOKEN = os.getenv("GOTIFY_TOKEN")

if not GOTIFY_TOKEN:
    print("Error: GOTIFY_TOKEN required")
    sys.exit(1)

alerted = set()


def send_gotify(aircraft, priority=7):
    hex_code = aircraft.get("hex")
    flight = aircraft.get("flight", hex_code)

    data = {
        "title": f"Military: {flight}",
        "message": (
            f"Type: {aircraft.get('type', 'Unknown')}\n"
            f"Altitude: {aircraft.get('alt', 0):,} ft\n"
            f"Speed: {aircraft.get('gs', 0)} kts\n"
            f"Distance: {aircraft.get('distance', 0):.1f} nm"
        ),
        "priority": priority,
        "extras": {
            "client::notification": {
                "click": {
                    "url": f"https://globe.adsbexchange.com/?icao={hex_code}"
                }
            }
        }
    }

    response = requests.post(
        f"{GOTIFY_URL}/message",
        json=data,
        headers={"X-Gotify-Key": GOTIFY_TOKEN}
    )

    if response.status_code == 200:
        print(f"Notified: {flight} ({hex_code})")
    else:
        print(f"Failed: {response.text}")


def main():
    print(f"Gotify alerts connected to {SKYSPY_URL}...")

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
                            send_gotify(aircraft, priority=10)
                            alerted.add(hex_code)
                        elif aircraft.get("military"):
                            send_gotify(aircraft, priority=7)
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
const GOTIFY_URL = process.env.GOTIFY_URL || 'http://localhost:8080';
const GOTIFY_TOKEN = process.env.GOTIFY_TOKEN;

if (!GOTIFY_TOKEN) {
  console.error('Error: GOTIFY_TOKEN required');
  process.exit(1);
}

const alerted = new Set();

async function sendGotify(aircraft, priority = 7) {
  const hex = aircraft.hex;
  const flight = aircraft.flight || hex;

  const data = {
    title: `Military: ${flight}`,
    message: `Type: ${aircraft.type || 'Unknown'}\nAltitude: ${aircraft.alt || 0} ft\nSpeed: ${aircraft.gs || 0} kts`,
    priority: priority,
    extras: {
      'client::notification': {
        click: {
          url: `https://globe.adsbexchange.com/?icao=${hex}`
        }
      }
    }
  };

  const response = await fetch(`${GOTIFY_URL}/message`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Gotify-Key': GOTIFY_TOKEN
    },
    body: JSON.stringify(data)
  });

  if (response.ok) {
    console.log(`Notified: ${flight}`);
  } else {
    console.error(`Failed: ${await response.text()}`);
  }
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;
    if (aircraft.military) {
      await sendGotify(aircraft);
      alerted.add(aircraft.hex);
    }
  }
});

console.log(`Gotify alerts connected to ${SKYSPY_URL}...`);
```

```json Response Example
{
  "id": 42,
  "appid": 1,
  "message": "Type: F-16\nAltitude: 25,000 ft\nSpeed: 450 kts",
  "title": "Military: VIPER01",
  "priority": 7,
  "date": "2024-01-15T10:30:00Z"
}
```

## Docker Setup

Deploy Gotify with Docker Compose for a persistent setup:

```yaml
# docker-compose.yml
version: "3"
services:
  gotify:
    image: gotify/server
    ports:
      - "8080:80"
    volumes:
      - gotify_data:/app/data
    environment:
      - GOTIFY_DEFAULTUSER_PASS=admin
    restart: unless-stopped

volumes:
  gotify_data:
```

Start with:
```shell
docker-compose up -d
```

## Get Gotify Credentials

1. Access Gotify at `http://your-server:8080`
2. Log in with default credentials (admin/admin) or your configured password
3. Go to **Apps** in the sidebar
4. Click **Create Application**
5. Name it "SkySpy Alerts" and copy the generated **Token**

## Priority Levels

| Priority | Effect | Use Case |
|----------|--------|----------|
| 0 | Min priority | Silent/logging |
| 1-3 | Low | Background updates |
| 4-7 | Normal | Standard alerts |
| 8-10 | High | Important notifications |

Priority affects how notifications appear on your device. Higher priorities may bypass Do Not Disturb on some clients.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `GOTIFY_URL` | Gotify server URL | `http://localhost:8080` |
| `GOTIFY_TOKEN` | Application token | Required |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### HTTPS with Reverse Proxy

For remote access, use a reverse proxy like Nginx or Caddy:

```nginx
server {
    listen 443 ssl;
    server_name gotify.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        # WebSocket support for real-time updates
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Client Apps

Install the Gotify client on your devices:

- **Android**: [Google Play](https://play.google.com/store/apps/details?id=com.github.gotify) or [F-Droid](https://f-droid.org/packages/com.github.gotify/)
- **iOS**: Use the web interface or third-party clients
- **Desktop**: Web interface at your Gotify URL

## Testing & Verification

1. **Test Gotify API directly**:
   ```shell
   curl -X POST "${GOTIFY_URL}/message" \
     -H "X-Gotify-Key: ${GOTIFY_TOKEN}" \
     -H "Content-Type: application/json" \
     -d '{"title": "Test", "message": "Hello from SkySpy!", "priority": 5}'
   ```
2. **Check the Gotify web interface** for the test message
3. **Verify your mobile app** receives the notification
4. **Start the script** and wait for matching aircraft

## Troubleshooting

### Connection Refused

**Problem:** `Connection refused` or unable to reach Gotify
**Solution:**
- Verify Gotify is running: `docker ps | grep gotify`
- Check the port is accessible: `curl http://localhost:8080/health`
- If using Docker, ensure the port mapping is correct

### Invalid Token

**Problem:** `401 Unauthorized` response
**Solution:**
- Verify token is copied correctly (no extra spaces)
- Regenerate the application token in Gotify
- Ensure you're using an App token, not a Client token

### Notifications Not Appearing

**Problem:** Messages appear in Gotify web but not on mobile
**Solution:**
- Ensure the Gotify app has notification permissions
- Check battery optimization isn't killing the app
- Verify the app is connected (check app status)
- For Android, disable battery saver for Gotify

### WebSocket Disconnects

**Problem:** Mobile app frequently disconnects
**Solution:**
- Enable WebSocket support in your reverse proxy
- Check for aggressive NAT timeouts
- Increase keep-alive intervals in Gotify config

## Related Recipes

- [Pushover Notifications](/docs/pushover-notifications) - Mobile push (paid)
- [ntfy.sh Integration](/docs/ntfy-integration) - Free self-hosted alternative
- [Telegram Bot](/docs/telegram-bot) - Free mobile notifications
- [Discord Alert Bot](/docs/discord-alert-bot) - Discord webhooks
