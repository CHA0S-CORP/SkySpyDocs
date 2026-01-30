---
title: /api/v1/mobile/session/end/
excerpt: |-
  End a mobile tracking session.

  Request body:
  {
      "session_id": "uuid"
  }

  Returns:
  {
      "session_id": "uuid",
      "encounters": [...],
      "duration_seconds": 1234
  }
api:
  file: skyspy-api.yaml
  operationId: api_v1_mobile_session_end_create
hidden: false
---