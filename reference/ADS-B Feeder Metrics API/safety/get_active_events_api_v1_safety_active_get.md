---
title: Get Active Safety Events
excerpt: |-
  Get all currently active safety events being tracked by the monitor.

  Active events include:
  - Emergency squawks (7500, 7600, 7700)
  - TCAS RAs and VS reversals
  - Proximity conflicts
  - Extreme vertical speeds

  Events are automatically removed after 5 minutes of inactivity.
api:
  file: openapi (1).json
  operationId: get_active_events_api_v1_safety_active_get
hidden: false
---