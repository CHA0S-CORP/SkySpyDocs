---
title: Get METARs by Location
excerpt: |-
  Get METAR observations within a geographic area.

  Returns current weather observations from airports sorted by distance.

  **Response includes:**
  - Raw METAR text
  - Decoded values (temp, wind, visibility, etc.)
  - Flight category (VFR/MVFR/IFR/LIFR)
  - Distance from center point

  **Data Source:** Uses Redis cache when available (5-minute TTL).
  Falls back to Aviation Weather Center API.

  **Parameters:**
  - `lat`, `lon` - Center point coordinates
  - `radius` - Search radius in nautical miles (default 100)
  - `limit` - Maximum results (default 20)
  - `hours` - Hours of observations (default 2)
api:
  file: openapi.json
  operationId: get_metars_by_location_api_v1_aviation_metars_get
hidden: false
---