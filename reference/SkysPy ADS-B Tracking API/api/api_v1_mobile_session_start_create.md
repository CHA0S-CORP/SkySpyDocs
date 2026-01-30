---
title: /api/v1/mobile/session/start/
excerpt: |-
  Start a new mobile tracking session.

  Request body:
  {
      "persistent": true  // Whether to save encounter history
  }

  Returns:
  {
      "session_id": "uuid",
      "persistent": true,
      "started_at": "2024-01-01T00:00:00Z"
  }
api:
  file: >-
    privatetmpclaude-Users-maxwatermolen-source-skyspycfddf7bd-246f-4143-b242-257ab7f89c1ascratchpadopenapi_fixed.yaml
  operationId: api_v1_mobile_session_start_create
hidden: false
---