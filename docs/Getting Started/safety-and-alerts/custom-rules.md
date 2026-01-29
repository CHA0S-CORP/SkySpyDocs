---
title: "Custom Rules"
slug: "safety-and-alerts/custom-rules"
excerpt: "Rule structure, conditions, and operators."
hidden: false
---

## Rule Structure

```json
{
  "name": "Rule name shown in alerts",
  "enabled": true,
  "priority": "high",
  "conditions": {
    "operator": "AND",
    "conditions": [
      { "field": "...", "operator": "...", "value": "..." }
    ]
  },
  "notification_enabled": true
}
```

| Field | Type | Description |
| :--- | :--- | :--- |
| `name` | string | 📝 Display name for the rule |
| `enabled` | boolean | ✅ Whether the rule is active |
| `priority` | string | 🎯 `low`, `medium`, `high`, or `critical` |
| `conditions` | object | ⚙️ Condition tree (see below) |
| `notification_enabled` | boolean | 📱 Send push notifications when triggered |

---

## Condition Fields

| Field | Type | Description | Example |
| :--- | :--- | :--- | :--- |
| `icao` | string | 🔢 24-bit ICAO hex address | `A12345` |
| `callsign` | string | 🏷️ Flight number or callsign | `UAL123` |
| `squawk` | string | 📻 Transponder code | `7700` |
| `altitude` | number | 📏 Barometric altitude (feet) | `35000` |
| `distance` | number | 📍 Distance from receiver (NM) | `10` |
| `type` | string | ✈️ ICAO aircraft type code | `B738` |
| `military` | boolean | 🎖️ Military aircraft flag | `true` |
| `category` | string | 📦 Aircraft size category | `A3` |

---

## Operators

| Operator | Meaning | Works With |
| :--- | :--- | :--- |
| `eq` | ✅ Equals | All types |
| `ne` | ❌ Not equals | All types |
| `lt` | ⬇️ Less than | Numbers |
| `gt` | ⬆️ Greater than | Numbers |
| `le` | ⬇️ Less than or equal | Numbers |
| `ge` | ⬆️ Greater than or equal | Numbers |
| `contains` | 🔍 Contains substring | Strings |
| `startswith` | 🏁 Starts with | Strings |

---

## Combining Conditions

Use `AND` and `OR` operators to build complex rules. You can nest condition groups.

### AND (all must match)

Alert when a military aircraft is below 10,000ft within 25nm:

```json
{
  "operator": "AND",
  "conditions": [
    { "field": "military", "operator": "eq", "value": true },
    { "field": "altitude", "operator": "lt", "value": 10000 },
    { "field": "distance", "operator": "lt", "value": 25 }
  ]
}
```

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    M["Military = true"] --> AND{"AND"}
    A["Altitude < 10000"] --> AND
    D["Distance < 25"] --> AND
    AND --> ALERT["Alert!"]

    style AND fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
    style ALERT fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
```

### OR (any must match)

Alert for any emergency squawk:

```json
{
  "operator": "OR",
  "conditions": [
    { "field": "squawk", "operator": "eq", "value": "7700" },
    { "field": "squawk", "operator": "eq", "value": "7600" },
    { "field": "squawk", "operator": "eq", "value": "7500" }
  ]
}
```

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    S1["Squawk = 7700"] --> OR{"OR"}
    S2["Squawk = 7600"] --> OR
    S3["Squawk = 7500"] --> OR
    OR --> ALERT["Alert!"]

    style OR fill:#0d4f8b,stroke:#3b82f6,stroke-width:2px,color:#fff
    style ALERT fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
```

### Nested Logic

Alert for specific callsigns OR any military within 10nm:

```json
{
  "operator": "OR",
  "conditions": [
    { "field": "callsign", "operator": "startswith", "value": "AFR" },
    {
      "operator": "AND",
      "conditions": [
        { "field": "military", "operator": "eq", "value": true },
        { "field": "distance", "operator": "lt", "value": 10 }
      ]
    }
  ]
}
```

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart TB
    CALL["Callsign starts with AFR"] --> OR{"OR"}

    subgraph Nested["Nested AND"]
        MIL["Military = true"] --> AND{"AND"}
        DIST["Distance < 10"] --> AND
    end

    AND --> OR
    OR --> ALERT["Alert!"]

    style OR fill:#0d4f8b,stroke:#3b82f6,stroke-width:2px,color:#fff
    style Nested fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
    style ALERT fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
```

---

## Creating Rules

### Dashboard

1. **Open Alert Rules** - Navigate to **Settings** → **Alert Rules** in the dashboard
2. **Create New Rule** - Click **New Rule** to open the rule builder
3. **Configure Conditions** - Add conditions using the visual builder and set priority
4. **Enable Notifications** - Toggle push notifications if desired

### API

```bash
curl -X POST http://localhost:5000/api/alerts/rules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Military Aircraft Nearby",
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
  }'
```
