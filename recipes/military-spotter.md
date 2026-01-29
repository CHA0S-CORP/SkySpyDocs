---
title: Military Aircraft Spotter
excerpt: Get notified when military aircraft are in your area.
hidden: false
recipe:
  color: '#6366F1'
  icon: 🎖️
difficulty: beginner
tags: [notifications, military, alerts, built-in]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Push notifications configured (optional, for mobile alerts)
- Basic understanding of SkySpy's alert rules API

## What You'll Build

A built-in alert rule that notifies you whenever military aircraft enter your receiver's coverage area. SkySpy automatically detects military aircraft through multiple methods:

- **ICAO hex ranges** - Military aircraft use reserved address ranges
- **ADS-B category flags** - Category A1-A7 with military designations
- **Callsign patterns** - Known military callsign prefixes (RCH, EVAC, etc.)
- **Aircraft type database** - Known military aircraft types

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
    "fmt"
    "net/http"
    "os"
)

func main() {
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

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
    resp, err := http.Post(skyspyURL+"/api/alerts/rules", "application/json", bytes.NewReader(body))
    if err != nil {
        fmt.Printf("Error creating rule: %v\n", err)
        os.Exit(1)
    }
    defer resp.Body.Close()

    if resp.StatusCode == 201 {
        fmt.Println("Military spotter rule created successfully!")
    } else {
        fmt.Printf("Failed to create rule: %s\n", resp.Status)
    }
}
```

```python Python
import os
import requests

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

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

response = requests.post(f"{SKYSPY_URL}/api/alerts/rules", json=rule)

if response.status_code == 201:
    print("Military spotter rule created successfully!")
    print(f"Rule ID: {response.json().get('id')}")
else:
    print(f"Failed to create rule: {response.status_code}")
    print(response.text)
```

```javascript JavaScript
const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';

fetch(`${SKYSPY_URL}/api/alerts/rules`, {
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
})
.then(res => res.json())
.then(data => {
  console.log('Military spotter rule created successfully!');
  console.log('Rule ID:', data.id);
})
.catch(err => console.error('Failed to create rule:', err));
```

```json Response Example
{
  "id": 1,
  "name": "Military Aircraft",
  "enabled": true,
  "priority": "high",
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "military", "operator": "eq", "value": true}
    ]
  },
  "notification_enabled": true,
  "created_at": "2024-01-15T12:00:00Z"
}
```

## Create Military Alert Rule

<!-- shell@1-14 -->
<!-- go@1-35 -->
<!-- python@1-23 -->
<!-- javascript@1-23 -->

Create an alert rule that triggers whenever a military aircraft is detected. The rule uses SkySpy's built-in military detection which combines multiple identification methods for high accuracy.

## Understanding Military Detection

SkySpy identifies military aircraft through several methods:

| Method | Description | Example |
|--------|-------------|---------|
| ICAO Range | Reserved military hex ranges | `AE0000-AFFFFF` (US Military) |
| Category | ADS-B aircraft category | Category A1 with mil flag |
| Callsign | Known military prefixes | `RCH`, `EVAC`, `NAVY` |
| Type | Military aircraft types | C-17, F-16, KC-135 |

### Common Military Callsign Prefixes

- **RCH** - Air Mobility Command (AMC)
- **EVAC** - Medical evacuation flights
- **NAVY** - US Navy aircraft
- **JAKE** - Marine Corps aircraft
- **DOOM** - B-52 bombers
- **IRON** - Tanker aircraft

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Priority Levels

| Priority | Use Case |
|----------|----------|
| `low` | Informational alerts |
| `medium` | Standard tracking |
| `high` | Important aircraft (recommended) |
| `critical` | Emergency situations |

### Additional Conditions

Combine with other conditions for more specific alerts:

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "military", "operator": "eq", "value": true},
      {"field": "alt", "operator": "lt", "value": 10000}
    ]
  }
}
```

This alerts only on low-flying military aircraft.

## Testing & Verification

1. **Create the rule** using any of the code examples above
2. **Verify in SkySpy UI** - Go to Alerts → Rules to see your new rule
3. **Wait for military aircraft** - Alerts appear in the Alerts tab when detected
4. **Check notifications** - If push notifications are configured, you'll receive mobile alerts

### List Existing Rules

```shell
curl http://localhost:5000/api/alerts/rules
```

### Delete a Rule

```shell
curl -X DELETE http://localhost:5000/api/alerts/rules/1
```

## Troubleshooting

### Rule Not Triggering

**Problem:** Military aircraft appear but no alerts are generated
**Solution:**
- Verify the rule is enabled: check `enabled: true` in the API response
- Check the Alerts tab in SkySpy for any suppressed alerts
- Ensure the aircraft is actually flagged as military (check the aircraft details)

### No Push Notifications

**Problem:** Alerts appear in SkySpy but not on your phone
**Solution:**
- Configure Apprise in SkySpy settings (Settings → Notifications)
- Test your Apprise configuration with a test notification
- Verify `notification_enabled: true` on the rule

### Too Many Alerts

**Problem:** Getting alerts for every military aircraft is overwhelming
**Solution:** Add filters to reduce alerts:

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "military", "operator": "eq", "value": true},
      {"field": "distance", "operator": "lt", "value": 25}
    ]
  },
  "cooldown": 3600
}
```

This limits alerts to aircraft within 25nm and adds a 1-hour cooldown per aircraft.

### False Positives

**Problem:** Non-military aircraft triggering alerts
**Solution:** SkySpy's military detection is heuristic-based. If you find false positives:
- Report the ICAO hex code to improve the database
- Use additional filters (aircraft type, callsign patterns)

## Related Recipes

- [Track Specific Aircraft](/docs/track-aircraft) - Monitor specific tail numbers
- [Emergency Alert Monitor](/docs/emergency-monitor) - Track emergency squawks
- [Discord Alert Bot](/docs/discord-alert-bot) - Send alerts to Discord
- [Telegram Bot](/docs/telegram-bot) - Send alerts to Telegram
