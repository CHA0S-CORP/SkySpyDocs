---
title: Get ACARS Statistics
excerpt: |-
  Get comprehensive ACARS service and database statistics with optional filters.

  **Filters:**
  - **hours**: Time range for statistics (1-168 hours)
  - **source**: Filter by source (acars, vdlm2)
  - **label**: Filter by ACARS label code(s), comma-separated
  - **icao_hex**: Filter by aircraft ICAO hex
  - **callsign**: Filter by callsign (partial match)

  Includes:
  - Total message counts (all time, last hour, filtered range)
  - Breakdown by source (ACARS vs VDL2)
  - Top 10 most common message labels
  - Top 10 aircraft by message count
  - Hourly message distribution
  - Real-time receiver service statistics
api:
  file: openapi.json
  operationId: acars_statistics_api_v1_acars_stats_get
hidden: false
---