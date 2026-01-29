---
title: Track Specific Aircraft
excerpt: Monitor specific aircraft by tail number, ICAO hex code, or callsign.
hidden: false
recipe:
  color: '#018FF4'
  icon: ✈️
difficulty: beginner
tags: [tracking, alerts, watchlist, built-in]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- The ICAO hex code, tail number, or callsign of the aircraft you want to track
- Push notifications configured (optional, for mobile alerts)

## What You'll Build

An alert rule that notifies you whenever a specific aircraft appears in your receiver's coverage area. This is useful for tracking:

- **Your own aircraft** - Get notified when you or friends are flying
- **Celebrity/VIP aircraft** - Track known private jets
- **Specific airline flights** - Monitor regular routes
- **Historic aircraft** - Watch for warbirds or rare types

## Finding Aircraft Identifiers

| Identifier | Description | Example | Best For |
|------------|-------------|---------|----------|
| ICAO Hex | Permanent 24-bit address | `A12345` | Most reliable |
| Tail Number | Registration (N-number) | `N12345` | US aircraft |
| Callsign | Flight identifier | `UAL123` | Specific flights |

### Convert Tail Number to ICAO

Use the FAA registry or online calculators to convert:
- `N12345` → `A12345` (example)

```shell Shell
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Track N12345",
    "enabled": true,
    "priority": "high",
    "conditions": {
      "operator": "OR",
      "conditions": [
        { "field": "hex", "operator": "eq", "value": "A12345" },
        { "field": "flight", "operator": "eq", "value": "N12345" }
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

    // Track by both ICAO hex and callsign for reliability
    rule := map[string]interface{}{
        "name":     "Track N12345",
        "enabled":  true,
        "priority": "high",
        "conditions": map[string]interface{}{
            "operator": "OR",
            "conditions": []map[string]interface{}{
                {"field": "hex", "operator": "eq", "value": "A12345"},
                {"field": "flight", "operator": "eq", "value": "N12345"},
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
        fmt.Println("Aircraft tracking rule created!")
    } else {
        fmt.Printf("Failed: %s\n", resp.Status)
    }
}
```

```python Python
import os
import requests

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

# Track by both ICAO hex and callsign for reliability
rule = {
    "name": "Track N12345",
    "enabled": True,
    "priority": "high",
    "conditions": {
        "operator": "OR",
        "conditions": [
            {"field": "hex", "operator": "eq", "value": "A12345"},
            {"field": "flight", "operator": "eq", "value": "N12345"}
        ]
    },
    "notification_enabled": True
}

response = requests.post(f"{SKYSPY_URL}/api/alerts/rules", json=rule)

if response.status_code == 201:
    print("Aircraft tracking rule created!")
    print(f"Rule ID: {response.json().get('id')}")
else:
    print(f"Failed: {response.status_code}")


# Track multiple aircraft with one rule
def create_watchlist(aircraft_list):
    """Create a rule to track multiple aircraft."""
    conditions = [
        {"field": "hex", "operator": "eq", "value": hex_code}
        for hex_code in aircraft_list
    ]

    rule = {
        "name": "Aircraft Watchlist",
        "enabled": True,
        "priority": "high",
        "conditions": {
            "operator": "OR",
            "conditions": conditions
        },
        "notification_enabled": True
    }

    return requests.post(f"{SKYSPY_URL}/api/alerts/rules", json=rule)


# Example: Track multiple VIP aircraft
# create_watchlist(["A835AF", "A326CA", "A45E85"])
```

```javascript JavaScript
const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';

// Track by both ICAO hex and callsign for reliability
fetch(`${SKYSPY_URL}/api/alerts/rules`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    name: 'Track N12345',
    enabled: true,
    priority: 'high',
    conditions: {
      operator: 'OR',
      conditions: [
        { field: 'hex', operator: 'eq', value: 'A12345' },
        { field: 'flight', operator: 'eq', value: 'N12345' }
      ]
    },
    notification_enabled: true
  })
})
.then(res => res.json())
.then(data => console.log('Rule created:', data.id))
.catch(err => console.error('Failed:', err));

// Track multiple aircraft
async function createWatchlist(hexCodes) {
  const conditions = hexCodes.map(hex => ({
    field: 'hex',
    operator: 'eq',
    value: hex
  }));

  const response = await fetch(`${SKYSPY_URL}/api/alerts/rules`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      name: 'Aircraft Watchlist',
      enabled: true,
      priority: 'high',
      conditions: { operator: 'OR', conditions },
      notification_enabled: true
    })
  });

  return response.json();
}

// Example: Track multiple VIP aircraft
// createWatchlist(['A835AF', 'A326CA', 'A45E85']);
```

```json Response Example
{
  "id": 2,
  "name": "Track N12345",
  "enabled": true,
  "priority": "high",
  "conditions": {
    "operator": "OR",
    "conditions": [
      {"field": "hex", "operator": "eq", "value": "A12345"},
      {"field": "flight", "operator": "eq", "value": "N12345"}
    ]
  },
  "notification_enabled": true,
  "created_at": "2024-01-15T12:00:00Z"
}
```

## Create Tracking Rule

<!-- shell@1-16 -->
<!-- go@1-40 -->
<!-- python@1-25 -->
<!-- javascript@1-23 -->

Create an alert rule that matches your target aircraft by ICAO hex code. Using OR conditions allows matching by either hex or callsign for reliability.

## Track Multiple Aircraft

<!-- python@28-48 -->
<!-- javascript@25-46 -->

Create a watchlist to track multiple aircraft with a single rule. This is more efficient than creating separate rules for each aircraft.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Condition Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equals | `"A12345"` |
| `contains` | Contains substring | `"UAL"` matches `UAL123` |
| `startswith` | Starts with | `"N"` matches N-numbers |
| `regex` | Regex match | `"^RCH[0-9]+$"` |

### Example: Track by Callsign Pattern

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "flight", "operator": "startswith", "value": "UAL"}
    ]
  }
}
```

This tracks all United Airlines flights.

## Common Tracked Aircraft

### VIP/Celebrity Jets

| Aircraft | ICAO | Notes |
|----------|------|-------|
| Air Force One | Variable | Uses different callsigns |
| Trump Jet | `A835AF` | Boeing 757 N757AF |
| Musk Jet | Various | Multiple aircraft |

### Historic Aircraft

| Aircraft | ICAO | Notes |
|----------|------|-------|
| Doc (B-29) | `A8A0E7` | Based in Wichita |
| Fifi (B-29) | `A2A2A2` | CAF aircraft |

## Testing & Verification

1. **Create the rule** using any of the code examples above
2. **Verify in SkySpy UI** - Go to Alerts → Rules to see your new rule
3. **Wait for the aircraft** - Or use FlightAware to check when it might be in range
4. **Check notifications** - If push notifications are configured, you'll receive mobile alerts

### Simulate for Testing

If the aircraft isn't flying, test with a more common aircraft:

```python
# Temporarily track any aircraft for testing
rule = {
    "name": "Test Rule",
    "conditions": {
        "operator": "AND",
        "conditions": [
            {"field": "alt", "operator": "gt", "value": 0}
        ]
    }
}
```

## Troubleshooting

### Aircraft Not Detected

**Problem:** The aircraft flew over but no alert was generated
**Solution:**
- Verify the ICAO hex code is correct (check FlightAware/FlightRadar24)
- The aircraft might have been ADS-B out equipped (not all aircraft are)
- Check if the aircraft was in range of your receiver

### Wrong ICAO Code

**Problem:** Using tail number instead of ICAO hex
**Solution:** Convert the tail number:
- US: Use FAA registry or N-number calculator
- International: Check country-specific registries

### Callsign Changes

**Problem:** Commercial flights use different callsigns for same route
**Solution:** Track by ICAO hex instead of callsign, or use multiple conditions:

```json
{
  "conditions": {
    "operator": "OR",
    "conditions": [
      {"field": "hex", "operator": "eq", "value": "A12345"},
      {"field": "flight", "operator": "startswith", "value": "UAL"}
    ]
  }
}
```

## Related Recipes

- [Military Aircraft Spotter](/docs/military-spotter) - Track military aircraft
- [VIP Aircraft Tracker](/docs/vip-tracker) - Pre-built VIP watchlists
- [Discord Alert Bot](/docs/discord-alert-bot) - Send alerts to Discord
- [Geofence Alerts](/docs/geofence-alerts) - Alert when aircraft enter zones
