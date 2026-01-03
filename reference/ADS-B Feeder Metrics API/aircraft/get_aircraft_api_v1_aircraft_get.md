---
title: Get All Tracked Aircraft
excerpt: |-
  Retrieve all aircraft currently being tracked by the ADS-B receiver.

  Each aircraft includes:
  - **hex**: ICAO 24-bit address (unique identifier)
  - **flight**: Callsign/flight number
  - **lat/lon**: Position coordinates
  - **alt_baro**: Barometric altitude in feet
  - **gs**: Ground speed in knots
  - **track**: Ground track in degrees
  - **baro_rate**: Vertical rate in feet/minute
  - **squawk**: Transponder code
  - **category**: Aircraft wake category
  - **distance_nm**: Calculated distance from feeder

  Data is refreshed every 2 seconds.
api:
  file: openapi (1).json
  operationId: get_aircraft_api_v1_aircraft_get
hidden: false
---