---
title: Upload Audio Transmission
excerpt: |-
  Upload an audio transmission from rtl-airband.

  The audio file will be:
  1. Saved to local storage
  2. Uploaded to S3 (if enabled)
  3. Queued for transcription (if enabled)

  Supported formats: MP3, WAV, OGG, FLAC

  **Note**: This endpoint is typically called by rtl-airband or a relay service,
  not directly by end users.
api:
  file: openapi.json
  operationId: upload_audio_api_v1_audio_upload_post
hidden: false
---