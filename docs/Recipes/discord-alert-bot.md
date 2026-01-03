---
title: "Discord Alert Bot"
slug: "discord-alert-bot"
excerpt: "Send real-time aircraft alerts to your Discord server using webhooks."
hidden: false
---

Build a Discord bot that sends aircraft alerts to your server in real-time.

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    SKYSPY["📡 SkySpy API"] -->|"📤 SSE Stream"| BOT["🐍 Python Bot"]
    BOT -->|"📨 Webhook"| DISCORD["💬 Discord Channel"]

    style SKYSPY fill:#0d4f8b,stroke:#3b82f6,stroke-width:2px,color:#fff
    style BOT fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
    style DISCORD fill:#5865F2,stroke:#7289da,stroke-width:2px,color:#fff
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

---

## Quick Setup

<Steps>
  <Step title="Create Discord Webhook">
    Right-click channel → **Edit Channel** → **Integrations** → **Webhooks** → **New Webhook**
  </Step>
  <Step title="Install Dependencies">
    ```bash
    pip install sseclient-py requests discord-webhook
    ```
  </Step>
  <Step title="Configure & Run">
    ```bash
    export DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..."
    python discord_bot.py
    ```
  </Step>
</Steps>

---

## Implementation

<Cards columns={2}>
  <Card title="Bot Code" icon="code" href="/docs/discord-alert-bot/code">
    Complete Python bot implementation
  </Card>
  <Card title="Run as Service" icon="server" href="/docs/discord-alert-bot/service">
    Systemd and Docker deployment
  </Card>
</Cards>

---

## Example Output

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

## Next Steps

<Cards columns={2}>
  <Card title="Track Specific Aircraft" icon="plane" href="/docs/track-aircraft">
    Add alerts for specific tail numbers
  </Card>
  <Card title="Safety & Alerts" icon="bell" href="/docs/safety-and-alerts">
    Configure custom alert rules in SkySpy
  </Card>
</Cards>
