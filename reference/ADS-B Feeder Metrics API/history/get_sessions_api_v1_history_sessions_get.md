---
title: Query Tracking Sessions
excerpt: |-
  Query aircraft tracking sessions.

  A session represents a continuous period of tracking an aircraft,
  from first detection to last signal. Sessions aggregate multiple
  sightings into summary statistics.

  Session data includes:
  - Duration and position count
  - Closest approach distance
  - Altitude range (min/max)
  - Maximum vertical rate
  - Aircraft type and military flag
api:
  file: openapi (1).json
  operationId: get_sessions_api_v1_history_sessions_get
hidden: false
---