---
title: Military Aircraft Spotter
description: Get notified when military aircraft are in your area.
hidden: true
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

# step1

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->

Set up alerts to track military aircraft in your receiver's coverage area. SkySpy detects military aircraft through ADS-B category flags, ICAO hex ranges, and known military callsign patterns.

# step2

To filter military aircraft within a specific distance (e.g., 25 nautical miles):

```shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Military Nearby",
    "priority": "high",
    "conditions": {
      "operator": "AND",
      "conditions": [
        { "field": "military", "operator": "eq", "value": true },
        { "field": "distance", "operator": "lt", "value": 25 }
      ]
    },
    "notification_enabled": true
  }'
```

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->

# step3

Common military aircraft types you may see:

- **F16, F15** - Fighter jets
- **C17, C130** - Transport aircraft
- **KC135** - Tanker
- **B52** - Bomber
- **TYPHOON, RAFALE, TORNADO** - NATO fighters

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->
