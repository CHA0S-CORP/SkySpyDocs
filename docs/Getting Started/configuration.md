---
title: "Configuration"
slug: "configuration"
excerpt: "Comprehensive guide to SkySpy environment variables and settings."
hidden: false
---

SkySpy is configured primarily via environment variables defined in your `.env` file.

## Required Settings

| Variable | Description | Example |
| :--- | :--- | :--- |
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@localhost:5432/adsb` |
| `ULTRAFEEDER_HOST` | Hostname of your ADS-B receiver | `ultrafeeder` |
| `ULTRAFEEDER_PORT` | Port for the ADS-B JSON API | `80` |
| `FEEDER_LAT` | Latitude for distance calculations | `47.9377` |
| `FEEDER_LON` | Longitude for distance calculations | `-121.9687` |

## Optional Settings

### Polling & Storage
| Variable | Default | Description |
| :--- | :--- | :--- |
| `POLLING_INTERVAL` | `2` | Seconds between aircraft polls from the receiver. |
| `DB_STORE_INTERVAL` | `10` | Seconds between writing positions to the database. |

### Safety Monitoring
| Variable | Default | Description |
| :--- | :--- | :--- |
| `SAFETY_MONITORING_ENABLED` | `true` | Enable/Disable safety analysis engine. |
| `SAFETY_PROXIMITY_NM` | `1.0` | Proximity alert distance threshold in Nautical Miles. |
| `SAFETY_ALTITUDE_DIFF_FT` | `1000` | Vertical separation threshold in Feet. |

### Notifications
SkySpy uses **Apprise** for notifications. You can configure multiple services separated by semicolons.

```bash
APPRISE_URLS="pushover://key@token;telegram://token/chatid"
NOTIFICATION_COOLDOWN=300  # Seconds between repeat notifications

```

### Advanced Integrations

* **UAT 978MHz**: Set `DUMP978_HOST` and `DUMP978_PORT`.
* **Redis**: Set `REDIS_URL` (e.g., `redis://localhost:6379`) to enable pub/sub for multi-worker deployments.
* **ACARS**: Set `ACARS_ENABLED=true` and `ACARS_PORT=5555`.
* **Photo Cache**: Set `PHOTO_CACHE_ENABLED=true` and `PHOTO_CACHE_DIR=/data/photos` to cache aircraft images locally.
