---
title: "Configuration"
slug: "configuration"
excerpt: "Comprehensive guide to SkySpy environment variables and settings."
hidden: false
---

SkySpy is configured via environment variables in your `.env` file. Copy the sample file to get started:

```bash
cp .env.test.sample .env
```

## Required Settings

> 🚧 Warning
>
> These variables **must** be set for SkySpy to function properly.

| Variable | Description | Example |
| :--- | :--- | :--- |
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@localhost:5432/adsb` |
| `ULTRAFEEDER_HOST` | Hostname of your ADS-B receiver | `ultrafeeder` |
| `ULTRAFEEDER_PORT` | Port for the ADS-B JSON API | `80` |
| `FEEDER_LAT` | Your receiver's latitude | `47.9377` |
| `FEEDER_LON` | Your receiver's longitude | `-121.9687` |

> 🚧 **Coordinates Required**
>
> `FEEDER_LAT` and `FEEDER_LON` are used for distance calculations and proximity alerts. Make sure these match your receiver's actual location.

## Polling & Storage

Control how frequently SkySpy fetches data and writes to the database:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `POLLING_INTERVAL` | `2` | Seconds between aircraft data fetches |
| `DB_STORE_INTERVAL` | `10` | Seconds between database writes |

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    subgraph Timing["⏱️ Data Flow Timing"]
        direction LR
        RX["📡 Receiver"] -->|"⚡ 2s"| API["🚀 API"]
        API -->|"💾 10s"| DB[("🗄️ Database")]
    end

    style Timing fill:#0d4f8b,stroke:#3b82f6,stroke-width:2px,color:#fff
```

> 📘 Info
>
> **Performance Tip** — Lower `POLLING_INTERVAL` values provide more responsive tracking but increase CPU and network usage. The default of 2 seconds works well for most setups.

## Safety Monitoring

Configure the safety analysis engine that detects TCAS events, proximity alerts, and emergency squawks:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `SAFETY_MONITORING_ENABLED` | `true` | Enable the safety analysis engine |
| `SAFETY_PROXIMITY_NM` | `1.0` | Proximity alert threshold (nautical miles) |
| `SAFETY_ALTITUDE_DIFF_FT` | `1000` | Vertical separation threshold (feet) |

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart TB
    subgraph Safety["🛡️ Safety Engine"]
        direction TB
        PROX["📏 Proximity Check<br/>< 1.0 NM horizontal"]
        ALT["📐 Altitude Check<br/>< 1000 ft vertical"]
        TCAS["⚠️ TCAS Detection<br/>RA/TA flags"]
        EMER["🚨 Emergency Squawks<br/>7700/7600/7500"]
    end

    PROX --> ALERT["🔔 Alert Triggered"]
    ALT --> ALERT
    TCAS --> ALERT
    EMER --> ALERT

    style Safety fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
    style ALERT fill:#991b1b,stroke:#ef4444,stroke-width:2px,color:#fff
```

## Notifications

SkySpy uses [Apprise](https://github.com/caronc/apprise) for push notifications, supporting 80+ services.

- **Pushover** - iOS/Android push
- **Telegram** - Bot messages
- **Discord** - Webhook alerts
- **Slack** - Channel posts

Configure multiple services by separating URLs with semicolons:

```bash
APPRISE_URLS="pushover://user_key@app_token;tgram://bot_token/chat_id"
NOTIFICATION_COOLDOWN=300
```

| Variable | Default | Description |
| :--- | :--- | :--- |
| `APPRISE_URLS` | — | Semicolon-separated list of Apprise URLs |
| `NOTIFICATION_COOLDOWN` | `300` | Minimum seconds between repeat notifications for the same alert |

<Accordion title="Pushover Setup" icon="fa-mobile-alt">

1. Create an app at [pushover.net](https://pushover.net)
2. Copy your User Key and API Token
3. Add to `.env`:
   ```bash
   APPRISE_URLS="pushover://USER_KEY@API_TOKEN"
   ```

</Accordion>

<Accordion title="Telegram Setup" icon="fa-paper-plane">

1. Create a bot via [@BotFather](https://t.me/botfather)
2. Get your chat ID from [@userinfobot](https://t.me/userinfobot)
3. Add to `.env`:
   ```bash
   APPRISE_URLS="tgram://BOT_TOKEN/CHAT_ID"
   ```

</Accordion>

<Accordion title="Discord Setup" icon="fa-discord">

1. Create a webhook in your Discord channel settings
2. Copy the webhook URL
3. Add to `.env`:
   ```bash
   APPRISE_URLS="discord://WEBHOOK_ID/WEBHOOK_TOKEN"
   ```

</Accordion>

<Accordion title="Slack Setup" icon="fa-slack">

1. Create an incoming webhook in Slack
2. Copy the webhook URL tokens
3. Add to `.env`:
   ```bash
   APPRISE_URLS="slack://TOKEN_A/TOKEN_B/TOKEN_C"
   ```

</Accordion>

See the [Apprise documentation](https://github.com/caronc/apprise/wiki) for the full list of 80+ supported services.

## Advanced Integrations

<Accordion title="UAT 978MHz Receiver" icon="fa-broadcast-tower">

Receive traffic from 978MHz UAT broadcasts (common for GA aircraft in the US):

| Variable | Default | Description |
| :--- | :--- | :--- |
| `DUMP978_HOST` | — | Hostname of your dump978 receiver |
| `DUMP978_PORT` | `30979` | Port for dump978 JSON output |

> 📘 Info
>
> UAT is primarily used by general aviation aircraft in the United States below 18,000 feet. Adding a 978MHz receiver significantly increases coverage of small aircraft.

</Accordion>

<Accordion title="Redis Pub/Sub" icon="fa-database">

Enable Redis for pub/sub messaging in multi-worker deployments:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `REDIS_URL` | — | Redis connection string (e.g., `redis://localhost:6379`) |

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    subgraph Workers["⚙️ Multiple API Workers"]
        W1["🔧 Worker 1"]
        W2["🔧 Worker 2"]
        W3["🔧 Worker 3"]
    end

    subgraph Redis["⚡ Redis Pub/Sub"]
        R[("🗄️ Redis")]
    end

    subgraph Clients["📱 SSE Clients"]
        C1["👤 Client A"]
        C2["👤 Client B"]
        C3["👤 Client C"]
    end

    W1 --> R
    W2 --> R
    W3 --> R
    R --> C1
    R --> C2
    R --> C3

    style Workers fill:#0d4f8b,stroke:#3b82f6,stroke-width:2px,color:#fff
    style Redis fill:#991b1b,stroke:#ef4444,stroke-width:2px,color:#fff
    style Clients fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
```

</Accordion>

<Accordion title="ACARS/VDL2 Messages" icon="fa-comment-alt">

Receive and display aircraft communication messages:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `ACARS_ENABLED` | `false` | Enable ACARS message reception |
| `ACARS_PORT` | `5555` | Port to receive ACARS JSON messages |

</Accordion>

<Accordion title="Photo Cache" icon="fa-images">

Cache aircraft photos locally to reduce API calls:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `PHOTO_CACHE_ENABLED` | `false` | Enable local photo caching |
| `PHOTO_CACHE_DIR` | `/data/photos` | Directory to store cached images |

</Accordion>

## Complete Configuration Example

### Full .env example

```bash
# ===================
# Required Settings
# ===================
DATABASE_URL=postgresql://skyspy:password@postgres:5432/skyspy
ULTRAFEEDER_HOST=ultrafeeder
ULTRAFEEDER_PORT=80
FEEDER_LAT=47.9377
FEEDER_LON=-121.9687

# ===================
# Polling & Storage
# ===================
POLLING_INTERVAL=2
DB_STORE_INTERVAL=10

# ===================
# Safety Monitoring
# ===================
SAFETY_MONITORING_ENABLED=true
SAFETY_PROXIMITY_NM=1.0
SAFETY_ALTITUDE_DIFF_FT=1000

# ===================
# Notifications
# ===================
APPRISE_URLS=pushover://user@token;tgram://bot/chat
NOTIFICATION_COOLDOWN=300

# ===================
# Optional: UAT 978MHz
# ===================
# DUMP978_HOST=dump978
# DUMP978_PORT=30979

# ===================
# Optional: Redis
# ===================
# REDIS_URL=redis://redis:6379

# ===================
# Optional: ACARS
# ===================
# ACARS_ENABLED=true
# ACARS_PORT=5555

# ===================
# Optional: Photo Cache
# ===================
# PHOTO_CACHE_ENABLED=true
# PHOTO_CACHE_DIR=/data/photos
```

## Next Steps

- **Safety & Alerts** - Configure custom alert rules and notification triggers. [Learn more →](/docs/safety-and-alerts)
- **Real-Time API** - Connect to live data streams via Socket.IO. [Learn more →](/docs/real-time-api)
