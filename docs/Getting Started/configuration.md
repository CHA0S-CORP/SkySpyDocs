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

These variables must be set for SkySpy to function:

| Variable | Description | Example |
| :--- | :--- | :--- |
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@localhost:5432/adsb` |
| `ULTRAFEEDER_HOST` | Hostname of your ADS-B receiver | `ultrafeeder` |
| `ULTRAFEEDER_PORT` | Port for the ADS-B JSON API | `80` |
| `FEEDER_LAT` | Your receiver's latitude | `47.9377` |
| `FEEDER_LON` | Your receiver's longitude | `-121.9687` |

> 🚧 Coordinates Required
>
> `FEEDER_LAT` and `FEEDER_LON` are used for distance calculations and proximity alerts. Make sure these match your receiver's actual location.

## Polling & Storage

Control how frequently SkySpy fetches data and writes to the database:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `POLLING_INTERVAL` | `2` | Seconds between aircraft data fetches |
| `DB_STORE_INTERVAL` | `10` | Seconds between database writes |

> 📘 Performance Tip
>
> Lower `POLLING_INTERVAL` values provide more responsive tracking but increase CPU and network usage. The default of 2 seconds works well for most setups.

## Safety Monitoring

Configure the safety analysis engine that detects TCAS events, proximity alerts, and emergency squawks:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `SAFETY_MONITORING_ENABLED` | `true` | Enable the safety analysis engine |
| `SAFETY_PROXIMITY_NM` | `1.0` | Proximity alert threshold (nautical miles) |
| `SAFETY_ALTITUDE_DIFF_FT` | `1000` | Vertical separation threshold (feet) |

## Notifications

SkySpy uses [Apprise](https://github.com/caronc/apprise) for push notifications, supporting 80+ services. Configure multiple services by separating URLs with semicolons:

```bash
APPRISE_URLS="pushover://user_key@app_token;tgram://bot_token/chat_id"
NOTIFICATION_COOLDOWN=300
```

| Variable | Default | Description |
| :--- | :--- | :--- |
| `APPRISE_URLS` | — | Semicolon-separated list of Apprise URLs |
| `NOTIFICATION_COOLDOWN` | `300` | Minimum seconds between repeat notifications for the same alert |

<Accordion title="Common Apprise URL formats">

| Service | URL Format |
| :--- | :--- |
| Pushover | `pushover://user_key@app_token` |
| Telegram | `tgram://bot_token/chat_id` |
| Discord | `discord://webhook_id/webhook_token` |
| Slack | `slack://token_a/token_b/token_c` |
| Email | `mailto://user:pass@smtp.example.com?to=you@example.com` |

See the [Apprise documentation](https://github.com/caronc/apprise/wiki) for the full list of supported services.

</Accordion>

## Advanced Integrations

### UAT 978MHz Receiver

Receive traffic from 978MHz UAT broadcasts (common for GA aircraft in the US):

| Variable | Default | Description |
| :--- | :--- | :--- |
| `DUMP978_HOST` | — | Hostname of your dump978 receiver |
| `DUMP978_PORT` | `30979` | Port for dump978 JSON output |

### Redis

Enable Redis for pub/sub messaging in multi-worker deployments:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `REDIS_URL` | — | Redis connection string (e.g., `redis://localhost:6379`) |

### ACARS/VDL2 Messages

Receive and display aircraft communication messages:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `ACARS_ENABLED` | `false` | Enable ACARS message reception |
| `ACARS_PORT` | `5555` | Port to receive ACARS JSON messages |

### Photo Cache

Cache aircraft photos locally to reduce API calls:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `PHOTO_CACHE_ENABLED` | `false` | Enable local photo caching |
| `PHOTO_CACHE_DIR` | `/data/photos` | Directory to store cached images |
