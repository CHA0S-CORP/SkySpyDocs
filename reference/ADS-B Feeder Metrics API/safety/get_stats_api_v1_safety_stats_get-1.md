---
title: Get Safety Statistics
excerpt: |-
  Get comprehensive safety monitoring statistics with optional filters.

  **Filters:**
  - **event_type**: Filter by specific event type(s) - comma-separated
  - **severity**: Filter by severity level(s) - comma-separated
  - **icao_hex**: Filter by aircraft ICAO hex

  **Returns:**
  - Current monitoring status and thresholds
  - Event counts by type and severity
  - Per-type severity breakdown
  - Hourly event distribution
  - Top aircraft involved in events
  - Recent events summary
  - Monitor internal state
api:
  file: openapi.json
  operationId: get_stats_api_v1_safety_stats_get
hidden: false
---