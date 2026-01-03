---
title: Get Cache Statistics
excerpt: |-
  Get statistics about the aircraft information cache.

  Returns:
  - **total_cached**: Total aircraft in the cache
  - **failed_lookups**: Number of aircraft where lookup failed
  - **with_photos**: Aircraft with cached photos
  - **cache_duration_hours**: How long successful lookups are cached
  - **retry_after_hours**: When failed lookups are retried
api:
  file: openapi (1).json
  operationId: get_cache_stats_api_v1_aircraft_info_cache_stats_get
hidden: false
---