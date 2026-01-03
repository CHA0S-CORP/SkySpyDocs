---
title: Discord Alert Bot
description: Send real-time aircraft alerts to your Discord server using webhooks.
hidden: true
recipe:
  color: '#5865F2'
  icon: 🤖
---
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
    resp, _ := http.Get("http://localhost:5000/api/v1/map/sse")
    defer resp.Body.Close()

    scanner := bufio.NewScanner(resp.Body)
    for scanner.Scan() {
        line := scanner.Text()
        if strings.HasPrefix(line, "data:") {
            var data map[string]interface{}
            json.Unmarshal([]byte(line[5:]), &data)
            if aircraft, ok := data["aircraft"].([]interface{}); ok {
                for _, a := range aircraft {
                    ac := a.(map[string]interface{})
                    if ac["military"] == true {
                        embed := map[string]interface{}{
                            "embeds": []map[string]interface{}{{
                                "title": fmt.Sprintf("🎖️ Military: %s", ac["flight"]),
                                "color": 0x5865F2,
                            }},
                        }
                        body, _ := json.Marshal(embed)
                        http.Post(webhookURL, "application/json", bytes.NewReader(body))
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
import requests
import sseclient
from discord_webhook import DiscordWebhook, DiscordEmbed

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
WEBHOOK_URL = os.getenv("DISCORD_WEBHOOK_URL")

response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True)
client = sseclient.SSEClient(response)
alerted = set()

for event in client.events():
    if event.event in ["aircraft_update", "aircraft_new"]:
        data = json.loads(event.data)
        for aircraft in data.get("aircraft", []):
            hex_code = aircraft.get("hex")
            if hex_code in alerted:
                continue
            if aircraft.get("military"):
                webhook = DiscordWebhook(url=WEBHOOK_URL)
                embed = DiscordEmbed(
                    title=f"🎖️ Military: {aircraft.get('flight', hex_code)}",
                    color="5865F2"
                )
                embed.add_embed_field(name="Type", value=aircraft.get("type", "Unknown"))
                embed.add_embed_field(name="Altitude", value=f"{aircraft.get('alt', 0):,} ft")
                webhook.add_embed(embed)
                webhook.execute()
                alerted.add(hex_code)
```

```javascript JavaScript
const EventSource = require('eventsource');
const fetch = require('node-fetch');

const WEBHOOK_URL = process.env.DISCORD_WEBHOOK_URL;
const alerted = new Set();

const es = new EventSource('http://localhost:5000/api/v1/map/sse');

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);
  for (const aircraft of data.aircraft) {
    if (alerted.has(aircraft.hex)) continue;
    if (aircraft.military) {
      await fetch(WEBHOOK_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          embeds: [{
            title: `🎖️ Military: ${aircraft.flight || aircraft.hex}`,
            color: 0x5865F2,
            fields: [
              { name: 'Type', value: aircraft.type || 'Unknown', inline: true },
              { name: 'Altitude', value: `${aircraft.alt?.toLocaleString() || 0} ft`, inline: true }
            ]
          }]
        })
      });
      alerted.add(aircraft.hex);
    }
  }
});
```

```json Response Example
{"embeds": [{"title": "🎖️ Military: RCH419", "color": 5793266, "fields": [{"name": "Type", "value": "C17"}, {"name": "Altitude", "value": "28,000 ft"}]}]}
```

# step1

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->

Build a Discord bot that sends aircraft alerts to your server in real-time. Create a Discord webhook first: Right-click channel → Edit Channel → Integrations → Webhooks → New Webhook.

# step2

The bot connects to SkySpy's SSE stream and sends rich embeds for matching aircraft. Customize the filter conditions to alert on military aircraft, emergencies, or specific callsigns.

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->

# step3

To run as a background service, use systemd (Linux) or Docker:

```shell
# Systemd
sudo systemctl enable skyspy-discord
sudo systemctl start skyspy-discord

# Docker
docker run -d -e DISCORD_WEBHOOK_URL="..." skyspy-discord
```

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->
