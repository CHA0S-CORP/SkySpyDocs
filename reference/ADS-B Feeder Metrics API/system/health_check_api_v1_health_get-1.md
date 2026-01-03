---
title: Health Check
excerpt: |-
  Check the health status of all system components.

  Checks:
  - **database**: PostgreSQL connection and latency
  - **ultrafeeder**: ADS-B data source availability
  - **redis**: Redis cache connection (if configured)
  - **sse**: Server-Sent Events service status
  - **acars**: ACARS receiver service (if enabled)

  Returns overall status:
  - `healthy`: All services operational
  - `degraded`: Some services have issues
  - `unhealthy`: Critical services unavailable
api:
  file: openapi.json
  operationId: health_check_api_v1_health_get
hidden: false
---