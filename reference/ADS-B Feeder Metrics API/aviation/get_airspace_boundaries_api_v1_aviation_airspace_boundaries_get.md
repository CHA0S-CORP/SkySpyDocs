---
title: Get Static Airspace Boundaries
excerpt: |-
  Get static airspace boundary polygons for controlled airspace areas.

  Returns GeoJSON-style polygon data for:
  - **Class B** - Major hub airports (LAX, JFK, ORD, etc.)
  - **Class C** - Busy airports with radar approach control
  - **Class D** - Airports with control towers
  - **MOA** - Military Operations Areas
  - **Restricted** - Restricted airspace

  **Note:** These are approximate boundaries for visualization purposes.
  Refer to official FAA sectional charts for precise limits.

  **Parameters:**
  - `lat`, `lon` - Center point to filter nearby airspaces (optional)
  - `radius` - Search radius in nautical miles (default 100, max 500)
  - `airspace_class` - Filter by class (B, C, D, MOA, Restricted)

  **Data:** Embedded static data covering major US airspaces
api:
  file: openapi (1).json
  operationId: get_airspace_boundaries_api_v1_aviation_airspace_boundaries_get
hidden: false
---