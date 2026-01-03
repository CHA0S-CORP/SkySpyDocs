---
title: Get Aircraft Photo URLs
excerpt: |-
  Get photo URLs for an aircraft.

  Returns:
  - **photo_url**: Full-size photo URL
  - **thumbnail_url**: Thumbnail image URL
  - **photographer**: Photo credit
  - **source**: Photo source (typically planespotters.net)

  Photos are sourced from Planespotters.net API.

  **Note**: Use `/{icao_hex}/photo/download` to proxy/download the actual image.
api:
  file: openapi.json
  operationId: get_aircraft_photo_api_v1_aircraft__icao_hex__photo_get
hidden: false
---