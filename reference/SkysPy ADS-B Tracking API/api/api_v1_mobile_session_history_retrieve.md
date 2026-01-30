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
  file: >-
    privatetmpclaude-Users-maxwatermolen-source-skyspycfddf7bd-246f-4143-b242-257ab7f89c1ascratchpadopenapi_fixed.yaml
  operationId: api_v1_mobile_session_history_retrieve
hidden: false
---