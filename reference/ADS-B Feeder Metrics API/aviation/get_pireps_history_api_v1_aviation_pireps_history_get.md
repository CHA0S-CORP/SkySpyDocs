---
title: Get Historical PIREPs
excerpt: >-
  Get historical Pilot Reports (PIREPs) from the database.


  Query stored PIREPs by time range with optional location and condition
  filters.


  **Parameters:**

  - `start` - Start time (ISO 8601 format, default: 24 hours ago)

  - `end` - End time (ISO 8601 format, default: now)

  - `lat`, `lon` - Center point for location filter (optional)

  - `radius` - Search radius in nautical miles (default 100)

  - `turbulence` - Filter to only turbulence reports

  - `icing` - Filter to only icing reports

  - `limit` - Maximum results (default 200)


  **Data Source:** Database (PIREPs are stored from API calls)
api:
  file: openapi.json
  operationId: get_pireps_history_api_v1_aviation_pireps_history_get
hidden: false
---