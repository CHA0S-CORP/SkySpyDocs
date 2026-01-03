---
title: "Safety & Alerts"
slug: "safety-and-alerts"
excerpt: "Configure safety monitoring and custom alert rules for aircraft tracking."
hidden: false
---

SkySpy provides two types of alerts: automatic **safety monitoring** that detects dangerous conditions, and **custom alert rules** you define to track specific aircraft or situations.

## Safety Monitoring

The safety engine continuously analyzes live traffic and automatically detects these events:

| Event | Trigger | Severity |
| :--- | :--- | :--- |
| **TCAS RA** | Resolution Advisory detected in ADS-B data | Critical |
| **TCAS TA** | Traffic Advisory detected | Warning |
| **Proximity Conflict** | Aircraft within threshold distance (default: 1.0 NM) | Critical |
| **Extreme Vertical Rate** | Climb/descent exceeding 4,500 ft/min | Warning |
| **Emergency Squawk** | Transponder code 7700, 7600, or 7500 | Critical |

<Accordion title="Emergency squawk codes">

| Code | Meaning | Description |
| :--- | :--- | :--- |
| `7700` | General Emergency | Aircraft in distress |
| `7600` | Radio Failure | Lost communications (NORDO) |
| `7500` | Hijack | Unlawful interference |

</Accordion>

### Configuration

Control safety monitoring thresholds in your `.env` file:

```bash
SAFETY_MONITORING_ENABLED=true
SAFETY_PROXIMITY_NM=1.0        # Nautical miles
SAFETY_ALTITUDE_DIFF_FT=1000   # Feet
```

See [Configuration](/docs/configuration#safety-monitoring) for all options.

## Custom Alert Rules

Create rules to get notified when specific aircraft appear or conditions are met. Rules support flexible AND/OR logic to combine multiple conditions.

### Creating Rules

<Tabs>
  <Tab title="Dashboard">

1. Open the SkySpy dashboard
2. Navigate to **Settings** → **Alert Rules**
3. Click **New Rule**
4. Configure conditions and save

  </Tab>
  <Tab title="API">

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

  </Tab>
</Tabs>

### Rule Structure

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
| `name` | string | Display name for the rule |
| `enabled` | boolean | Whether the rule is active |
| `priority` | string | `low`, `medium`, `high`, or `critical` |
| `conditions` | object | Condition tree (see below) |
| `notification_enabled` | boolean | Send push notifications when triggered |

### Condition Fields

| Field | Type | Description | Example |
| :--- | :--- | :--- | :--- |
| `icao` | string | 24-bit ICAO hex address | `A12345` |
| `callsign` | string | Flight number or callsign | `UAL123` |
| `squawk` | string | Transponder code | `7700` |
| `altitude` | number | Barometric altitude (feet) | `35000` |
| `distance` | number | Distance from receiver (NM) | `10` |
| `type` | string | ICAO aircraft type code | `B738` |
| `military` | boolean | Military aircraft flag | `true` |
| `category` | string | Aircraft size category | `A3` |

### Operators

| Operator | Meaning | Works With |
| :--- | :--- | :--- |
| `eq` | Equals | All types |
| `ne` | Not equals | All types |
| `lt` | Less than | Numbers |
| `gt` | Greater than | Numbers |
| `le` | Less than or equal | Numbers |
| `ge` | Greater than or equal | Numbers |
| `contains` | Contains substring | Strings |
| `startswith` | Starts with | Strings |

### Combining Conditions

Use `AND` and `OR` operators to build complex rules. You can nest condition groups.

<Tabs>
  <Tab title="AND (all must match)">

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

  </Tab>
  <Tab title="OR (any must match)">

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

  </Tab>
  <Tab title="Nested">

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

  </Tab>
</Tabs>

## Example Rules

<Accordion title="Track specific aircraft by ICAO">

```json
{
  "name": "Track N12345",
  "priority": "medium",
  "conditions": {
    "operator": "OR",
    "conditions": [
      { "field": "icao", "operator": "eq", "value": "A12345" },
      { "field": "icao", "operator": "eq", "value": "A67890" }
    ]
  },
  "notification_enabled": true
}
```

</Accordion>

<Accordion title="Low-flying aircraft nearby">

```json
{
  "name": "Low Flyer Alert",
  "priority": "high",
  "conditions": {
    "operator": "AND",
    "conditions": [
      { "field": "altitude", "operator": "lt", "value": 1000 },
      { "field": "distance", "operator": "lt", "value": 5 }
    ]
  },
  "notification_enabled": true
}
```

</Accordion>

<Accordion title="Airline callsign prefix">

```json
{
  "name": "United Airlines",
  "priority": "low",
  "conditions": {
    "operator": "AND",
    "conditions": [
      { "field": "callsign", "operator": "startswith", "value": "UAL" }
    ]
  },
  "notification_enabled": false
}
```

</Accordion>

<Accordion title="Heavy jets overhead">

```json
{
  "name": "Heavy Traffic",
  "priority": "medium",
  "conditions": {
    "operator": "AND",
    "conditions": [
      { "field": "category", "operator": "eq", "value": "A5" },
      { "field": "distance", "operator": "lt", "value": 20 }
    ]
  },
  "notification_enabled": true
}
```

</Accordion>

## Notifications

When alerts trigger, SkySpy can send push notifications via [Apprise](https://github.com/caronc/apprise). Configure notification services in your `.env`:

```bash
APPRISE_URLS="pushover://user@token;tgram://bot/chat"
NOTIFICATION_COOLDOWN=300
```

The cooldown prevents duplicate notifications for the same alert within the specified seconds.

See [Configuration → Notifications](/docs/configuration#notifications) for setup details.