---
title: Download Aircraft Photo
excerpt: >-
  Download/proxy the aircraft photo.


  This endpoint returns the photo directly, first checking the local cache,

  then falling back to fetching from the remote source.


  **Query Parameters:**

  - `thumbnail`: If true, returns the smaller thumbnail image (default: false)


  **Response**: The actual image data (JPEG/PNG) with appropriate Content-Type
  header.


  **Headers included:**

  - `X-Photo-Source`: Where the photo was served from (local or remote URL)

  - `X-Photo-Photographer`: Photo credit (if available)

  - `X-Photo-Cached`: "true" if served from local cache
api:
  file: openapi (1).json
  operationId: download_aircraft_photo_api_v1_aircraft__icao_hex__photo_download_get
hidden: false
---