---
title: Telegram Bot
description: Send aircraft alerts to Telegram with inline keyboards.
hidden: false
recipe:
  color: '#0088CC'
  icon: 📱
---
```shell Shell
pip install sseclient-py requests
export TELEGRAM_BOT_TOKEN="123456:ABC-DEF..."
export TELEGRAM_CHAT_ID="123456789"
python telegram_bot.py
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
    botToken := os.Getenv("TELEGRAM_BOT_TOKEN")
    chatID := os.Getenv("TELEGRAM_CHAT_ID")
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
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
                        if alerted[hex] {
                            continue
                        }
                        if ac["military"] == true {
                            msg := fmt.Sprintf("🎖️ Military: %s\nType: %s\nAlt: %v ft",
                                ac["flight"], ac["type"], ac["alt"])
                            sendTelegram(botToken, chatID, msg)
                            alerted[hex] = true
                        }
                    }
                }
            }
        }
        resp.Body.Close()
    }
}

func sendTelegram(token, chatID, message string) {
    apiURL := fmt.Sprintf("https://api.telegram.org/bot%s/sendMessage", token)
    http.PostForm(apiURL, url.Values{
        "chat_id":    {chatID},
        "text":       {message},
        "parse_mode": {"Markdown"},
    })
}
```

```python Python
import json
import os
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
CHAT_ID = os.getenv("TELEGRAM_CHAT_ID")

def send_telegram(text):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    requests.post(url, json={"chat_id": CHAT_ID, "text": text, "parse_mode": "Markdown"})

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
                msg = f"🎖️ *Military: {aircraft.get('flight', hex_code)}*\nType: {aircraft.get('type')}\nAlt: {aircraft.get('alt'):,} ft"
                send_telegram(msg)
                alerted.add(hex_code)
```

```javascript JavaScript
const EventSource = require('eventsource');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const BOT_TOKEN = process.env.TELEGRAM_BOT_TOKEN;
const CHAT_ID = process.env.TELEGRAM_CHAT_ID;
const alerted = new Set();

async function sendTelegram(text) {
    await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ chat_id: CHAT_ID, text, parse_mode: 'Markdown' })
    });
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
    const data = JSON.parse(e.data);
    for (const aircraft of data.aircraft || []) {
        if (alerted.has(aircraft.hex)) continue;
        if (aircraft.military) {
            const msg = `🎖️ *Military: ${aircraft.flight || aircraft.hex}*\nType: ${aircraft.type}\nAlt: ${aircraft.alt?.toLocaleString()} ft`;
            await sendTelegram(msg);
            alerted.add(aircraft.hex);
        }
    }
});
```

```json Response Example
{"ok": true, "result": {"message_id": 123, "chat": {"id": 123456789}, "text": "🎖️ Military: RCH419"}}
```

# Install Dependencies & Configure

<!-- shell@1-4 -->
<!-- go@1-11 -->
<!-- python@1-8 -->
<!-- javascript@1-6 -->

Install the required packages and set your Telegram credentials. Get a bot token from [@BotFather](https://t.me/botfather) and your chat ID from [@userinfobot](https://t.me/userinfobot).

# Connect to SSE Stream

<!-- go@22-26 -->
<!-- python@14-16 -->
<!-- javascript@16-17 -->

Connect to SkySpy's Server-Sent Events stream to receive real-time aircraft updates. The connection stays open and receives events as aircraft data changes.

# Filter & Send Alerts

<!-- go@30-42 -->
<!-- python@18-28 -->
<!-- javascript@18-28 -->

When an aircraft matches your filter (military in this example), format a message with flight details and send it via the Telegram Bot API.
