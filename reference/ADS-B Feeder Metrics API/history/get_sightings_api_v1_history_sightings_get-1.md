---
title: Query Aircraft Sightings
excerpt: |-
  Query historical aircraft position reports (sightings).

  Each sighting represents a single position report from an aircraft,
  stored every 10 seconds while the aircraft is in range.

  Filters:
  - **icao_hex**: Filter by aircraft ICAO address
  - **callsign**: Filter by callsign (partial match)
  - **military_only**: Only military aircraft
  - **hours**: Time range to query
  - **min_altitude/max_altitude**: Altitude range filter

  Results are returned newest first, limited to prevent large responses.
api:
  file: openapi.json
  operationId: get_sightings_api_v1_history_sightings_get
hidden: false
---