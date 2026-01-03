---
title: Get Photo Cache Statistics
excerpt: |-
  Get statistics about the local aircraft photo cache.

  Returns:
  - **enabled**: Whether photo caching is enabled
  - **cache_dir**: Directory where photos are stored
  - **total_photos**: Number of full-size photos cached
  - **total_thumbnails**: Number of thumbnail images cached
  - **total_size_mb**: Total disk space used by cached photos
  - **unique_aircraft_seen**: Number of unique aircraft seen this session
api:
  file: openapi.json
  operationId: get_photo_cache_status_api_v1_photos_cache_get
hidden: false
---