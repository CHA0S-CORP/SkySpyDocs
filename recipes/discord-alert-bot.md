---
title: Discord Alert Bot
excerpt: Send real-time aircraft alerts to your Discord server using webhooks.
hidden: false
recipe:
  color: '#5865F2'
  icon: 🤖
difficulty: beginner
tags: [notifications, discord, webhooks, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Discord server with permission to create webhooks
- Python: `pip install sseclient-py requests discord-webhook`
- Node.js: `npm install eventsource node-fetch`
- Go: No external dependencies required

## What You'll Build

A bot that monitors SkySpy's real-time SSE stream and sends rich Discord embeds when military aircraft are detected. The bot uses Discord webhooks for easy setup without needing a full bot application.

```shell Shell
pip install sseclient-py requests discord-webhook
export DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..."
python discord_bot.py
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
    webhookURL := os.Getenv("DISCORD_WEBHOOK_URL")
    if webhookURL == "" {
        fmt.Println("Error: DISCORD_WEBHOOK_URL not set")
        os.Exit(1)
    }

    resp, err := http.Get("http://localhost:5000/api/v1/map/sse")
    if err != nil {
        fmt.Printf("Error connecting to SkySpy: %v\n", err)
        os.Exit(1)
    }
    defer resp.Body.Close()

    alerted := make(map[string]bool)
    scanner := bufio.NewScanner(resp.Body)

    fmt.Println("Discord bot connected, monitoring for military aircraft...")

    for scanner.Scan() {
        line := scanner.Text()
        if strings.HasPrefix(line, "data:") {
            var data map[string]interface{}
            if err := json.Unmarshal([]byte(line[5:]), &data); err != nil {
                continue
            }
            if aircraft, ok := data["aircraft"].([]interface{}); ok {
                for _, a := range aircraft {
                    ac := a.(map[string]interface{})
                    hex := fmt.Sprint(ac["hex"])
                    if alerted[hex] {
                        continue
                    }
                    if ac["military"] == true {
                        embed := map[string]interface{}{
                            "embeds": []map[string]interface{}{{
                                "title":       fmt.Sprintf("🎖️ Military: %s", ac["flight"]),
                                "description": fmt.Sprintf("Type: %s", ac["type"]),
                                "color":       0x5865F2,
                                "fields": []map[string]interface{}{
                                    {"name": "Altitude", "value": fmt.Sprintf("%v ft", ac["alt"]), "inline": true},
                                    {"name": "Speed", "value": fmt.Sprintf("%v kts", ac["gs"]), "inline": true},
                                    {"name": "ICAO", "value": hex, "inline": true},
                                },
                            }},
                        }
                        body, _ := json.Marshal(embed)
                        http.Post(webhookURL, "application/json", bytes.NewReader(body))
                        alerted[hex] = true
                        fmt.Printf("Alerted: %s (%s)\n", ac["flight"], hex)
                    }
                }
            }
        }
    }
}
```

```python Python
import json
import os
import sys
import requests
import sseclient
from discord_webhook import DiscordWebhook, DiscordEmbed

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
WEBHOOK_URL = os.getenv("DISCORD_WEBHOOK_URL")

if not WEBHOOK_URL:
    print("Error: DISCORD_WEBHOOK_URL not set")
    sys.exit(1)

try:
    response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
    response.raise_for_status()
except requests.RequestException as e:
    print(f"Error connecting to SkySpy: {e}")
    sys.exit(1)

client = sseclient.SSEClient(response)
alerted = set()

print("Discord bot connected, monitoring for military aircraft...")

for event in client.events():
    if event.event in ["aircraft_update", "aircraft_new"]:
        try:
            data = json.loads(event.data)
        except json.JSONDecodeError:
            continue

        for aircraft in data.get("aircraft", []):
            hex_code = aircraft.get("hex")
            if hex_code in alerted:
                continue
            if aircraft.get("military"):
                webhook = DiscordWebhook(url=WEBHOOK_URL)
                embed = DiscordEmbed(
                    title=f"🎖️ Military: {aircraft.get('flight', hex_code)}",
                    description=f"Type: {aircraft.get('type', 'Unknown')}",
                    color="5865F2"
                )
                embed.add_embed_field(name="Altitude", value=f"{aircraft.get('alt', 0):,} ft", inline=True)
                embed.add_embed_field(name="Speed", value=f"{aircraft.get('gs', 0)} kts", inline=True)
                embed.add_embed_field(name="ICAO", value=hex_code, inline=True)
                webhook.add_embed(embed)
                webhook.execute()
                alerted.add(hex_code)
                print(f"Alerted: {aircraft.get('flight')} ({hex_code})")
```

```javascript JavaScript
const EventSource = require('eventsource');
const fetch = require('node-fetch');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const WEBHOOK_URL = process.env.DISCORD_WEBHOOK_URL;

if (!WEBHOOK_URL) {
  console.error('Error: DISCORD_WEBHOOK_URL not set');
  process.exit(1);
}

const alerted = new Set();
const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.onerror = (err) => {
  console.error('SSE connection error:', err);
};

es.onopen = () => {
  console.log('Discord bot connected, monitoring for military aircraft...');
};

es.addEventListener('aircraft_update', async (e) => {
  let data;
  try {
    data = JSON.parse(e.data);
  } catch (err) {
    return;
  }

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;
    if (aircraft.military) {
      try {
        await fetch(WEBHOOK_URL, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            embeds: [{
              title: `🎖️ Military: ${aircraft.flight || aircraft.hex}`,
              description: `Type: ${aircraft.type || 'Unknown'}`,
              color: 0x5865F2,
              fields: [
                { name: 'Altitude', value: `${aircraft.alt?.toLocaleString() || 0} ft`, inline: true },
                { name: 'Speed', value: `${aircraft.gs || 0} kts`, inline: true },
                { name: 'ICAO', value: aircraft.hex, inline: true }
              ]
            }]
          })
        });
        alerted.add(aircraft.hex);
        console.log(`Alerted: ${aircraft.flight} (${aircraft.hex})`);
      } catch (err) {
        console.error('Failed to send webhook:', err);
      }
    }
  }
});
```

```json Response Example
{
  "embeds": [{
    "title": "🎖️ Military: RCH419",
    "description": "Type: C17",
    "color": 5793266,
    "fields": [
      {"name": "Altitude", "value": "28,000 ft", "inline": true},
      {"name": "Speed", "value": "450 kts", "inline": true},
      {"name": "ICAO", "value": "AE1234", "inline": true}
    ]
  }]
}
```

## Install Dependencies & Configure

<!-- shell@1-3 -->
<!-- go@1-12 -->
<!-- python@1-14 -->
<!-- javascript@1-11 -->

Install the required packages and set your Discord webhook URL. Create a webhook by right-clicking your Discord channel → Edit Channel → Integrations → Webhooks → New Webhook. Copy the webhook URL and set it as an environment variable.

## Connect to SSE Stream

<!-- shell@3 -->
<!-- go@16-23 -->
<!-- python@16-21 -->
<!-- javascript@13-21 -->

Connect to SkySpy's Server-Sent Events stream to receive real-time aircraft updates. The bot will process each event as it arrives. Error handling ensures the bot reports connection issues.

## Send Discord Embeds

<!-- go@25-52 -->
<!-- python@27-50 -->
<!-- javascript@23-53 -->

When an aircraft matches your filter (military in this example), create a rich Discord embed with aircraft details and send it via the webhook. A tracking set prevents duplicate alerts for the same aircraft.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `DISCORD_WEBHOOK_URL` | Discord webhook URL (required) | None |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Customizing Filters

To alert on different aircraft types, modify the filter condition:

```python
# Alert on any aircraft below 1000ft
if aircraft.get("alt", 99999) < 1000:

# Alert on specific callsigns
if aircraft.get("flight", "").startswith("UAL"):

# Alert on helicopters
if aircraft.get("category") == "A7":
```

## Testing & Verification

1. **Start the bot** and verify it connects (you'll see "Discord bot connected...")
2. **Wait for military aircraft** or temporarily change the filter to match more aircraft
3. **Check Discord** for incoming embeds with aircraft details
4. **Verify deduplication** - the same aircraft should only trigger one alert

### Test Mode

For testing, temporarily modify the filter to alert on all aircraft:

```python
# Alert on ALL aircraft (testing only)
if aircraft.get("hex"):
    # ... send webhook
```

## Troubleshooting

### Webhook URL Invalid

**Problem:** `Error: DISCORD_WEBHOOK_URL not set` or webhook fails
**Solution:** Ensure you've exported the full webhook URL including the token. It should look like `https://discord.com/api/webhooks/123456789/abcdefg...`

### No Alerts Appearing

**Problem:** Bot is running but no alerts are being sent
**Solution:**
- Verify military aircraft are in range using SkySpy's web interface
- Check that the SSE stream is working: `curl http://localhost:5000/api/v1/map/sse`
- Temporarily broaden the filter for testing

### Rate Limiting

**Problem:** Some alerts are being dropped
**Solution:** Discord webhooks have rate limits. Add a small delay between alerts or use a queue:

```python
import time
time.sleep(1)  # Wait 1 second between alerts
```

### Connection Drops

**Problem:** Bot stops receiving events after some time
**Solution:** Implement reconnection logic:

```python
while True:
    try:
        # ... existing code
    except Exception as e:
        print(f"Reconnecting in 5s: {e}")
        time.sleep(5)
```

## Related Recipes

- [Telegram Bot](/docs/telegram-bot) - Send alerts to Telegram instead
- [Slack Integration](/docs/slack-integration) - Post to Slack channels
- [Military Aircraft Spotter](/docs/military-spotter) - Built-in military alerts
- [Track Specific Aircraft](/docs/track-aircraft) - Monitor specific tail numbers
