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

# step1

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->

Set up real-time monitoring for aircraft broadcasting emergency squawk codes. Emergency codes indicate serious aviation emergencies:

- **7700** - General Emergency (Mayday)
- **7600** - Radio Failure (NORDO)
- **7500** - Hijack

# step2

Enable built-in safety monitoring in your `.env` file:

```shell
SAFETY_MONITORING_ENABLED=true
```

When enabled, emergency squawks trigger dashboard alerts and push notifications automatically.

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->

# step3

> ❗️ Important
>
> **Do not interfere with emergency operations.** Pilots occasionally squawk emergency codes accidentally or during training. If you observe a real emergency, do not attempt to contact the aircraft — ATC and first responders are already handling it.

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->
