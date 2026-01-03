---
title: Get ACARS Messages
excerpt: |-
  Query ACARS and VDL2 messages from the database.

  Filter by:
  - **icao_hex**: Aircraft ICAO 24-bit address
  - **callsign**: Flight callsign (partial match)
  - **label**: ACARS message label code
  - **source**: Message source (acars or vdlm2)
  - **hours**: Time range to query (1-168 hours)

  Common ACARS labels:
  - H1: Flight plan / departure clearance
  - SA: Position report
  - B6: Departure message
  - QA: Weather request
  - _d: Air-ground voice

  Messages are returned newest first.
api:
  file: openapi.json
  operationId: list_acars_messages_api_v1_acars_messages_get
hidden: false
---