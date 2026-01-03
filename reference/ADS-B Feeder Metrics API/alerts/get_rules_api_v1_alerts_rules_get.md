---
title: List Alert Rules
excerpt: |-
  Get all configured alert rules.

  Alert rules define conditions that trigger notifications when aircraft
  match specified criteria.

  Rule types:
  - **icao**: Match specific ICAO hex address
  - **callsign**: Match flight callsign
  - **squawk**: Match transponder code
  - **altitude**: Compare altitude (lt, gt, le, ge)
  - **distance**: Compare distance from feeder
  - **type**: Match aircraft type code
  - **military**: Match military aircraft flag

  Complex rules support AND/OR logic with multiple condition groups.
api:
  file: openapi (1).json
  operationId: get_rules_api_v1_alerts_rules_get
hidden: false
---