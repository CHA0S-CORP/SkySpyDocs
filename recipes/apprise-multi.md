---
title: Apprise Multi-Notification
description: Send alerts to multiple services with Apprise.
hidden: false
recipe:
  color: '#FF6B6B'
  icon: 🔔
---
```shell Shell
apprise -t "Aircraft Alert" -b "Military: RCH419 at 35,000 ft" \
  "tgram://bot_token/chat_id" \
  "slack://token_a/token_b/token_c/#channel" \
  "discord://webhook_id/webhook_token"
```

```go Go
package main

import (
	"bytes"
	"encoding/json"
	"net/http"
	"os"
)

func main() {
	appriseURL := os.Getenv("APPRISE_URL") // e.g., http://localhost:8000/notify

	payload := map[string]interface{}{
		"title": "Aircraft Alert",
		"body":  "Military: RCH419 at 35,000 ft",
		"type":  "warning",
		"urls": []string{
			os.Getenv("TELEGRAM_APPRISE_URL"),
			os.Getenv("SLACK_APPRISE_URL"),
			os.Getenv("DISCORD_APPRISE_URL"),
		},
	}

	body, _ := json.Marshal(payload)
	http.Post(appriseURL, "application/json", bytes.NewReader(body))
}
```

```python Python
import apprise
import os

# Create Apprise instance
apobj = apprise.Apprise()

# Add notification services
apobj.add(os.getenv("TELEGRAM_APPRISE_URL"))  # tgram://bot_token/chat_id
apobj.add(os.getenv("SLACK_APPRISE_URL"))     # slack://token/#channel
apobj.add(os.getenv("DISCORD_APPRISE_URL"))   # discord://webhook_id/token

def send_alert(title, body, notify_type=apprise.NotifyType.WARNING):
    apobj.notify(
        title=title,
        body=body,
        notify_type=notify_type
    )

# Example usage
send_alert("Aircraft Alert", "Military: RCH419 at 35,000 ft")
```

```javascript JavaScript
const APPRISE_URL = process.env.APPRISE_URL;

const NOTIFICATION_URLS = [
    process.env.TELEGRAM_APPRISE_URL,
    process.env.SLACK_APPRISE_URL,
    process.env.DISCORD_APPRISE_URL,
];

async function sendAlert(title, body, type = 'warning') {
    await fetch(APPRISE_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            title,
            body,
            type,
            urls: NOTIFICATION_URLS,
        }),
    });
}

// Example usage
sendAlert('Aircraft Alert', 'Military: RCH419 at 35,000 ft');
```

```json Response Example
{"status": "ok", "notifications_sent": 3}
```

# Configure Services

<!-- shell@2-4 -->
<!-- go@14-19 -->
<!-- python@5-10 -->
<!-- javascript@3-7 -->

Add Apprise URLs for each notification service. Apprise supports 80+ services including Telegram, Slack, Discord, Email, and more.

# Create Alert Function

<!-- go@11-22 -->
<!-- python@12-17 -->
<!-- javascript@9-18 -->

Build a reusable function that sends to all configured services at once. Set notification type (info, success, warning, failure).

# Send to All Services

<!-- shell@1 -->
<!-- go@24-25 -->
<!-- python@19-20 -->
<!-- javascript@20-21 -->

One call sends to all configured services. Failed services don't block others from receiving the alert.
