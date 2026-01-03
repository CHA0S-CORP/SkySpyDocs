---
title: Get Historical Airspace Advisories
excerpt: |-
  Get historical airspace advisories for a time range.

  Returns past G-AIRMET advisories stored in the database.

  **Parameters:**
  - `start` - Start time (ISO 8601 format, default: 24 hours ago)
  - `end` - End time (ISO 8601 format, default: now)
  - `hazard` - Filter by hazard type

  **Data Source:** Database (refreshed every 5 minutes from aviationweather.gov)
api:
  file: openapi (1).json
  operationId: get_airspace_history_api_v1_aviation_airspaces_history_get
hidden: false
---