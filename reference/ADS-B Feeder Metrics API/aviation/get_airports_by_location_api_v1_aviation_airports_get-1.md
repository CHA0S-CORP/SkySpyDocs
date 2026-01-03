---
title: Get Airports by Location
excerpt: >-
  Find airports within a geographic area.


  Returns airports sorted by distance from the specified coordinates.

  Useful for finding alternates or nearby airfields.


  **Data Source:** Uses locally cached data (refreshed daily) for fast
  responses.

  Falls back to Aviation Weather Center API if cache is empty.


  **Parameters:**

  - `lat`, `lon` - Center point coordinates

  - `radius` - Search radius in nautical miles (default 50)

  - `limit` - Maximum results (default 20)
api:
  file: openapi.json
  operationId: get_airports_by_location_api_v1_aviation_airports_get
hidden: false
---