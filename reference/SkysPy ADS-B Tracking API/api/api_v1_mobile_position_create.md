---
title: /api/v1/mobile/position/
excerpt: |-
  Update mobile device position and return nearby threats.

  Request body:
  {
      "lat": 34.05,
      "lon": -118.25,
      "session_id": "optional-session-id",
      "heading": 180,  // optional device heading
      "radius_nm": 25  // optional threat radius
  }

  Returns:
  {
      "session_id": "generated-or-provided-session-id",
      "position": {"lat": 34.05, "lon": -118.25},
      "threats": [...],
      "timestamp": "2024-01-01T00:00:00Z"
  }
api:
  file: >-
    privatetmpclaude-Users-maxwatermolen-source-skyspycfddf7bd-246f-4143-b242-257ab7f89c1ascratchpadopenapi_fixed.yaml
  operationId: api_v1_mobile_position_create
hidden: false
---