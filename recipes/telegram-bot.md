---
title: Telegram Bot
excerpt: Send aircraft alerts to Telegram with inline keyboards and photos.
hidden: false
recipe:
  color: '#0088CC'
  icon: 📱
difficulty: beginner
tags: [notifications, telegram, bot, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Telegram account
- Bot token from [@BotFather](https://t.me/botfather)
- Your chat ID (use [@userinfobot](https://t.me/userinfobot))
- Python: `pip install sseclient-py requests python-telegram-bot`

## What You'll Build

A Telegram bot that sends real-time aircraft alerts with rich formatting, inline keyboards for quick actions, and optional aircraft photos.

```shell Shell
pip install sseclient-py requests python-telegram-bot
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

    if botToken == "" || chatID == "" {
        fmt.Println("Error: TELEGRAM_BOT_TOKEN and TELEGRAM_CHAT_ID required")
        os.Exit(1)
    }

    alerted := make(map[string]bool)

    for {
        resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
        if err != nil {
            fmt.Printf("Connection failed: %v, retrying...\n", err)
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
                        if alerted[hex] {
                            continue
                        }
                        if ac["military"] == true {
                            msg := fmt.Sprintf("🎖️ *Military Aircraft*\n\n"+
                                "✈️ *%s*\n"+
                                "Type: %s\n"+
                                "Altitude: %v ft\n"+
                                "Speed: %v kts\n"+
                                "ICAO: `%s`",
                                ac["flight"], ac["type"], ac["alt"], ac["gs"], hex)

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
import sys
import requests
import sseclient
from telegram import Bot, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.constants import ParseMode
import asyncio

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
CHAT_ID = os.getenv("TELEGRAM_CHAT_ID")

if not BOT_TOKEN or not CHAT_ID:
    print("Error: TELEGRAM_BOT_TOKEN and TELEGRAM_CHAT_ID required")
    sys.exit(1)

bot = Bot(token=BOT_TOKEN)
alerted = set()


async def send_alert(aircraft):
    hex_code = aircraft.get("hex")
    flight = aircraft.get("flight", hex_code)

    message = (
        f"🎖️ *Military Aircraft Detected*\n\n"
        f"✈️ *{flight}*\n"
        f"Type: {aircraft.get('type', 'Unknown')}\n"
        f"Altitude: {aircraft.get('alt', 0):,} ft\n"
        f"Speed: {aircraft.get('gs', 0)} kts\n"
        f"Distance: {aircraft.get('distance', 0):.1f} nm\n"
        f"ICAO: `{hex_code}`"
    )

    # Inline keyboard for quick actions
    keyboard = [
        [
            InlineKeyboardButton("View on FlightRadar24", url=f"https://www.flightradar24.com/{hex_code}"),
            InlineKeyboardButton("View on ADS-B Exchange", url=f"https://globe.adsbexchange.com/?icao={hex_code}")
        ]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)

    await bot.send_message(
        chat_id=CHAT_ID,
        text=message,
        parse_mode=ParseMode.MARKDOWN,
        reply_markup=reply_markup
    )
    print(f"Alerted: {flight} ({hex_code})")


async def main():
    print(f"Telegram bot connected, monitoring {SKYSPY_URL}...")

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
                            await send_alert(aircraft)
                            alerted.add(hex_code)
        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            await asyncio.sleep(5)


if __name__ == "__main__":
    asyncio.run(main())
```

```javascript JavaScript
const EventSource = require('eventsource');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const BOT_TOKEN = process.env.TELEGRAM_BOT_TOKEN;
const CHAT_ID = process.env.TELEGRAM_CHAT_ID;

if (!BOT_TOKEN || !CHAT_ID) {
  console.error('Error: TELEGRAM_BOT_TOKEN and TELEGRAM_CHAT_ID required');
  process.exit(1);
}

const alerted = new Set();
const TELEGRAM_API = `https://api.telegram.org/bot${BOT_TOKEN}`;

async function sendTelegram(text, inlineKeyboard = null) {
  const body = {
    chat_id: CHAT_ID,
    text,
    parse_mode: 'Markdown'
  };

  if (inlineKeyboard) {
    body.reply_markup = JSON.stringify({ inline_keyboard: inlineKeyboard });
  }

  await fetch(`${TELEGRAM_API}/sendMessage`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body)
  });
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.onopen = () => console.log('Telegram bot connected...');

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;

    if (aircraft.military) {
      const message = `🎖️ *Military Aircraft Detected*\n\n` +
        `✈️ *${aircraft.flight || aircraft.hex}*\n` +
        `Type: ${aircraft.type || 'Unknown'}\n` +
        `Altitude: ${aircraft.alt?.toLocaleString() || 0} ft\n` +
        `Speed: ${aircraft.gs || 0} kts\n` +
        `ICAO: \`${aircraft.hex}\``;

      const keyboard = [[
        { text: 'FlightRadar24', url: `https://www.flightradar24.com/${aircraft.hex}` },
        { text: 'ADS-B Exchange', url: `https://globe.adsbexchange.com/?icao=${aircraft.hex}` }
      ]];

      await sendTelegram(message, keyboard);
      alerted.add(aircraft.hex);
      console.log(`Alerted: ${aircraft.flight} (${aircraft.hex})`);
    }
  }
});
```

```json Response Example
{
  "ok": true,
  "result": {
    "message_id": 123,
    "chat": {"id": 123456789, "type": "private"},
    "text": "🎖️ Military Aircraft Detected\n\n✈️ RCH419\nType: C17\nAltitude: 28,000 ft"
  }
}
```

## Create Telegram Bot

1. Message [@BotFather](https://t.me/botfather) on Telegram
2. Send `/newbot` and follow the prompts
3. Copy the bot token (looks like `123456789:ABC-DEF...`)
4. Message [@userinfobot](https://t.me/userinfobot) to get your chat ID

## Connect and Monitor

Connect to SkySpy's SSE stream and monitor for aircraft matching your criteria. When a match is found, format and send a Telegram message.

## Add Inline Keyboards

Inline keyboards add interactive buttons to your messages. Users can tap to view the aircraft on external tracking sites.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `TELEGRAM_BOT_TOKEN` | Bot token from BotFather (required) | None |
| `TELEGRAM_CHAT_ID` | Your chat or group ID (required) | None |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Send to Groups

To send alerts to a group:
1. Add your bot to the group
2. Make the bot an admin (or disable privacy mode via BotFather)
3. Use the group's chat ID (starts with `-`)

### Add Photos

Include aircraft photos using the Planespotters API:

```python
photo_url = f"https://api.planespotters.net/pub/photos/hex/{hex_code}"
await bot.send_photo(chat_id=CHAT_ID, photo=photo_url, caption=message)
```

## Testing & Verification

1. **Start the bot** and check for "connected" message
2. **Send a test message** manually to verify the bot token works:
   ```shell
   curl "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage?chat_id=$TELEGRAM_CHAT_ID&text=Test"
   ```
3. **Wait for aircraft** or temporarily broaden the filter
4. **Check Telegram** for incoming messages with inline buttons

## Troubleshooting

### Bot Not Responding

**Problem:** No messages appearing in Telegram
**Solution:**
- Verify bot token: `curl https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getMe`
- Check chat ID is correct
- Ensure the bot can message you (start a chat with it first)

### Invalid Chat ID

**Problem:** Error 400: "Chat not found"
**Solution:**
- For private chats: message the bot first to initialize the chat
- For groups: ensure the bot is a member and has permission to post

### Rate Limiting

**Problem:** Some messages aren't being sent
**Solution:** Telegram has rate limits (30 messages/second to same chat). Add delays:

```python
await asyncio.sleep(0.5)  # Wait between messages
```

## Related Recipes

- [Discord Alert Bot](/docs/discord-alert-bot) - Send to Discord
- [Military Aircraft Spotter](/docs/military-spotter) - Built-in alerts
- [Emergency Alert Monitor](/docs/emergency-monitor) - Track emergencies
