---
title: Get Aircraft Statistics
excerpt: >-
  Get aggregate statistics about currently tracked aircraft with optional
  filters.


  **Filters:**

  - **category**: Filter by aircraft category (A0-D7, comma-separated)

  - **military_only**: Only include military aircraft

  - **min_altitude/max_altitude**: Filter by altitude range in feet

  - **min_distance/max_distance**: Filter by distance range in nautical miles


  Returns:

  - **total**: Total aircraft count (after filters)

  - **with_position**: Aircraft with valid GPS position

  - **military**: Military aircraft count

  - **emergency**: Aircraft squawking emergency codes (7500/7600/7700)

  - **categories**: Count by aircraft category (A0-D7)

  - **altitude**: Count by altitude band (ground, low, medium, high)

  - **messages**: Total messages received by feeder


  Altitude bands:

  - Ground: On ground or ≤0 ft

  - Low: 1-9,999 ft

  - Medium: 10,000-29,999 ft

  - High: ≥30,000 ft
api:
  file: openapi (1).json
  operationId: get_aircraft_stats_api_v1_aircraft_stats_get
hidden: false
---