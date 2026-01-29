---
title: Military Aircraft Spotter
description: Get notified when military aircraft are in your area.
hidden: false
recipe:
  color: '#6366F1'
  icon: 🎖️
---
```shell Shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Military Aircraft",
    "enabled": true,
    "priority": "high",
    "conditions": {
      "operator": "AND",
      "conditions": [
        { "field": "military", "operator": "eq", "value": true }
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
        "name":     "Military Aircraft",
        "enabled":  true,
        "priority": "high",
        "conditions": map[string]interface{}{
            "operator": "AND",
            "conditions": []map[string]interface{}{
                {"field": "military", "operator": "eq", "value": true},
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
    "name": "Military Aircraft",
    "enabled": True,
    "priority": "high",
    "conditions": {
        "operator": "AND",
        "conditions": [
            {"field": "military", "operator": "eq", "value": True}
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
    name: 'Military Aircraft',
    enabled: true,
    priority: 'high',
    conditions: {
      operator: 'AND',
      conditions: [
        { field: 'military', operator: 'eq', value: true }
      ]
    },
    notification_enabled: true
  })
});
```

```json Response Example
{"id": 1, "name": "Military Aircraft", "enabled": true, "priority": "high"}
```

# Create Military Alert Rule

<!-- shell@1-14 -->
<!-- go@1-18 -->
<!-- python@1-15 -->
<!-- javascript@1-17 -->

Create an alert rule to get notified when military aircraft enter your coverage area. SkySpy detects military aircraft through ADS-B category flags, ICAO hex ranges, and callsign patterns.

# Set Military Condition

<!-- shell@7-11 -->
<!-- go@10-13 -->
<!-- python@6-11 -->
<!-- javascript@8-13 -->

The `military` field is a boolean that SkySpy automatically sets based on aircraft identification. Set it to `true` to match all military aircraft.

# Enable Push Notifications

<!-- shell@12-13 -->
<!-- go@14-15 -->
<!-- python@12-14 -->
<!-- javascript@14-16 -->

With `notification_enabled` set to true, you'll receive alerts on your phone or other configured notification channels when military aircraft are detected.