---
title: "Rule Examples"
slug: "safety-and-alerts/examples"
excerpt: "Common alert rule patterns and use cases."
hidden: false
---

## Track Specific Aircraft

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

---

## Low-Flying Aircraft Nearby

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

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    A["📏 Altitude < 1000ft"] --> AND{"✅ AND"}
    D["📍 Distance < 5nm"] --> AND
    AND --> ALERT["🔔 Alert!"]

    style AND fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
    style ALERT fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
```

---

## Airline Callsign Prefix

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

---

## Heavy Jets Overhead

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

---

## Military Aircraft

```json
{
  "name": "Military Alert",
  "priority": "high",
  "conditions": {
    "operator": "AND",
    "conditions": [
      { "field": "military", "operator": "eq", "value": true },
      { "field": "distance", "operator": "lt", "value": 25 }
    ]
  },
  "notification_enabled": true
}
```

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    M["🎖️ Military = true"] --> AND{"✅ AND"}
    D["📍 Distance < 25nm"] --> AND
    AND --> ALERT["🔔 Alert!"]

    style AND fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
    style ALERT fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
```

---

## Emergency Squawks

```json
{
  "name": "Emergency Alert",
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
}
```

---

## Specific Aircraft Type

```json
{
  "name": "Boeing 747 Spotter",
  "priority": "medium",
  "conditions": {
    "operator": "AND",
    "conditions": [
      { "field": "type", "operator": "startswith", "value": "B74" },
      { "field": "distance", "operator": "lt", "value": 50 }
    ]
  },
  "notification_enabled": true
}
```
