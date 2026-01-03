---
title: Get PIREPs by Location
excerpt: |-
  Get Pilot Reports (PIREPs) within a geographic area.

  PIREPs are reports from pilots about actual conditions:
  - Turbulence intensity and altitude
  - Icing conditions
  - Cloud tops/bases
  - Visibility
  - Other significant weather

  Useful for understanding real conditions aloft.

  **Data Source:** Uses database cache for recent PIREPs.
  Fetches from Aviation Weather Center API and stores new reports.

  **Parameters:**
  - `lat`, `lon` - Center point coordinates
  - `radius` - Search radius in nautical miles (default 100)
  - `limit` - Maximum results (default 50)
  - `hours` - Hours of reports to include (default 2)
api:
  file: openapi.json
  operationId: get_pireps_by_location_api_v1_aviation_pireps_get
hidden: false
---