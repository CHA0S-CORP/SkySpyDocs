---
title: Get Airspace Advisories by Location
excerpt: >-
  Get active airspace advisories (G-AIRMETs) within a geographic area.


  G-AIRMETs provide graphical depictions of en route aviation weather hazards:

  - IFR conditions (low ceilings/visibility)

  - Mountain obscuration

  - Turbulence (low, moderate, high level)

  - Icing (low, moderate, high level)

  - Freezing level

  - Strong surface winds


  **Note:** This endpoint returns active G-AIRMET advisories from Aviation
  Weather Center.

  For static airspace boundaries (Class B/C/D, MOAs, Restricted areas), consult 

  FAA sectional charts or the FAA UAS Data Delivery System at 

  https://udds-faa.opendata.arcgis.com/


  **Parameters:**

  - `lat`, `lon` - Center point coordinates (for distance calculation)

  - `hazard` - Filter by hazard type (optional)


  **Data Source:** aviationweather.gov
api:
  file: openapi (1).json
  operationId: get_airspaces_by_location_api_v1_aviation_airspaces_get
hidden: false
---