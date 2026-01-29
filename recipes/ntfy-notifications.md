---
title: ntfy.sh Integration
description: Self-hosted push notifications with ntfy.sh.
hidden: false
recipe:
  color: '#57A64A'
  icon: 🔔
---
```shell Shell
curl -d "Military aircraft RCH419 detected at 35,000 ft" \
  -H "Title: Aircraft Alert" \
  -H "Priority: high" \
  -H "Tags: airplane,warning" \
  https://ntfy.sh/skyspy-alerts
```

```go Go
package main

import (
	"bytes"
	"net/http"
	"os"
)

func sendNtfy(topic, title, message, priority string) {
	ntfyURL := os.Getenv("NTFY_URL")
	if ntfyURL == "" {
		ntfyURL = "https://ntfy.sh"
	}

	req, _ := http.NewRequest("POST", ntfyURL+"/"+topic, bytes.NewBufferString(message))
	req.Header.Set("Title", title)
	req.Header.Set("Priority", priority)
	req.Header.Set("Tags", "airplane,warning")

	http.DefaultClient.Do(req)
}

func main() {
	sendNtfy("skyspy-alerts", "Aircraft Alert", "Military aircraft RCH419 at 35,000 ft", "high")
}
```

```python Python
import os
import requests

NTFY_URL = os.getenv("NTFY_URL", "https://ntfy.sh")

def send_ntfy(topic, title, message, priority="default"):
    requests.post(
        f"{NTFY_URL}/{topic}",
        data=message,
        headers={
            "Title": title,
            "Priority": priority,
            "Tags": "airplane,warning"
        }
    )

# Example usage
send_ntfy("skyspy-alerts", "Aircraft Alert", "Military aircraft RCH419 at 35,000 ft", "high")
```

```javascript JavaScript
const NTFY_URL = process.env.NTFY_URL || 'https://ntfy.sh';

async function sendNtfy(topic, title, message, priority = 'default') {
    await fetch(`${NTFY_URL}/${topic}`, {
        method: 'POST',
        body: message,
        headers: {
            'Title': title,
            'Priority': priority,
            'Tags': 'airplane,warning',
        },
    });
}

// Example usage
sendNtfy('skyspy-alerts', 'Aircraft Alert', 'Military aircraft RCH419 at 35,000 ft', 'high');
```

```json Response Example
{"id": "abc123", "event": "message", "topic": "skyspy-alerts"}
```

# Configure ntfy Endpoint

<!-- shell@5 -->
<!-- go@1-13 -->
<!-- python@1-4 -->
<!-- javascript@1 -->

Use the public ntfy.sh server or self-host your own. Choose a unique topic name for your alerts.

# Build Notification

<!-- shell@1-4 -->
<!-- go@15-20 -->
<!-- python@6-14 -->
<!-- javascript@3-12 -->

Set headers for title, priority (min, low, default, high, urgent), and tags for emoji icons.

# Send to Topic

<!-- go@22-23 -->
<!-- python@16-17 -->
<!-- javascript@14-15 -->

POST the message to your topic. Subscribe on mobile via the ntfy app or any HTTP client.
