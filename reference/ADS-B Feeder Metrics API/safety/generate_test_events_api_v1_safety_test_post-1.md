---
title: Generate Test Safety Events
excerpt: >-
  Generate test events for all safety event types.


  Creates one test event for each type:

  - Emergency squawk (7700)

  - TCAS RA

  - VS reversal

  - Extreme vertical speed

  - Proximity conflict


  Test events are marked with is_test=True and will appear in the active events
  list.

  They will expire normally after 5 minutes or can be cleared manually.
api:
  file: openapi.json
  operationId: generate_test_events_api_v1_safety_test_post
hidden: false
---