---
title: Get Navaids by Location
excerpt: >-
  Find navigation aids within a geographic area.


  Returns VORs, VORTACs, NDBs, and other navaids sorted by distance.

  Useful for flight planning and navigation.


  **Data Source:** Uses locally cached data (refreshed daily) for fast
  responses.

  Falls back to Aviation Weather Center API if cache is empty.


  **Navaid Types:**

  - VOR - VHF Omnidirectional Range

  - VORTAC - VOR with TACAN (military DME)

  - VOR-DME - VOR with Distance Measuring Equipment

  - NDB - Non-Directional Beacon

  - TACAN - Tactical Air Navigation (military)

  - DME - Distance Measuring Equipment


  **Parameters:**

  - `lat`, `lon` - Center point coordinates

  - `radius` - Search radius in nautical miles (default 50)

  - `limit` - Maximum results (default 20)

  - `type` - Filter by navaid type (optional)
api:
  file: openapi.json
  operationId: get_navaids_by_location_api_v1_aviation_navaids_get
hidden: false
---