---
title: Get Per-Aircraft Safety Statistics
excerpt: |-
  Get safety event statistics aggregated by aircraft.

  Shows which aircraft have been involved in the most safety events,
  their event breakdown, and worst severity levels.

  **Filters:**
  - **event_type**: Filter by specific event type(s) - comma-separated
  - **severity**: Filter by severity level(s) - comma-separated
  - **min_events**: Only include aircraft with at least this many events
api:
  file: openapi (1).json
  operationId: get_aircraft_stats_api_v1_safety_stats_aircraft_get
hidden: false
---