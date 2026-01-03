---
title: "Real-Time API"
slug: "real-time-api"
excerpt: "Comprehensive guide to the SkySpy Socket.IO streaming API, including events, topics, and on-demand data requests."
hidden: false
---

The SkySpy Real-Time API is built on **Socket.IO**, providing a robust, bi-directional communication channel for streaming aircraft data, safety alerts, and aviation intelligence.

## Connection

To connect to the real-time stream, establish a Socket.IO connection to the API server.

| Setting | Value |
| :--- | :--- |
| **Base URL** | `http://<host>:5000` |
| **Path** | `/socket.io/socket.io` |
| **Transports** | `websocket`, `polling` |
| **Query Params** | `topics` (comma-separated list of rooms to join) |

### Client Implementation

#### Vanilla JavaScript
```javascript
import { io } from 'socket.io-client';

const socket = io('http://localhost:5000', {
  path: '/socket.io/socket.io',
  query: { topics: 'aircraft,safety,alerts' },
  transports: ['websocket', 'polling']
});

socket.on('connect', () => {
  console.log('Connected with ID:', socket.id);
});

```

#### React Hook Example

```javascript
import { useEffect, useState } from 'react';
import { io } from 'socket.io-client';

export function useSkySpySocket(topics = 'all') {
  const [connected, setConnected] = useState(false);

  useEffect(() => {
    const socket = io('http://localhost:5000', {
      path: '/socket.io/socket.io',
      query: { topics },
      transports: ['websocket', 'polling']
    });

    socket.on('connect', () => setConnected(true));
    socket.on('disconnect', () => setConnected(false));

    return () => socket.disconnect();
  }, [topics]);

  return { connected };
}

```

---

## Subscription Topics

You can subscribe to specific data streams to optimize bandwidth. These can be set via the `topics` query parameter or dynamically using `subscribe` events.

| Topic | Events Included | Description |
| --- | --- | --- |
| `aircraft` | `aircraft:*` | Live positions, metadata updates, and removals. |
| `safety` | `safety:event` | TCAS conflicts, proximity warnings, and extreme vertical rates. |
| `alerts` | `alert:triggered` | User-defined custom alert rule matches. |
| `airspace` | `airspace:*` | G-AIRMET advisories and airspace boundary definitions. |
| `acars` | `acars:message` | Real-time ACARS and VDL2 text messages. |
| `all` | *All Events* | Subscribes to all available rooms. |

> 📘 Dynamic Subscription
> You can change subscriptions after connecting:
> ```javascript
> socket.emit('subscribe', { topics: ['acars'] });
> socket.emit('unsubscribe', { topics: ['safety'] });
> 
> ```
> 
> 

---

## Event Reference

### Aircraft Events

These events are broadcast to the `aircraft` topic.

#### `aircraft:snapshot`

Sent immediately upon connection or topic subscription. Contains the full state of all currently tracked aircraft.

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
      "vr": -500,
      "squawk": "1200",
      "category": "A3",
      "type": "B738",
      "military": false,
      "emergency": false
    }
  ],
  "count": 1,
  "timestamp": "2024-01-15T12:00:00Z"
}

```

#### `aircraft:update`

Emitted when tracking data changes (position, altitude, speed, etc.).

```json
{
  "aircraft": [
    {
      "hex": "A12345",
      "alt": 35100,
      "vr": 1200
    }
  ],
  "timestamp": "2024-01-15T12:00:01Z"
}

```

#### `aircraft:remove`

Emitted when aircraft leave coverage or signal is lost.

```json
{
  "icaos": ["A12345", "B67890"],
  "timestamp": "2024-01-15T12:05:00Z"
}

```

### Safety Events

Broadcast to the `safety` topic when the monitoring engine detects a conflict.

#### `safety:event`

| Field | Type | Description |
| --- | --- | --- |
| `event_type` | string | `proximity_conflict`, `tcas_ra_detected`, `extreme_vertical_rate`, `emergency_squawk` |
| `severity` | string | `warning` or `critical` |
| `details` | object | Context specific to the event type. |

```json
{
  "event_type": "proximity_conflict",
  "severity": "critical",
  "icao": "A12345",
  "icao_2": "B67890",
  "callsign": "UAL123",
  "message": "Proximity conflict: 0.5nm lateral, 500ft vertical separation",
  "details": {
    "distance_nm": 0.5,
    "altitude_diff_ft": 500
  },
  "timestamp": "2024-01-15T12:00:00Z"
}

```

### Alert Events

Broadcast to the `alerts` topic when a custom user rule is matched.

#### `alert:triggered`

```json
{
  "rule_id": 1,
  "rule_name": "Low Altitude Alert",
  "icao": "A12345",
  "callsign": "UAL123",
  "message": "Aircraft below 3000ft within 10nm",
  "priority": "warning",
  "aircraft_data": {
    "hex": "A12345",
    "alt": 2500,
    "lat": 47.6,
    "lon": -122.3
  }
}

```

### ACARS Events

Broadcast to the `acars` topic.

#### `acars:message`

```json
{
  "source": "acars",
  "icao_hex": "A12345",
  "registration": "N12345",
  "label": "H1",
  "text": "DEPARTURE CLEARANCE CONFIRMED SEA",
  "frequency": 130.025,
  "signal_level": -42.5
}

```

---

## Request/Response API

SkySpy supports an RPC-style Request/Response pattern over Socket.IO. This allows the frontend to fetch heavy aviation data (like weather or airspace boundaries) on-demand without managing separate HTTP endpoints.

### Usage Pattern

1. **Emit** a `request` event with a unique `request_id`.
2. **Listen** for a `response` (success) or `error` event matching that ID.

### Sending a Request

```javascript
const requestId = 'req-12345';

socket.emit('request', {
  type: 'pireps',
  request_id: requestId,
  params: { 
    lat: 47.5, 
    lon: -122.3, 
    radius: 150 
  }
});

```

### Receiving Data

```javascript
socket.on('response', (data) => {
  if (data.request_id === 'req-12345') {
    console.log('Received PIREPs:', data.data);
  }
});

socket.on('error', (err) => {
  if (err.request_id === 'req-12345') {
    console.error('Request failed:', err.error);
  }
});

```

### Supported Request Types

| Request Type | Required Params | Optional Params | Description |
| --- | --- | --- | --- |
| `airspaces` | `lat`, `lon` | `hazard` | Get G-AIRMET advisories for a location. |
| `airspace-boundaries` | - | `lat`, `lon`, `radius`, `class` | Get static airspace boundary geometry. |
| `pireps` | `lat`, `lon` | `radius`, `hours` | Get Pilot Reports (turbulence, icing). |
| `metars` | `lat`, `lon` | `radius`, `hours`, `limit` | Get METARs for airports in area. |
| `metar` | `station` | `hours` | Get specific station METAR. |
| `taf` | `station` | - | Get Terminal Aerodrome Forecast. |
| `sigmets` | - | `hazard`, `lat`, `lon`, `radius` | Get active SIGMETs. |
| `airports` | `lat`, `lon` | `radius`, `limit` | Get nearby airport info. |
| `navaids` | `lat`, `lon` | `radius`, `limit`, `type` | Get nearby VORs/NDBs. |
| `aircraft-info` | `icao` | - | Get static database info for an ICAO. |
