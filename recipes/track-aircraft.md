---
title: Track Specific Aircraft
excerpt: Monitor specific aircraft by tail number, ICAO hex code, or callsign.
hidden: true
recipe:
  color: '#018FF4'
  icon: ✈️
---
```shell Shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Track N12345",
    "enabled": true,
    "priority": "high",
    "conditions": {
      "operator": "AND",
      "conditions": [
        { "field": "icao", "operator": "eq", "value": "A12345" }
      ]
    },
    "notification_enabled": true
  }'
```

```go Go
package main

import (
    "bytes"
    "encoding/json"
    "net/http"
)

func main() {
    rule := map[string]interface{}{
        "name":    "Track N12345",
        "enabled": true,
        "priority": "high",
        "conditions": map[string]interface{}{
            "operator": "AND",
            "conditions": []map[string]interface{}{
                {"field": "icao", "operator": "eq", "value": "A12345"},
            },
        },
        "notification_enabled": true,
    }
    body, _ := json.Marshal(rule)
    http.Post("http://localhost:5000/api/alerts/rules", "application/json", bytes.NewReader(body))
}
```

```python Python
import requests

rule = {
    "name": "Track N12345",
    "enabled": True,
    "priority": "high",
    "conditions": {
        "operator": "AND",
        "conditions": [
            {"field": "icao", "operator": "eq", "value": "A12345"}
        ]
    },
    "notification_enabled": True
}
requests.post("http://localhost:5000/api/alerts/rules", json=rule)
```

```javascript JavaScript
fetch('http://localhost:5000/api/alerts/rules', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    name: 'Track N12345',
    enabled: true,
    priority: 'high',
    conditions: {
      operator: 'AND',
      conditions: [
        { field: 'icao', operator: 'eq', value: 'A12345' }
      ]
    },
    notification_enabled: true
  })
});
```

```json Response Example
{"id": 1, "name": "Track N12345", "enabled": true, "created_at": "2024-01-15T12:00:00Z"}
```

# Create Alert Rule

<!-- shell@1-14 -->
<!-- go@1-18 -->
<!-- python@1-15 -->
<!-- javascript@1-17 -->

Create an alert rule to track a specific aircraft by its ICAO hex code. The rule will trigger whenever this aircraft appears in your receiver's coverage area.

# Set ICAO Condition

<!-- shell@7-11 -->
<!-- go@10-13 -->
<!-- python@6-11 -->
<!-- javascript@8-13 -->

The `icao` field matches the aircraft's 24-bit ICAO hex address (e.g., A12345). This is a permanent identifier - unlike callsigns which can change between flights.

# Enable Notifications

<!-- shell@12-13 -->
<!-- go@14-15 -->
<!-- python@12-14 -->
<!-- javascript@14-16 -->

Set `notification_enabled` to true to receive push notifications when your tracked aircraft is detected. Configure Apprise in SkySpy settings for push alerts.
