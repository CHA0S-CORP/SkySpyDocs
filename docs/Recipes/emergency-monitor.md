---
title: "Emergency Alert Monitor"
slug: "emergency-monitor"
excerpt: "Monitor emergency squawks (7700, 7600, 7500) in your area."
hidden: false
---

Set up real-time monitoring for aircraft broadcasting emergency squawk codes.

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    subgraph Squawks["🚨 Emergency Squawks"]
        S7700["🆘 7700 - Emergency"]
        S7600["📻 7600 - Radio Failure"]
        S7500["⚠️ 7500 - Hijack"]
    end

    subgraph Alert["🔔 Immediate Alert"]
        PUSH["📱 Push Notification"]
        DASH["🖥️ Dashboard Alert"]
    end

    Squawks --> Alert

    style Squawks fill:#991b1b,stroke:#ef4444,stroke-width:2px,color:#fff
    style Alert fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
```

## Emergency Squawk Codes

<Warning>
These codes indicate serious aviation emergencies. SkySpy automatically detects and highlights these.
</Warning>

| Code | Meaning | Description |
| :--- | :--- | :--- |
| **7700** | 🚨 General Emergency | Mayday - aircraft in distress |
| **7600** | 📻 Radio Failure | Lost communications (NORDO) |
| **7500** | ⚠️ Hijack | Unlawful interference |

---

## Built-in Safety Monitoring

Enable in your `.env` file:

```bash
SAFETY_MONITORING_ENABLED=true
```

When enabled, emergency squawks trigger dashboard alerts and push notifications automatically.

---

## Custom Alert Rule

```bash
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

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart TB
    S1["🆘 Squawk = 7700"] --> OR{"☑️ OR"}
    S2["📻 Squawk = 7600"] --> OR
    S3["⚠️ Squawk = 7500"] --> OR
    OR --> ALERT["🚨 CRITICAL ALERT"]

    style OR fill:#991b1b,stroke:#ef4444,stroke-width:2px,color:#fff
    style ALERT fill:#7f1d1d,stroke:#dc2626,stroke-width:2px,color:#fff
```

---

## Dashboard Integration

<CardGroup cols={2}>
  <Card title="Visual Highlighting" icon="eye">
    Emergency aircraft shown in red with blinking indicator
  </Card>
  <Card title="Priority Sorting" icon="arrow-up">
    Emergency traffic sorted to top of aircraft list
  </Card>
</CardGroup>

---

## Important Notes

<Info>
**False Positives**: Pilots occasionally squawk emergency codes accidentally or during training.
</Info>

<Warning>
**Do Not Interfere**: If you observe a real emergency, do not attempt to contact the aircraft. ATC and first responders are already handling it.
</Warning>

---

## Next Steps

<Cards columns={2}>
  <Card title="Safety & Alerts" icon="shield" href="/docs/safety-and-alerts">
    Configure all safety monitoring options
  </Card>
  <Card title="Discord Alert Bot" icon="discord" href="/docs/discord-alert-bot">
    Send emergency alerts to Discord
  </Card>
</Cards>
