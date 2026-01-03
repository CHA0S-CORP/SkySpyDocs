---
title: Get GeoJSON Map Overlays
excerpt: |-
  Get cached GeoJSON boundary data for map overlays.

  **Data Types:**
  - `states` - US state and province boundaries
  - `countries` - Country boundaries
  - `water` - Major lakes and water bodies

  **Data Source:** Cached from Natural Earth (refreshed daily).

  **Parameters:**
  - `data_type` - Type of GeoJSON data to retrieve
  - `lat`, `lon` - Optional center point to filter nearby features
  - `radius` - Search radius in nautical miles (default 500)
api:
  file: openapi.json
  operationId: get_geojson_overlays_api_v1_aviation_geojson__data_type__get
hidden: false
---