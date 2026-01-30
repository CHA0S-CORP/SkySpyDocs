---
title: /api/v1/mobile/session/history/
excerpt: |-
  Get encounter history for a persistent session.

  Query params:
  - session_id: Session ID

  Returns:
  {
      "session_id": "uuid",
      "encounters": [...],
      "started_at": "2024-01-01T00:00:00Z"
  }
api:
  file: skyspy-api.yaml
  operationId: api_v1_mobile_session_history_retrieve
hidden: false
---