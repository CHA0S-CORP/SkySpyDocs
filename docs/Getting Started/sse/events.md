---
title: "Events"
slug: "sse/events"
excerpt: "SSE event types and JSON payloads."
hidden: false
---

## Event Flow

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#3b82f6', 'primaryTextColor': '#fff', 'primaryBorderColor': '#60a5fa', 'lineColor': '#60a5fa', 'actorTextColor': '#fff', 'actorBkg': '#1e3a5f', 'actorBorder': '#3b82f6'}}}%%
sequenceDiagram
    participant C as 📱 Client
    participant S as 🖥️ Server

    C->>S: 🔗 GET /api/v1/map/sse
    S->>C: ✅ Connection established

    opt 📜 replay_history=true
        S->>C: 📦 Buffered events
    end

    loop 🔄 Continuous stream
        S->>C: ✈️ aircraft_update
        S->>C: 🆕 aircraft_new
        S->>C: 🚨 safety_event
        S->>C: 💓 heartbeat (every 30s)
    end

    S->>C: 👋 aircraft_remove
```

---

## Event Reference

<Tabs>
  <Tab title="✈️ aircraft_update">
    Emitted when aircraft positions or telemetry change.

    ```json
    {
      "aircraft": [
        {
          "hex": "A12345",
          "flight": "UAL123",
          "lat": 47.6062,
          "lon": -122.3321,
          "alt": 35000,
          "gs": 450,
          "track": 270,
          "vr": 0,
          "squawk": "1200",
          "category": "A3",
          "type": "B738",
          "rssi": -12.5,
          "military": false,
          "emergency": false
        }
      ],
      "timestamp": "2024-01-15T12:00:00Z"
    }
    ```
  </Tab>
  <Tab title="🆕 aircraft_new">
    Emitted when a new aircraft enters coverage.

    ```json
    {
      "aircraft": [
        { "hex": "B67890", "flight": "DAL88" }
      ],
      "timestamp": "2024-01-15T12:00:05Z"
    }
    ```
  </Tab>
  <Tab title="👋 aircraft_remove">
    Emitted when aircraft leave coverage or signal is lost.

    ```json
    {
      "icaos": ["A12345"],
      "timestamp": "2024-01-15T12:05:00Z"
    }
    ```
  </Tab>
  <Tab title="🛡️ safety_event">
    Emitted when the safety engine detects a conflict.

    ```json
    {
      "event_type": "proximity_conflict",
      "severity": "critical",
      "icao": "A12345",
      "icao_2": "B67890",
      "callsign": "UAL123",
      "callsign_2": "DAL456",
      "message": "Proximity conflict: 0.5nm lateral, 500ft vertical",
      "details": {
        "distance_nm": 0.5,
        "altitude_diff_ft": 500
      },
      "timestamp": "2024-01-15T12:00:00Z"
    }
    ```
  </Tab>
  <Tab title="🔔 alert_triggered">
    Emitted when a custom alert rule matches.

    ```json
    {
      "rule_id": 1,
      "rule_name": "Low Altitude",
      "icao": "A12345",
      "callsign": "UAL123",
      "message": "Aircraft below 3000ft",
      "priority": "warning",
      "aircraft_data": {
        "hex": "A12345",
        "alt": 2500,
        "lat": 47.5,
        "lon": -122.3
      }
    }
    ```
  </Tab>
  <Tab title="📡 acars_message">
    Emitted when an ACARS/VDL2 message is decoded.

    ```json
    {
      "source": "acars",
      "icao_hex": "A12345",
      "registration": "N12345",
      "callsign": "UAL123",
      "label": "H1",
      "text": "CONFIRM DEPARTURE",
      "frequency": 130.025,
      "signal_level": -15.0,
      "timestamp": "2024-01-15T12:10:00Z"
    }
    ```
  </Tab>
  <Tab title="💓 heartbeat">
    Emitted every ~30 seconds to keep the connection alive.

    ```json
    {
      "count": 45,
      "timestamp": "2024-01-15T12:00:30Z"
    }
    ```
  </Tab>
</Tabs>

---

## Error Handling

SSE connections automatically reconnect on failure:

```javascript
stream.onerror = (err) => {
  console.error('SSE connection error:', err);
  // EventSource will auto-reconnect
};
```

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    subgraph Normal["✅ Normal Flow"]
        CONN["🔗 Connected"] --> DATA["📡 Receiving Data"]
    end

    subgraph Error["⚠️ Error Recovery"]
        ERR["❌ Connection Lost"] --> WAIT["⏳ Wait 3s"]
        WAIT --> RETRY["🔄 Auto-Reconnect"]
        RETRY --> CONN
    end

    DATA --> ERR

    style Normal fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
    style Error fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
```

<Warning>
**Cross-Origin Requests** — If connecting from a different origin, ensure CORS is configured on the API. The SSE endpoint supports CORS by default.
</Warning>

---

## Status Endpoint

Check SSE broadcaster status:

```bash
curl http://localhost:5000/api/v1/map/sse/status
```

```json
{
  "mode": "redis",
  "redis_enabled": true,
  "subscribers": 15,
  "tracked_aircraft": 45,
  "history": {
    "size": 500,
    "max_size": 5000
  }
}
```
