---
title: "Discord Alert Bot"
slug: "discord-alert-bot"
excerpt: "Send real-time aircraft alerts to your Discord server using webhooks."
hidden: false
---

Build a Discord bot that sends aircraft alerts to your server in real-time. Get notified about military aircraft, emergency squawks, and custom alert rules.

```mermaid
flowchart LR
    SKYSPY[SkySpy API] -->|SSE Stream| BOT[Python Bot]
    BOT -->|Webhook| DISCORD[Discord Channel]

    style SKYSPY fill:#e3f2fd
    style BOT fill:#fff3e0
    style DISCORD fill:#7289da,color:#fff
```

## What You'll Build

<CardGroup cols={2}>
  <Card title="Real-time Alerts" icon="bolt">
    Instant notifications when aircraft match your rules
  </Card>
  <Card title="Rich Embeds" icon="image">
    Beautiful Discord embeds with aircraft details
  </Card>
  <Card title="Safety Events" icon="shield">
    TCAS, proximity, and emergency alerts
  </Card>
  <Card title="Customizable" icon="gear">
    Filter by aircraft type, distance, altitude
  </Card>
</CardGroup>

## Prerequisites

<Check>
**SkySpy running** — API accessible at `http://localhost:5000`
</Check>

<Check>
**Discord webhook** — Create one in your channel settings
</Check>

<Check>
**Python 3.8+** — With pip for package installation
</Check>

---

## Step 1: Create Discord Webhook

<Steps>
  <Step title="Open Channel Settings">
    Right-click your Discord channel → **Edit Channel**
  </Step>
  <Step title="Create Webhook">
    Go to **Integrations** → **Webhooks** → **New Webhook**
  </Step>
  <Step title="Copy URL">
    Name your webhook "SkySpy Alerts" and copy the webhook URL
  </Step>
</Steps>

Your webhook URL will look like:
```
https://discord.com/api/webhooks/1234567890/abcdefghijklmnop
```

---

## Step 2: Install Dependencies

```bash
pip install sseclient-py requests discord-webhook
```

---

## Step 3: Create the Bot

Create a file called `discord_bot.py`:

```python
#!/usr/bin/env python3
"""
SkySpy Discord Alert Bot

Streams aircraft events from SkySpy and posts alerts to Discord.
"""

import json
import os
from datetime import datetime

import requests
import sseclient
from discord_webhook import DiscordWebhook, DiscordEmbed

# Configuration
SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
DISCORD_WEBHOOK_URL = os.getenv("DISCORD_WEBHOOK_URL", "YOUR_WEBHOOK_URL_HERE")

# Alert filters (customize these!)
MIN_DISTANCE_NM = 0       # Minimum distance to alert (0 = all)
MAX_DISTANCE_NM = 50      # Maximum distance to alert
ALERT_MILITARY = True     # Alert on military aircraft
ALERT_EMERGENCIES = True  # Alert on emergency squawks
ALERT_SAFETY = True       # Alert on safety events (TCAS, proximity)


def create_aircraft_embed(aircraft: dict, event_type: str) -> DiscordEmbed:
    """Create a Discord embed for an aircraft alert."""

    # Determine color based on event type
    colors = {
        "military": "5865F2",      # Discord blurple
        "emergency": "ED4245",     # Red
        "safety": "FEE75C",        # Yellow
        "custom_alert": "57F287",  # Green
    }
    color = colors.get(event_type, "5865F2")

    # Build title
    callsign = aircraft.get("flight", aircraft.get("hex", "Unknown"))
    title = f"✈️ {callsign}"

    if event_type == "military":
        title = f"🎖️ Military: {callsign}"
    elif event_type == "emergency":
        title = f"🚨 EMERGENCY: {callsign}"
    elif event_type == "safety":
        title = f"⚠️ Safety Alert: {callsign}"

    embed = DiscordEmbed(title=title, color=color)

    # Add fields
    if aircraft.get("type"):
        embed.add_embed_field(name="Type", value=aircraft["type"], inline=True)

    if aircraft.get("alt"):
        embed.add_embed_field(
            name="Altitude",
            value=f"{aircraft['alt']:,} ft",
            inline=True
        )

    if aircraft.get("gs"):
        embed.add_embed_field(
            name="Speed",
            value=f"{aircraft['gs']} kts",
            inline=True
        )

    if aircraft.get("distance"):
        embed.add_embed_field(
            name="Distance",
            value=f"{aircraft['distance']:.1f} NM",
            inline=True
        )

    if aircraft.get("squawk"):
        embed.add_embed_field(
            name="Squawk",
            value=aircraft["squawk"],
            inline=True
        )

    # Add ICAO hex
    embed.add_embed_field(
        name="ICAO",
        value=aircraft.get("hex", "Unknown"),
        inline=True
    )

    # Timestamp
    embed.set_timestamp()
    embed.set_footer(text="SkySpy Alert")

    return embed


def send_discord_alert(embed: DiscordEmbed):
    """Send an embed to Discord."""
    webhook = DiscordWebhook(url=DISCORD_WEBHOOK_URL)
    webhook.add_embed(embed)

    try:
        response = webhook.execute()
        if response.status_code == 200:
            print(f"✅ Alert sent to Discord")
        else:
            print(f"❌ Discord error: {response.status_code}")
    except Exception as e:
        print(f"❌ Failed to send alert: {e}")


def should_alert(aircraft: dict) -> tuple[bool, str]:
    """Check if we should alert on this aircraft."""

    # Check distance filter
    distance = aircraft.get("distance", 0)
    if distance < MIN_DISTANCE_NM or distance > MAX_DISTANCE_NM:
        return False, ""

    # Check for military
    if ALERT_MILITARY and aircraft.get("military"):
        return True, "military"

    # Check for emergency squawk
    if ALERT_EMERGENCIES and aircraft.get("squawk") in ["7700", "7600", "7500"]:
        return True, "emergency"

    return False, ""


def handle_safety_event(event_data: dict):
    """Handle safety events (TCAS, proximity, etc.)."""
    if not ALERT_SAFETY:
        return

    embed = DiscordEmbed(
        title=f"⚠️ {event_data.get('event_type', 'Safety Event')}",
        description=event_data.get("message", "Unknown safety event"),
        color="FEE75C"
    )

    embed.add_embed_field(
        name="Severity",
        value=event_data.get("severity", "unknown").upper(),
        inline=True
    )

    if event_data.get("icao"):
        embed.add_embed_field(
            name="Aircraft",
            value=event_data["icao"],
            inline=True
        )

    embed.set_timestamp()
    embed.set_footer(text="SkySpy Safety Monitor")

    send_discord_alert(embed)


def handle_custom_alert(event_data: dict):
    """Handle custom alert rule triggers."""
    embed = DiscordEmbed(
        title=f"🔔 {event_data.get('rule_name', 'Custom Alert')}",
        description=event_data.get("message", ""),
        color="57F287"
    )

    aircraft_data = event_data.get("aircraft_data", {})

    if aircraft_data.get("flight"):
        embed.add_embed_field(
            name="Callsign",
            value=aircraft_data["flight"],
            inline=True
        )

    if aircraft_data.get("alt"):
        embed.add_embed_field(
            name="Altitude",
            value=f"{aircraft_data['alt']:,} ft",
            inline=True
        )

    embed.set_timestamp()
    embed.set_footer(text="SkySpy Alert Rule")

    send_discord_alert(embed)


def main():
    """Main event loop."""
    print(f"🛫 Starting SkySpy Discord Bot")
    print(f"📡 Connecting to {SKYSPY_URL}")
    print(f"📍 Filtering: {MIN_DISTANCE_NM}-{MAX_DISTANCE_NM} NM")
    print(f"🎖️  Military alerts: {ALERT_MILITARY}")
    print(f"🚨 Emergency alerts: {ALERT_EMERGENCIES}")
    print(f"⚠️  Safety alerts: {ALERT_SAFETY}")
    print()

    # Track already-alerted aircraft to avoid spam
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

                    # Handle aircraft updates
                    if event.event in ["aircraft_update", "aircraft_new"]:
                        for aircraft in data.get("aircraft", []):
                            hex_code = aircraft.get("hex")

                            # Skip if already alerted recently
                            if hex_code in alerted_aircraft:
                                continue

                            should_send, event_type = should_alert(aircraft)

                            if should_send:
                                embed = create_aircraft_embed(aircraft, event_type)
                                send_discord_alert(embed)
                                alerted_aircraft.add(hex_code)

                    # Handle safety events
                    elif event.event == "safety_event":
                        handle_safety_event(data)

                    # Handle custom alerts
                    elif event.event == "alert_triggered":
                        handle_custom_alert(data)

                    # Clean up old alerts periodically
                    if len(alerted_aircraft) > 1000:
                        alerted_aircraft.clear()

                except json.JSONDecodeError:
                    continue
                except Exception as e:
                    print(f"Error processing event: {e}")

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

---

## Step 4: Configure the Bot

Set your Discord webhook URL:

<Tabs>
  <Tab title="Environment Variable">
    ```bash
    export DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..."
    export SKYSPY_URL="http://localhost:5000"
    ```
  </Tab>
  <Tab title="Edit the Script">
    Edit the configuration section at the top of `discord_bot.py`:

    ```python
    DISCORD_WEBHOOK_URL = "https://discord.com/api/webhooks/..."
    SKYSPY_URL = "http://localhost:5000"
    ```
  </Tab>
</Tabs>

### Customize Alert Filters

```python
# Only alert for aircraft within 25 NM
MAX_DISTANCE_NM = 25

# Disable military alerts
ALERT_MILITARY = False

# Enable all safety events
ALERT_SAFETY = True
```

---

## Step 5: Run the Bot

```bash
python discord_bot.py
```

You should see:

```
🛫 Starting SkySpy Discord Bot
📡 Connecting to http://localhost:5000
📍 Filtering: 0-50 NM
🎖️  Military alerts: True
🚨 Emergency alerts: True
⚠️  Safety alerts: True

✅ Connected to SkySpy SSE stream
```

---

## Example Discord Output

When the bot detects a matching aircraft, it sends a rich embed:

```
┌────────────────────────────────────────┐
│ 🎖️ Military: EVAC26                    │
├────────────────────────────────────────┤
│ Type: C17     │ Altitude: 28,000 ft    │
│ Speed: 420 kts│ Distance: 12.4 NM      │
│ Squawk: 4621  │ ICAO: AE1234           │
├────────────────────────────────────────┤
│ SkySpy Alert • Today at 3:42 PM        │
└────────────────────────────────────────┘
```

---

## Running as a Service

<Accordion title="Systemd service (Linux)" icon="linux">

Create `/etc/systemd/system/skyspy-discord.service`:

```ini
[Unit]
Description=SkySpy Discord Alert Bot
After=network.target

[Service]
Type=simple
User=your-user
WorkingDirectory=/path/to/bot
Environment="DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/..."
Environment="SKYSPY_URL=http://localhost:5000"
ExecStart=/usr/bin/python3 discord_bot.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl enable skyspy-discord
sudo systemctl start skyspy-discord
```

</Accordion>

<Accordion title="Docker" icon="docker">

Create a `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY discord_bot.py .
RUN pip install sseclient-py requests discord-webhook

CMD ["python", "discord_bot.py"]
```

Run:

```bash
docker build -t skyspy-discord .
docker run -d \
  -e DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..." \
  -e SKYSPY_URL="http://skyspy-api:5000" \
  --name skyspy-discord \
  skyspy-discord
```

</Accordion>

---

## Next Steps

<Cards columns={2}>
  <Card title="Track Specific Aircraft" icon="plane" href="/docs/track-aircraft">
    Add alerts for specific tail numbers
  </Card>
  <Card title="Safety & Alerts" icon="bell" href="/docs/safety-and-alerts">
    Configure custom alert rules in SkySpy
  </Card>
</Cards>
