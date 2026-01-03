---
title: Get METAR for Station
excerpt: |-
  Get current METAR (Meteorological Aerodrome Report) for a specific airport.

  METAR provides current weather conditions including:
  - Wind direction and speed
  - Visibility
  - Cloud coverage and ceiling
  - Temperature and dewpoint
  - Altimeter setting
  - Flight category (VFR/MVFR/IFR/LIFR)

  **Data Source:** Uses Redis cache when available (5-minute TTL).
  Falls back to Aviation Weather Center API.

  Station IDs are 4-letter ICAO codes (e.g., KSEA, KJFK, EGLL).
api:
  file: openapi.json
  operationId: get_metar_api_v1_aviation_metar__station__get
hidden: false
---