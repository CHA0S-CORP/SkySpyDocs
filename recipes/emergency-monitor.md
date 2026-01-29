---
title: Emergency Alert Monitor
excerpt: Monitor emergency squawks (7700, 7600, 7500) in your area.
hidden: true
recipe:
  color: '#EF4444'
  icon: 🚨
---
```shell Shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Emergency Squawk Monitor",
    "enabled": true,
    "priority": "critical",
    "conditions": {
      "operator": "OR",
      "conditions": [
        { "field": "squawk", "operator": "eq", "value": "7700" },
        { "field": "squawk", "operator": "eq", "value": "7600" },
        { "field": "squawk", "operator": "eq", "value": "7500" }
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
        "name":     "Emergency Squawk Monitor",
        "enabled":  true,
        "priority": "critical",
        "conditions": map[string]interface{}{
            "operator": "OR",
            "conditions": []map[string]interface{}{
                {"field": "squawk", "operator": "eq", "value": "7700"},
                {"field": "squawk", "operator": "eq", "value": "7600"},
                {"field": "squawk", "operator": "eq", "value": "7500"},
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
    "name": "Emergency Squawk Monitor",
    "enabled": True,
    "priority": "critical",
    "conditions": {
        "operator": "OR",
        "conditions": [
            {"field": "squawk", "operator": "eq", "value": "7700"},
            {"field": "squawk", "operator": "eq", "value": "7600"},
            {"field": "squawk", "operator": "eq", "value": "7500"}
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
    name: 'Emergency Squawk Monitor',
    enabled: true,
    priority: 'critical',
    conditions: {
      operator: 'OR',
      conditions: [
        { field: 'squawk', operator: 'eq', value: '7700' },
        { field: 'squawk', operator: 'eq', value: '7600' },
        { field: 'squawk', operator: 'eq', value: '7500' }
      ]
    },
    notification_enabled: true
  })
});
```

```json Response Example
{"id": 1, "name": "Emergency Squawk Monitor", "enabled": true, "priority": "critical"}
```

# Create Emergency Monitor Rule

<!-- shell@1-16 -->
<!-- go@1-19 -->
<!-- python@1-14 -->
<!-- javascript@1-18 -->

Set up real-time monitoring for aircraft broadcasting emergency squawk codes. Emergency codes indicate serious aviation emergencies:

- **7700** - General Emergency (Mayday)
- **7600** - Radio Failure (NORDO)
- **7500** - Hijack

# Configure Squawk Conditions

<!-- shell@7-14 -->
<!-- go@10-15 -->
<!-- python@6-12 -->
<!-- javascript@6-13 -->

The rule uses an OR operator to match any of the three emergency squawk codes. When any aircraft broadcasts 7700, 7600, or 7500, you'll receive an alert.

# Enable Notifications

<!-- shell@15-16 -->
<!-- go@16-18 -->
<!-- python@13-14 -->
<!-- javascript@14-17 -->

Set `notification_enabled` to true to receive push notifications when emergency squawks are detected. Make sure Apprise is configured in your SkySpy settings.
