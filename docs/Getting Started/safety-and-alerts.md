---
title: "Safety & Alerts"
slug: "safety-and-alerts"
excerpt: "Configuring custom alert rules and understanding safety monitoring events."
hidden: false
---

## Safety Monitoring Engine

SkySpy automatically monitors live traffic for safety-related events. These events are logged to the database, displayed on the dashboard, and can trigger push notifications.

**Detected Events:**
* **TCAS Alerts**: Resolution Advisory (RA) and Traffic Advisory (TA) detection.
* **Proximity Warnings**: Aircraft within the configurable distance threshold (Default: 1.0 NM).
* **Extreme Vertical Rates**: Climb or descent rates exceeding 4,500 ft/min.
* **Emergency Squawks**:
    * `7700`: General Emergency
    * `7600`: Radio Failure
    * `7500`: Hijack

## Custom Alert Rules

You can create custom alerts using flexible condition logic via the API or Dashboard.

### Rule Structure

A rule consists of a name, priority, and a set of conditions using `AND`/`OR` logic.

**Example JSON Payload:**
```json
{
  "name": "Military Aircraft Alert",
  "enabled": true,
  "priority": "high",
  "conditions": {
    "operator": "AND",
    "conditions": [
      { "field": "military", "operator": "eq", "value": true },
      { "field": "distance", "operator": "lt", "value": 50 }
    ]
  },
  "notification_enabled": true
}

```

### Available Fields & Operators

| Field | Description |
| --- | --- |
| `icao` | The 24-bit ICAO hex code. |
| `callsign` | The flight number or callsign. |
| `squawk` | Transponder code. |
| `altitude` | Barometric altitude in feet. |
| `distance` | Distance from feeder in Nautical Miles. |
| `type` | Aircraft type code (e.g., B737). |
| `military` | Boolean flag for military database matches. |

**Operators:** `eq` (equals), `ne` (not equals), `lt` (less than), `gt` (greater than), `le` (less or equal), `ge` (greater or equal), `contains`, `startswith`.


