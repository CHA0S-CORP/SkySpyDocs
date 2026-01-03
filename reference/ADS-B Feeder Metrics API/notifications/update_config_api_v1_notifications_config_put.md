---
title: Update Notification Configuration
excerpt: |-
  Update the notification configuration.

  **Apprise URL Format:**
  URLs for notification services in Apprise format, separated by commas.

  Examples:
  - Pushover: `pover://user_key@app_token`
  - Telegram: `tgram://bot_token/chat_id`
  - Discord: `discord://webhook_id/webhook_token`
  - Slack: `slack://token_a/token_b/token_c`
  - Email: `mailto://user:pass@gmail.com`

  See https://github.com/caronc/apprise for full documentation.

  **Cooldown:**
  Minimum seconds between notifications to prevent flooding.
  Recommended: 300 (5 minutes) for normal use.
api:
  file: openapi (1).json
  operationId: update_config_api_v1_notifications_config_put
hidden: false
---