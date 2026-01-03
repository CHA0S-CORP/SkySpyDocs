---
title: Get Historical Statistics
excerpt: |-
  Get aggregate statistics about historical data with optional filters.

  **Filters:**
  - **military_only**: Only include military aircraft
  - **min_altitude/max_altitude**: Filter by altitude range
  - **aircraft_type**: Filter by type code (B738, A320, comma-separated)
  - **callsign**: Partial callsign match

  Returns:
  - Total sightings and sessions
  - Unique aircraft count
  - Military session count
  - Time range covered
  - Altitude and distance statistics
api:
  file: openapi.json
  operationId: get_stats_api_v1_history_stats_get
hidden: false
---