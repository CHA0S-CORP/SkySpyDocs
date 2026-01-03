---
title: "Overview"
slug: "overview"
excerpt: "Introduction to SkySpy, a real-time ADS-B aircraft tracking and monitoring system."
hidden: false
---

SkySpy is a real-time aircraft tracking platform that captures ADS-B position data from 1090MHz Mode S and 978MHz UAT receivers. It displays aircraft on an interactive map, monitors safety conditions, and provides custom alerts, weather integration, and push notifications.

![SkySpy Demo](https://raw.githubusercontent.com/cha0s-corp/skyspy/main/docs/screenshots/aircraft-detail.gif)

## What You Can Do with SkySpy

| Capability | Description |
| :--- | :--- |
| **Track Aircraft** | Monitor live positions with distance, altitude, speed, and climb rate from your ADS-B receiver |
| **Detect Safety Events** | Get alerts for TCAS RA/TA, proximity warnings, and emergency squawks (7700/7600/7500) |
| **Create Custom Alerts** | Build rules with AND/OR logic on ICAO, callsign, squawk, altitude, distance, and more |
| **View Weather Data** | Access METARs, TAFs, PIREPs, SIGMETs, and G-AIRMETs for your area |
| **Receive Notifications** | Push alerts via 80+ services including Pushover, Telegram, Slack, and Discord |
| **Explore Aircraft Data** | Look up registrations, photos, airframe details, and operator information |

## Architecture

SkySpy consists of two main components:

- **Backend API** — Python/FastAPI service that processes ADS-B data, manages alerts, and provides REST/SSE endpoints
- **Web Dashboard** — React-based frontend with an interactive canvas radar display

> 📘 Data Sources
>
> SkySpy integrates with [Ultrafeeder](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder) (readsb/dump1090) for ADS-B data, dump978 for UAT data, and external APIs like OpenSky Network and Aviation Weather Center for enriched metadata.

## Next Steps

<Cards columns={2}>
  <Card title="Quick Start" icon="rocket" href="/docs/quick-start">
    Deploy SkySpy with Docker Compose in minutes
  </Card>
  <Card title="Configuration" icon="gear" href="/docs/configuration">
    Customize receiver settings, alerts, and integrations
  </Card>
  <Card title="Safety & Alerts" icon="bell" href="/docs/safety-and-alerts">
    Set up custom alert rules and notifications
  </Card>
  <Card title="Real-Time API" icon="bolt" href="/docs/real-time-api">
    Connect to the SSE stream for live aircraft data
  </Card>
</Cards>