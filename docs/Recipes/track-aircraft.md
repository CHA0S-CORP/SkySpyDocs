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

# step1

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->

Set up alerts to track specific aircraft whenever they appear in your receiver's coverage area. You can track by ICAO hex code (permanent identifier) or callsign (can change between flights).

# step2

To track multiple aircraft, use OR conditions:

```shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Friends Fleet",
    "conditions": {
      "operator": "OR",
      "conditions": [
        { "field": "icao", "operator": "eq", "value": "A12345" },
        { "field": "icao", "operator": "eq", "value": "A67890" }
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

To track by callsign prefix (e.g., all United flights):

```shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "United Airlines",
    "conditions": {
      "operator": "AND",
      "conditions": [
        { "field": "callsign", "operator": "startswith", "value": "UAL" }
      ]
    }
  }'
```

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->
