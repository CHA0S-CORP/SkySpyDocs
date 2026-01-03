---
title: "Bot Code"
slug: "discord-alert-bot/code"
excerpt: "Complete Python bot implementation."
hidden: false
---

Create `discord_bot.py`:

```python
#!/usr/bin/env python3
"""SkySpy Discord Alert Bot"""

import json
import os
from datetime import datetime
import requests
import sseclient
from discord_webhook import DiscordWebhook, DiscordEmbed

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
DISCORD_WEBHOOK_URL = os.getenv("DISCORD_WEBHOOK_URL", "YOUR_WEBHOOK_URL_HERE")

MAX_DISTANCE_NM = 50
ALERT_MILITARY = True
ALERT_EMERGENCIES = True
ALERT_SAFETY = True


def create_aircraft_embed(aircraft: dict, event_type: str) -> DiscordEmbed:
    colors = {
        "military": "5865F2",
        "emergency": "ED4245",
        "safety": "FEE75C",
    }
    color = colors.get(event_type, "5865F2")
    callsign = aircraft.get("flight", aircraft.get("hex", "Unknown"))

    titles = {
        "military": f"🎖️ Military: {callsign}",
        "emergency": f"🚨 EMERGENCY: {callsign}",
        "safety": f"⚠️ Safety Alert: {callsign}",
    }

    embed = DiscordEmbed(title=titles.get(event_type, f"✈️ {callsign}"), color=color)

    if aircraft.get("type"):
        embed.add_embed_field(name="Type", value=aircraft["type"], inline=True)
    if aircraft.get("alt"):
        embed.add_embed_field(name="Altitude", value=f"{aircraft['alt']:,} ft", inline=True)
    if aircraft.get("gs"):
        embed.add_embed_field(name="Speed", value=f"{aircraft['gs']} kts", inline=True)
    if aircraft.get("distance"):
        embed.add_embed_field(name="Distance", value=f"{aircraft['distance']:.1f} NM", inline=True)

    embed.set_timestamp()
    embed.set_footer(text="SkySpy Alert")
    return embed


def send_discord_alert(embed: DiscordEmbed):
    webhook = DiscordWebhook(url=DISCORD_WEBHOOK_URL)
    webhook.add_embed(embed)
    webhook.execute()


def should_alert(aircraft: dict) -> tuple[bool, str]:
    distance = aircraft.get("distance", 0)
    if distance > MAX_DISTANCE_NM:
        return False, ""

    if ALERT_MILITARY and aircraft.get("military"):
        return True, "military"
    if ALERT_EMERGENCIES and aircraft.get("squawk") in ["7700", "7600", "7500"]:
        return True, "emergency"

    return False, ""


def main():
    print(f"🛫 Starting SkySpy Discord Bot")
    print(f"📡 Connecting to {SKYSPY_URL}")

    alerted_aircraft = set()

    while True:
        try:
            url = f"{SKYSPY_URL}/api/v1/map/sse"
            response = requests.get(url, stream=True)
            client = sseclient.SSEClient(response)

            print("✅ Connected to SkySpy SSE stream")

            for event in client.events():
                try:
                    data = json.loads(event.data)

                    if event.event in ["aircraft_update", "aircraft_new"]:
                        for aircraft in data.get("aircraft", []):
                            hex_code = aircraft.get("hex")
                            if hex_code in alerted_aircraft:
                                continue

                            should_send, event_type = should_alert(aircraft)
                            if should_send:
                                embed = create_aircraft_embed(aircraft, event_type)
                                send_discord_alert(embed)
                                alerted_aircraft.add(hex_code)

                    elif event.event == "safety_event" and ALERT_SAFETY:
                        embed = DiscordEmbed(
                            title=f"⚠️ {data.get('event_type', 'Safety Event')}",
                            description=data.get("message", ""),
                            color="FEE75C"
                        )
                        embed.set_timestamp()
                        send_discord_alert(embed)

                    if len(alerted_aircraft) > 1000:
                        alerted_aircraft.clear()

                except json.JSONDecodeError:
                    continue

        except requests.exceptions.ConnectionError:
            print("❌ Connection lost, reconnecting in 5s...")
            import time
            time.sleep(5)
        except KeyboardInterrupt:
            print("\n👋 Shutting down")
            break


if __name__ == "__main__":
    main()
```
