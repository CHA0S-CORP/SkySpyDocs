---
title: Gotify Notifications
description: Self-hosted push notifications with Gotify.
hidden: false
recipe:
  color: '#2196F3'
  icon: 📬
---
```shell Shell
curl -X POST "$GOTIFY_URL/message?token=$GOTIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title": "Aircraft Alert", "message": "Military: RCH419 at 35,000 ft", "priority": 8}'
```

```go Go
package main

import (
	"bytes"
	"encoding/json"
	"net/http"
	"os"
)

type GotifyMessage struct {
	Title    string `json:"title"`
	Message  string `json:"message"`
	Priority int    `json:"priority"`
}

func sendGotify(title, message string, priority int) {
	gotifyURL := os.Getenv("GOTIFY_URL")
	token := os.Getenv("GOTIFY_TOKEN")

	msg := GotifyMessage{Title: title, Message: message, Priority: priority}
	body, _ := json.Marshal(msg)

	http.Post(gotifyURL+"/message?token="+token, "application/json", bytes.NewReader(body))
}

func main() {
	sendGotify("Aircraft Alert", "Military: RCH419 at 35,000 ft", 8)
}
```

```python Python
import os
import requests

GOTIFY_URL = os.getenv("GOTIFY_URL")
GOTIFY_TOKEN = os.getenv("GOTIFY_TOKEN")

def send_gotify(title, message, priority=5):
    requests.post(
        f"{GOTIFY_URL}/message",
        params={"token": GOTIFY_TOKEN},
        json={"title": title, "message": message, "priority": priority}
    )

# Example usage
send_gotify("Aircraft Alert", "Military: RCH419 at 35,000 ft", priority=8)
```

```javascript JavaScript
const GOTIFY_URL = process.env.GOTIFY_URL;
const GOTIFY_TOKEN = process.env.GOTIFY_TOKEN;

async function sendGotify(title, message, priority = 5) {
    await fetch(`${GOTIFY_URL}/message?token=${GOTIFY_TOKEN}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ title, message, priority }),
    });
}

// Example usage
sendGotify('Aircraft Alert', 'Military: RCH419 at 35,000 ft', 8);
```

```json Response Example
{"id": 42, "appid": 1, "message": "Military: RCH419 at 35,000 ft", "priority": 8}
```

# Configure Gotify Server

<!-- shell@1 -->
<!-- go@1-9 -->
<!-- python@1-4 -->
<!-- javascript@1-2 -->

Deploy Gotify on your server and create an application token. Set GOTIFY_URL (e.g., https://gotify.example.com) and GOTIFY_TOKEN.

# Build Message

<!-- shell@2-3 -->
<!-- go@11-20 -->
<!-- python@6-12 -->
<!-- javascript@4-8 -->

Set title, message body, and priority (0-10). Higher priorities trigger more prominent notifications.

# Send Notification

<!-- go@22-23 -->
<!-- python@14 -->
<!-- javascript@10-11 -->

POST to the Gotify message endpoint. Install the Gotify mobile app to receive push notifications.
