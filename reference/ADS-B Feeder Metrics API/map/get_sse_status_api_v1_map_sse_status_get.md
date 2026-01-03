---
title: Get SSE Status
excerpt: |-
  Get the current status of the SSE service.

  Returns:
  - **mode**: Operating mode (memory or redis)
  - **subscribers**: Current subscriber count
  - **tracked_aircraft**: Aircraft in state cache
  - **last_publish**: Time of last broadcast
  - **history**: Event history buffer info
api:
  file: openapi (1).json
  operationId: get_sse_status_api_v1_map_sse_status_get
hidden: false
---