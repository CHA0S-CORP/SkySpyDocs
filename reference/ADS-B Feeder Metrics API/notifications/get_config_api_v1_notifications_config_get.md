---
title: Get Notification Configuration
excerpt: |-
  Get the current notification configuration.

  Returns:
  - **enabled**: Whether notifications are active
  - **apprise_urls**: Configured notification service URLs (masked)
  - **cooldown_seconds**: Minimum time between notifications
  - **server_count**: Number of configured notification servers

  Apprise URLs are masked for security.
api:
  file: openapi (1).json
  operationId: get_config_api_v1_notifications_config_get
hidden: false
---