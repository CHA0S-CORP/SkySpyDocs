---
title: "Recipes"
slug: "recipes"
excerpt: "Step-by-step guides for common SkySpy use cases and integrations."
hidden: false
---

Recipes are practical, copy-paste examples that solve real-world problems. Each recipe includes working code in Shell, Go, Python, and JavaScript with step-by-step explanations.

## Quick Start

New to SkySpy? Start with these foundational recipes:

| Recipe | Description | Difficulty |
|--------|-------------|------------|
| [Discord Alert Bot](/docs/discord-alert-bot) | Send aircraft alerts to Discord | 🟢 Beginner |
| [Military Aircraft Spotter](/docs/military-spotter) | Track military flights | 🟢 Beginner |
| [Track Specific Aircraft](/docs/track-aircraft) | Monitor by tail number | 🟢 Beginner |
| [Export to CSV](/docs/export-csv) | Log sightings to files | 🟢 Beginner |

---

## Notifications & Alerts

Send aircraft alerts to messaging platforms and notification services.

| Recipe | Description | Difficulty |
|--------|-------------|------------|
| [Discord Alert Bot](/docs/discord-alert-bot) | Rich Discord embeds with webhooks | 🟢 Beginner |
| [Telegram Bot](/docs/telegram-bot) | Alerts with inline keyboards | 🟢 Beginner |
| [Slack Integration](/docs/slack-integration) | Block Kit formatted messages | 🟢 Beginner |
| [Email Alerts](/docs/email-alerts) | SMTP email notifications | 🟢 Beginner |
| [Pushover Notifications](/docs/pushover-notifications) | Mobile push with priorities | 🟢 Beginner |
| [ntfy.sh Integration](/docs/ntfy-integration) | Self-hosted push notifications | 🟢 Beginner |
| [Webhook Notifications](/docs/webhook-notifications) | Generic webhook POST | 🟢 Beginner |
| [Military Aircraft Spotter](/docs/military-spotter) | Built-in military alerts | 🟢 Beginner |
| [Emergency Alert Monitor](/docs/emergency-monitor) | Track 7700/7600/7500 squawks | 🟢 Beginner |
| [Microsoft Teams Integration](/docs/teams-integration) | Adaptive Cards to Teams channels | 🟢 Beginner |
| [IFTTT Applets](/docs/ifttt-integration) | Trigger IFTTT automations | 🟢 Beginner |
| [Gotify Notifications](/docs/gotify-notifications) | Self-hosted push with Gotify | 🟢 Beginner |

---

## Data & Analytics

Store, query, and analyze aircraft data.

| Recipe | Description | Difficulty |
|--------|-------------|------------|
| [Export to CSV](/docs/export-csv) | Log to CSV files | 🟢 Beginner |
| [JSON Lines Logging](/docs/jsonl-logging) | Append-only JSON logs | 🟢 Beginner |
| [SQLite Local DB](/docs/sqlite-db) | Lightweight local storage | 🟢 Beginner |
| [Grafana Dashboard](/docs/grafana-dashboard) | Real-time metrics visualization | 🟡 Intermediate |
| [Prometheus Metrics](/docs/prometheus-metrics) | Expose metrics for Prometheus | 🟡 Intermediate |
| [InfluxDB Logging](/docs/influxdb-logging) | Time-series data storage | 🟡 Intermediate |
| [Redis Cache](/docs/redis-cache) | Real-time cache for fast lookups | 🟡 Intermediate |
| [PostgreSQL Archive](/docs/postgresql-archive) | Persistent aircraft history | 🟡 Intermediate |
| [MongoDB Storage](/docs/mongodb-storage) | NoSQL aircraft logging | 🟡 Intermediate |
| [Aircraft Statistics](/docs/aircraft-statistics) | Daily and weekly flight reports | 🟡 Intermediate |

---

## Geographic & Filtering

Filter aircraft by location and other criteria.

| Recipe | Description | Difficulty |
|--------|-------------|------------|
| [Bounding Box Filter](/docs/bounding-box-filter) | Aircraft within lat/lon bounds | 🟢 Beginner |
| [Radius Filter](/docs/radius-filter) | Aircraft within X miles of point | 🟢 Beginner |
| [Track Specific Aircraft](/docs/track-aircraft) | Monitor specific tail numbers | 🟢 Beginner |
| [Altitude Band Filter](/docs/altitude-filter) | Filter by altitude ranges | 🟢 Beginner |
| [Speed Filtering](/docs/speed-filter) | Detect fast or slow aircraft | 🟢 Beginner |
| [Geofence Alerts](/docs/geofence-alerts) | Enter/exit zone notifications | 🟡 Intermediate |
| [Airport Proximity Monitor](/docs/airport-proximity) | Track arrivals and departures | 🟡 Intermediate |

---

## Visualization & Display

Display aircraft data in various formats.

| Recipe | Description | Difficulty |
|--------|-------------|------------|
| [Leaflet.js Map](/docs/leaflet-map) | Interactive web map | 🟢 Beginner |
| [Terminal Dashboard](/docs/terminal-dashboard) | CLI aircraft display | 🟢 Beginner |
| [OBS Browser Overlay](/docs/obs-overlay) | Stream overlay for OBS | 🟢 Beginner |
| [Build a Live Dashboard](/docs/live-dashboard) | React dashboard with live updates | 🟡 Intermediate |
| [MapLibre Dashboard](/docs/maplibre-dashboard) | Open-source map visualization | 🟡 Intermediate |

---

## Smart Home & IoT

Integrate with home automation systems.

| Recipe | Description | Difficulty |
|--------|-------------|------------|
| [Philips Hue Alerts](/docs/philips-hue) | Flash smart lights on events | 🟢 Beginner |
| [Home Assistant](/docs/home-assistant) | HA sensors and automations | 🟡 Intermediate |
| [MQTT Publisher](/docs/mqtt-publisher) | Publish to MQTT brokers | 🟡 Intermediate |
| [Node-RED Flows](/docs/node-red-flows) | Visual automation flows | 🟡 Intermediate |

---

## Specialty & Fun

Unique use cases and specialized tracking.

| Recipe | Description | Difficulty |
|--------|-------------|------------|
| [VIP Aircraft Tracker](/docs/vip-tracker) | Track known VIP tail numbers | 🟢 Beginner |
| [Helicopter Watch](/docs/helicopter-watch) | Dedicated helicopter monitoring | 🟢 Beginner |
| [Rare Aircraft Spotter](/docs/rare-spotter) | Alert on unusual aircraft types | 🟡 Intermediate |

---

## Recipe Structure

Each recipe includes:

1. **Prerequisites** - What you need before starting
2. **What You'll Build** - Overview of the end result
3. **Code Examples** - Working code in 4 languages
4. **Configuration** - Environment variables and options
5. **Testing & Verification** - How to verify it works
6. **Troubleshooting** - Common issues and solutions
7. **Related Recipes** - Links to related content

---

## Difficulty Levels

| Level | Description |
|-------|-------------|
| 🟢 Beginner | Copy-paste ready, minimal setup |
| 🟡 Intermediate | Some configuration required |
| 🔴 Advanced | Complex setup or deep knowledge needed |

---

## Contributing Recipes

Have a recipe idea? We welcome contributions! Check the [template](/docs/recipes/_template) for the standard format.
