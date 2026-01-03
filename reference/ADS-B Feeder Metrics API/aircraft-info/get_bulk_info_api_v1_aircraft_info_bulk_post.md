---
title: Bulk Aircraft Info Lookup
excerpt: |-
  Get aircraft information for multiple aircraft at once.

  **Important**: This endpoint only returns **cached** data. It does not
  fetch from external sources to avoid rate limiting.

  Use the single aircraft endpoint first to populate the cache for
  aircraft you're interested in.

  Request body: Array of ICAO hex codes (maximum 100)
api:
  file: openapi (1).json
  operationId: get_bulk_info_api_v1_aircraft_info_bulk_post
hidden: false
---