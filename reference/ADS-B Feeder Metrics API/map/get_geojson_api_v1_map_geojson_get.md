---
title: Get Aircraft as GeoJSON
excerpt: |-
  Get all aircraft positions as a GeoJSON FeatureCollection.

  Each aircraft is represented as a Point feature with properties:
  - **icao**: ICAO hex address
  - **callsign**: Flight callsign
  - **altitude**: Barometric altitude in feet
  - **speed**: Ground speed in knots
  - **track**: Ground track in degrees
  - **vrate**: Vertical rate in ft/min
  - **squawk**: Transponder code
  - **type**: Aircraft type code
  - **category**: Aircraft category
  - **military**: Military flag
  - **emergency**: Emergency flag
  - **distance_nm**: Distance from feeder

  Perfect for use with mapping libraries like Leaflet, MapLibre, or OpenLayers.
api:
  file: openapi (1).json
  operationId: get_geojson_api_v1_map_geojson_get
hidden: false
---