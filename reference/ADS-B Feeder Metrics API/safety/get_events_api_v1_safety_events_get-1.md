---
title: Query Safety Events
excerpt: |-
  Query recorded safety events.

  **Event Types:**
  - **tcas_ra**: TCAS Resolution Advisory - evasive action recommended
  - **tcas_ta**: TCAS Traffic Advisory - traffic alert
  - **extreme_vs**: Extreme vertical speed change detected
  - **proximity**: Two aircraft in close proximity

  **Severity Levels:**
  - **info**: Informational event
  - **warning**: Potential concern
  - **critical**: Immediate attention required

  Events are stored when detected by the safety monitor during 
  aircraft tracking.
api:
  file: openapi.json
  operationId: get_events_api_v1_safety_events_get
hidden: false
---