---
title: "SSE Streaming API"
slug: "sse"
excerpt: "Lightweight, one-way real-time data streaming using Server-Sent Events."
hidden: false
---

For applications that only need to receive data without sending messages back to the server, SkySpy provides a standard **Server-Sent Events (SSE)** stream. This is a lightweight alternative to the Socket.IO API, supported natively in all modern browsers.

## Connection Details

| Setting | Value |
| :--- | :--- |
| **Endpoint** | `/api/v1/map/sse` |
| **Method** | `GET` |
| **Content-Type** | `text/event-stream` |

### Query Parameters

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `replay_history` | `boolean` | `false` | If `true`, the server will replay recent events (buffered in memory/Redis) immediately upon connection. |

## Client Implementation

You can connect using the native browser `EventSource` API.

```javascript
const stream = new EventSource('http://localhost:5000/api/v1/map/sse?replay_history=true');

stream.onopen = () => {
  console.log('Connected to SkySpy SSE Stream');
};

stream.addEventListener('aircraft_update', (event) => {
  const data = JSON.parse(event.data);
  console.log('Updated Aircraft:', data.aircraft);
});

stream.addEventListener('safety_event', (event) => {
  const warning = JSON.parse(event.data);
  console.warn('Safety Alert:', warning.message);
});

stream.onerror = (err) => {
  console.error('Stream connection lost', err);
  stream.close();
};

```

## Event Reference

The stream emits specific event types that you can listen for using `addEventListener`.

### `aircraft_update`

Emitted when aircraft positions or telemetry change. The payload contains a list of aircraft objects that have changed since the last update.

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

### `aircraft_new`

Emitted when a new aircraft enters the coverage area.

```json
{
  "aircraft": [
    { "hex": "B67890", "flight": "DAL88" }
  ],
  "timestamp": "2024-01-15T12:00:05Z"
}

```

### `aircraft_remove`

Emitted when an aircraft leaves the coverage area or signals are lost.

```json
{
  "icaos": ["A12345"],
  "timestamp": "2024-01-15T12:05:00Z"
}

```

### `safety_event`

Emitted when the safety monitoring engine detects a conflict (TCAS RA, proximity, etc.).

```json
{
  "event_type": "proximity_conflict",
  "severity": "critical",
  "icao": "A12345",
  "icao_2": "B67890",
  "callsign": "UAL123",
  "callsign_2": "DAL456",
  "message": "Proximity conflict: 0.5nm lateral, 500ft vertical separation",
  "details": {
    "distance_nm": 0.5,
    "altitude_diff_ft": 500
  },
  "timestamp": "2024-01-15T12:00:00Z"
}

```

### `alert_triggered`

Emitted when a user-defined custom alert rule is matched.

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

### `acars_message`

Emitted when a new text message is decoded from ACARS or VDL2.

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

### `heartbeat`

Emitted periodically (approx. every 30 seconds) to keep the connection alive and sync aircraft counts.

```json
{
  "count": 45,
  "timestamp": "2024-01-15T12:00:30Z"
}

```

## Service Status

You can check the status of the SSE broadcaster, including subscriber counts and mode (Memory vs. Redis), via the status endpoint.

**Endpoint**: `GET /api/v1/map/sse/status`

**Response:**

```json
{
  "mode": "redis",
  "redis_enabled": true,
  "subscribers": 15,
  "subscribers_local": 2,
  "tracked_aircraft": 45,
  "last_publish": "2024-01-15T12:00:00Z",
  "history": {
    "size": 500,
    "max_size": 5000
  },
  "timestamp": "2024-01-15T12:00:01Z"
}

```
