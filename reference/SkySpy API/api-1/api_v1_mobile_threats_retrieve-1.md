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
  file: skyspy-api.yaml
  operationId: api_v1_mobile_threats_retrieve
hidden: false
---