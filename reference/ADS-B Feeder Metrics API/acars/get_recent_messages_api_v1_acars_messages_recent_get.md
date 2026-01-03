---
title: Get Recent Messages (Fast)
excerpt: |-
  Get recent ACARS messages from the in-memory buffer.

  This endpoint is faster than the database query but limited to 
  approximately the last 100 messages received.

  Use this for real-time displays where speed is important.
api:
  file: openapi (1).json
  operationId: get_recent_messages_api_v1_acars_messages_recent_get
hidden: false
---