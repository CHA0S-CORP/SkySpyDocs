---
title: Emergency Alert Monitor
excerpt: Monitor emergency squawks (7700, 7600, 7500) in your area.
hidden: false
recipe:
  color: '#EF4444'
  icon: 🚨
difficulty: beginner
tags: [notifications, emergency, squawk, safety, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Push notifications configured (strongly recommended)
- Understanding of aviation emergency codes

## What You'll Build

A monitoring rule that alerts you when aircraft broadcast emergency transponder codes. These are serious aviation situations:

| Squawk | Meaning | Description |
|--------|---------|-------------|
| **7700** | General Emergency | Mayday - mechanical failure, medical emergency, fire |
| **7600** | Radio Failure | NORDO - aircraft cannot communicate via radio |
| **7500** | Hijack | Aircraft hijacking or unlawful interference |

Emergency squawks are rare but significant events. This monitor ensures you're notified immediately when one occurs in your coverage area.

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
    "notification_enabled": true,
    "cooldown": 0
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
        "cooldown":             0, // No cooldown for emergencies
    }

    body, _ := json.Marshal(rule)
    resp, err := http.Post(skyspyURL+"/api/alerts/rules", "application/json", bytes.NewReader(body))
    if err != nil {
        fmt.Printf("Error creating rule: %v\n", err)
        os.Exit(1)
    }
    defer resp.Body.Close()

    if resp.StatusCode == 201 {
        fmt.Println("Emergency monitor rule created successfully!")
        fmt.Println("You will be alerted for 7700, 7600, and 7500 squawks")
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
    "notification_enabled": True,
    "cooldown": 0  # No cooldown for emergencies
}

response = requests.post(f"{SKYSPY_URL}/api/alerts/rules", json=rule)

if response.status_code == 201:
    print("Emergency monitor rule created successfully!")
    print("You will be alerted for 7700, 7600, and 7500 squawks")
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
    notification_enabled: true,
    cooldown: 0  // No cooldown for emergencies
  })
})
.then(res => res.json())
.then(data => {
  console.log('Emergency monitor rule created successfully!');
  console.log('You will be alerted for 7700, 7600, and 7500 squawks');
  console.log('Rule ID:', data.id);
})
.catch(err => console.error('Failed to create rule:', err));
```

```json Response Example
{
  "id": 3,
  "name": "Emergency Squawk Monitor",
  "enabled": true,
  "priority": "critical",
  "conditions": {
    "operator": "OR",
    "conditions": [
      {"field": "squawk", "operator": "eq", "value": "7700"},
      {"field": "squawk", "operator": "eq", "value": "7600"},
      {"field": "squawk", "operator": "eq", "value": "7500"}
    ]
  },
  "notification_enabled": true,
  "cooldown": 0,
  "created_at": "2024-01-15T12:00:00Z"
}
```

## Create Emergency Monitor Rule

<!-- shell@1-18 -->
<!-- go@1-43 -->
<!-- python@1-29 -->
<!-- javascript@1-28 -->

Set up real-time monitoring for aircraft broadcasting emergency squawk codes. The rule uses critical priority to ensure notifications are never delayed or batched.

## Understanding Emergency Squawks

### 7700 - General Emergency (Mayday)

The most common emergency squawk. Causes include:
- Engine failure or fire
- Medical emergency on board
- Structural damage
- Fuel emergency
- Severe weather encounter
- Any situation requiring immediate landing

### 7600 - Radio Failure (NORDO)

Indicates communication failure:
- Complete radio failure
- Unable to transmit or receive
- Often benign - pilot may not be aware
- ATC uses radar to vector the aircraft

### 7500 - Hijack

The most serious code:
- Aircraft hijacking
- Unlawful interference
- Crew may squawk silently to alert ATC
- Triggers immediate security response

**Important:** 7500 is sometimes set accidentally (fat-fingering 7600). True hijacks are extremely rare.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Priority Levels

For emergencies, always use `critical`:

| Priority | Behavior |
|----------|----------|
| `critical` | Immediate notification, no batching |
| `high` | Fast notification |
| `medium` | Standard delivery |
| `low` | May be batched |

### Cooldown Setting

The `cooldown: 0` setting ensures:
- Every update triggers a notification
- No alerts are suppressed
- You're always informed of the current situation

For less critical monitoring, add a cooldown:

```json
{
  "cooldown": 300  // Only alert every 5 minutes per aircraft
}
```

## Additional Monitoring Options

### 7700 Only (Most Common Emergencies)

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "squawk", "operator": "eq", "value": "7700"}
    ]
  }
}
```

### With Distance Filter

Only alert for nearby emergencies:

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {
        "operator": "OR",
        "conditions": [
          {"field": "squawk", "operator": "eq", "value": "7700"},
          {"field": "squawk", "operator": "eq", "value": "7600"},
          {"field": "squawk", "operator": "eq", "value": "7500"}
        ]
      },
      {"field": "distance", "operator": "lt", "value": 50}
    ]
  }
}
```

## Testing & Verification

1. **Create the rule** using any of the code examples above
2. **Verify in SkySpy UI** - Go to Alerts → Rules to see your new rule
3. **Check notifications are configured** - Settings → Notifications
4. **Monitor the Alerts tab** - Emergency squawks will appear with critical priority

### Testing the Rule

Emergency squawks are rare. To test your setup:

1. Temporarily create a rule for a common squawk (like 1200)
2. Verify alerts are generated
3. Delete the test rule
4. Your emergency monitor remains active

```shell
# Temporarily test with common VFR squawk
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{"name": "Test", "conditions": {"operator": "AND", "conditions": [{"field": "squawk", "operator": "eq", "value": "1200"}]}, "enabled": true, "notification_enabled": true}'
```

## Troubleshooting

### No Alerts When Expected

**Problem:** You saw a 7700 on FlightRadar but didn't get an alert
**Solution:**
- The aircraft may have been outside your receiver range
- Check if the aircraft was visible in SkySpy at all
- Verify the rule is enabled in the Rules list

### Too Many False Alerts

**Problem:** Getting alerts that don't seem like real emergencies
**Solution:**
- Some training flights use emergency squawks
- Add a distance filter to only alert on nearby aircraft
- Add an altitude filter (emergencies at FL350 are less urgent to you)

### Notifications Not Arriving

**Problem:** Alerts appear in SkySpy but not on your phone
**Solution:**
- Verify Apprise is configured correctly
- Test notifications from Settings → Notifications
- Check your phone's notification settings
- Ensure `notification_enabled: true` on the rule

### Rule Deleted Accidentally

**Problem:** Can't find the emergency monitor rule
**Solution:** Simply run the creation code again. Rules are idempotent - creating another won't cause duplicates (though you may want to name them differently).

## What to Do When You See an Emergency

1. **Don't panic** - Many emergencies are precautionary
2. **Don't interfere** - Emergency services are already responding
3. **Monitor LiveATC** - Listen to ATC communications if available
4. **Track in SkySpy** - Watch the aircraft's progress
5. **Check news later** - Media often covers significant events

**Never** call airports, ATC, or emergency services about what you see on SkySpy. They are already aware.

## Related Recipes

- [Military Aircraft Spotter](/docs/military-spotter) - Track military flights
- [Discord Alert Bot](/docs/discord-alert-bot) - Send alerts to Discord
- [Telegram Bot](/docs/telegram-bot) - Send alerts to Telegram
- [Track Specific Aircraft](/docs/track-aircraft) - Monitor specific planes
