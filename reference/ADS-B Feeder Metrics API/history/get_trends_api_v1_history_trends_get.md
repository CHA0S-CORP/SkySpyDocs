---
title: Get Historical Trends
excerpt: |-
  Get time-series data for aircraft metrics over a specified time range.

  **Parameters:**
  - **hours**: Time range (1-168 hours)
  - **interval**: Aggregation interval (15min, hour, day)
  - **military_only**: Filter to military aircraft only
  - **aircraft_type**: Filter by type codes (comma-separated)

  Returns metrics per interval:
  - Aircraft count and unique aircraft
  - Average/max altitude, distance, speed
  - Position count
api:
  file: openapi (1).json
  operationId: get_trends_api_v1_history_trends_get
hidden: false
---