---
title: Pushover Notifications
description: Send mobile push notifications via Pushover.
hidden: false
recipe:
  color: '#249DF1'
  icon: 📲
---
```shell Shell
curl -s --form-string "token=$PUSHOVER_APP_TOKEN" \
  --form-string "user=$PUSHOVER_USER_KEY" \
  --form-string "title=Military Aircraft Alert" \
  --form-string "message=RCH419 (C-17) at 35,000 ft" \
  --form-string "priority=1" \
  --form-string "sound=siren" \
  https://api.pushover.net/1/messages.json
```

```go Go
package main

import (
	"net/http"
	"net/url"
	"os"
)

func sendPushover(title, message string, priority int) {
	appToken := os.Getenv("PUSHOVER_APP_TOKEN")
	userKey := os.Getenv("PUSHOVER_USER_KEY")

	http.PostForm("https://api.pushover.net/1/messages.json", url.Values{
		"token":    {appToken},
		"user":     {userKey},
		"title":    {title},
		"message":  {message},
		"priority": {string(rune('0' + priority))},
		"sound":    {"siren"},
	})
}

func main() {
	sendPushover("Military Aircraft Alert", "RCH419 (C-17) at 35,000 ft", 1)
}
```

```python Python
import os
import requests

PUSHOVER_APP_TOKEN = os.getenv("PUSHOVER_APP_TOKEN")
PUSHOVER_USER_KEY = os.getenv("PUSHOVER_USER_KEY")

def send_pushover(title, message, priority=0):
    requests.post("https://api.pushover.net/1/messages.json", data={
        "token": PUSHOVER_APP_TOKEN,
        "user": PUSHOVER_USER_KEY,
        "title": title,
        "message": message,
        "priority": priority,
        "sound": "siren"
    })

# Example usage
send_pushover("Military Aircraft Alert", "RCH419 (C-17) at 35,000 ft", priority=1)
```

```javascript JavaScript
const PUSHOVER_APP_TOKEN = process.env.PUSHOVER_APP_TOKEN;
const PUSHOVER_USER_KEY = process.env.PUSHOVER_USER_KEY;

async function sendPushover(title, message, priority = 0) {
    const formData = new URLSearchParams({
        token: PUSHOVER_APP_TOKEN,
        user: PUSHOVER_USER_KEY,
        title,
        message,
        priority: String(priority),
        sound: 'siren',
    });

    await fetch('https://api.pushover.net/1/messages.json', {
        method: 'POST',
        body: formData,
    });
}

// Example usage
sendPushover('Military Aircraft Alert', 'RCH419 (C-17) at 35,000 ft', 1);
```

```json Response Example
{"status": 1, "request": "abc123-def456"}
```

# Configure Pushover Credentials

<!-- shell@1-2 -->
<!-- go@1-12 -->
<!-- python@1-5 -->
<!-- javascript@1-2 -->

Get your User Key from [pushover.net](https://pushover.net) and create an application to get an App Token.

# Build Notification

<!-- shell@3-6 -->
<!-- go@14-21 -->
<!-- python@7-15 -->
<!-- javascript@4-13 -->

Set the title, message, priority level (0=normal, 1=high, 2=emergency), and notification sound.

# Send Push Notification

<!-- shell@7 -->
<!-- go@23-24 -->
<!-- python@17-18 -->
<!-- javascript@15-19 -->

POST to the Pushover API. High priority notifications bypass Do Not Disturb on mobile devices.
