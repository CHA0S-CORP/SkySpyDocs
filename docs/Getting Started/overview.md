---
title: "Overview"
slug: "overview"
excerpt: "Introduction to SkySpy, a real-time ADS-B aircraft tracking and monitoring system."
hidden: false
---

# SkySpy

**Real-time ADS-B aircraft tracking and monitoring system with a web-based dashboard.**

SkySpy is a sophisticated aircraft tracking platform that captures position data from 1090MHz Mode S and 978MHz UAT receivers. It displays aircraft on an interactive map, monitors safety conditions, and provides advanced features like custom alerts, weather integration, and push notifications.

![SkySpy Demo](https://raw.githubusercontent.com/cha0s-corp/skyspy/main/docs/screenshots/map-view.gif)

## Key Features

* **Real-Time Aircraft Tracking**: Live position updates from ADS-B receivers with distance, altitude, speed, and climb rate.
* **Interactive Map Dashboard**: Canvas-based radar display with aircraft icons, flight paths, and detailed information panels.
* **Safety Monitoring**: TCAS RA/TA detection, proximity alerts, extreme vertical speed warnings, and emergency squawk detection (7700/7600/7500).
* **Custom Alert Rules**: Flexible AND/OR logic conditions on ICAO, callsign, squawk, altitude, distance, aircraft type, and military status.
* **Historical Data**: PostgreSQL-backed sighting history with session tracking and analytics.
* **Aviation Weather**: METARs, TAFs, PIREPs, SIGMETs, and G-AIRMET integration.
* **Push Notifications**: Apprise integration supporting 80+ services (Pushover, Telegram, Slack, Discord, email, etc.).
* **Aircraft Information**: Registration lookups, photos, airframe data, and operator information.
* **ACARS/VDL2 Messages**: Aircraft communication message reception and display.

> 📘 Data Sources
> 
> SkySpy integrates with **Ultrafeeder** (readsb/dump1090) for ADS-B data, **dump978** for UAT data, and external APIs like **OpenSky Network** and **Aviation Weather Center** for enriched metadata.