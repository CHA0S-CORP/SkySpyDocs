---
title: /api/v1/mobile/threats/
excerpt: |-
  Get threats for a stored session position.

  Query params:
  - session_id: Session ID from previous position update
  - radius_nm: Optional threat radius (default 25nm)

  Returns:
  {
      "threats": [...],
      "position": {"lat": 34.05, "lon": -118.25},
      "timestamp": "2024-01-01T00:00:00Z"
  }
api:
  file: >-
    privatetmpclaude-Users-maxwatermolen-source-skyspycfddf7bd-246f-4143-b242-257ab7f89c1ascratchpadopenapi_fixed.yaml
  operationId: api_v1_mobile_threats_retrieve
hidden: false
---