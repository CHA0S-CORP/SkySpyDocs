---
title: Get Weather Cache Statistics
excerpt: |-
  Get comprehensive statistics about all weather caching.

  Includes both METAR (Redis) and PIREP (database) cache statistics.

  Returns:
  - METAR cache hits, misses, hit rate
  - PIREP storage counts and query stats
  - API request metrics
  - Last fetch/store timestamps
api:
  file: openapi.json
  operationId: get_weather_cache_stats_api_v1_aviation_weather_stats_get
hidden: false
---