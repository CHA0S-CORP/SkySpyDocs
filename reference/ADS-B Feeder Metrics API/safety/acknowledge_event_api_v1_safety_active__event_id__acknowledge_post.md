---
title: Acknowledge Safety Event
excerpt: >-
  Acknowledge a safety event by its ID.


  Acknowledged events are still tracked but won't trigger alarms

  in the UI. The event will be cleared when it naturally expires.


  Accepts either:

  - String event ID (e.g., "proximity_conflict:A1801C:AC940A") for active events

  - Numeric database ID (e.g., "123") which will find the matching active event


  Note: Only currently active events (within the last 5 minutes) can be
  acknowledged.

  Historical events from the database cannot be acknowledged after they expire.
api:
  file: openapi (1).json
  operationId: acknowledge_event_api_v1_safety_active__event_id__acknowledge_post
hidden: false
---