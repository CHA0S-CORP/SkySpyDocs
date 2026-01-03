---
title: "Safety & Alerts"
slug: "safety-and-alerts"
excerpt: "Configure safety monitoring and custom alert rules for aircraft tracking."
hidden: false
---

SkySpy provides two types of alerts: automatic **safety monitoring** that detects dangerous conditions, and **custom alert rules** you define to track specific aircraft or situations.

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    subgraph Input["📡 Live Traffic"]
        AC["✈️ Aircraft Data"]
    end

    subgraph Detection["🔍 Detection"]
        SAFETY["🛡️ Safety Engine"]
        RULES["📋 Custom Rules"]
    end

    subgraph Output["📬 Alerts"]
        DASH["🖥️ Dashboard"]
        PUSH["📱 Push Notifications"]
    end

    AC --> SAFETY
    AC --> RULES
    SAFETY --> DASH
    SAFETY --> PUSH
    RULES --> DASH
    RULES --> PUSH

    style Input fill:#0d4f8b,stroke:#3b82f6,stroke-width:2px,color:#fff
    style Detection fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
    style Output fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
```

## Safety Monitoring

The safety engine continuously analyzes live traffic and automatically detects dangerous conditions.

<CardGroup cols={2}>
  <Card title="🚨 TCAS RA" icon="circle-exclamation">
    **Critical** — Resolution Advisory detected
  </Card>
  <Card title="⚠️ TCAS TA" icon="triangle-exclamation">
    **Warning** — Traffic Advisory detected
  </Card>
  <Card title="🎯 Proximity Conflict" icon="arrows-to-circle">
    **Critical** — Aircraft within threshold distance
  </Card>
  <Card title="↕️ Extreme Vertical Rate" icon="arrows-up-down">
    **Warning** — Climb/descent exceeding 4,500 ft/min
  </Card>
  <Card title="📻 Emergency Squawk" icon="radio">
    **Critical** — 7700, 7600, or 7500 detected
  </Card>
</CardGroup>

### Emergency Squawk Codes

<Warning>
These codes indicate serious aviation emergencies and trigger immediate alerts.
</Warning>

| Code | Icon | Meaning |
| :--- | :--- | :--- |
| `7700` | 🚨 | **General Emergency** — Aircraft in distress |
| `7600` | 📻 | **Radio Failure** — Lost communications (NORDO) |
| `7500` | ⚠️ | **Hijack** — Unlawful interference |

### Configuration

```bash
SAFETY_MONITORING_ENABLED=true
SAFETY_PROXIMITY_NM=1.0        # Nautical miles
SAFETY_ALTITUDE_DIFF_FT=1000   # Feet
```

---

## Custom Alert Rules

Create rules to get notified when specific aircraft appear or conditions are met.

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart TB
    subgraph Rule["📋 Alert Rule"]
        direction TB
        NAME["📝 Rule Name"]
        COND["⚙️ Conditions"]
        PRIO["🎯 Priority Level"]
    end

    subgraph Fields["📊 Available Fields"]
        direction LR
        ICAO["🔢 ICAO Hex"]
        CALL["🏷️ Callsign"]
        ALT["📏 Altitude"]
        DIST["📍 Distance"]
        MIL["🎖️ Military"]
    end

    Rule --> Fields

    style Rule fill:#0d4f8b,stroke:#3b82f6,stroke-width:2px,color:#fff
    style Fields fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
```

### Quick Example

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

---

## Notifications

When alerts trigger, SkySpy sends push notifications via [Apprise](https://github.com/caronc/apprise).

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    ALERT["🔔 Alert Triggered"] --> APPRISE["📤 Apprise"]

    APPRISE --> PO["📱 Pushover"]
    APPRISE --> TG["✈️ Telegram"]
    APPRISE --> DC["💬 Discord"]
    APPRISE --> MORE["🔌 80+ more..."]

    style ALERT fill:#991b1b,stroke:#ef4444,stroke-width:2px,color:#fff
    style APPRISE fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
```

```bash
APPRISE_URLS="pushover://user@token;tgram://bot/chat"
NOTIFICATION_COOLDOWN=300
```

---

## Implementation

<Cards columns={2}>
  <Card title="📋 Custom Rules" icon="list-check" href="/docs/safety-and-alerts/custom-rules">
    Rule structure, conditions, and operators
  </Card>
  <Card title="📝 Rule Examples" icon="code" href="/docs/safety-and-alerts/examples">
    Common rule patterns and use cases
  </Card>
</Cards>

---

## Next Steps

<Cards columns={2}>
  <Card title="Real-Time API" icon="bolt" href="/docs/real-time-api">
    Subscribe to safety events via Socket.IO
  </Card>
  <Card title="SSE Streaming" icon="signal-stream" href="/docs/sse">
    Receive alerts via Server-Sent Events
  </Card>
</Cards>
