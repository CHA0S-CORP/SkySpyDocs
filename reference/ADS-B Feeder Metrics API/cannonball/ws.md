---
title: "Cannonball WebSocket"
excerpt: "Real-time threat updates via WebSocket connection"
hidden: false
---

## WebSocket Connection

```
wss://api.skyspy.io/ws/cannonball/
```

The Cannonball WebSocket provides real-time threat detection optimized for mobile devices. It maintains a persistent connection for receiving threat updates based on your GPS position.

## Connection

Connect to the WebSocket endpoint. Upon successful connection, you will receive a session confirmation:

```json
{
  "type": "session_started",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2024-01-15T12:30:00Z"
}
```

## Client Messages

### Position Update

Send your GPS position to receive filtered threats:

```json
{
  "type": "position_update",
  "lat": 34.0522,
  "lon": -118.2437,
  "heading": 180,
  "accuracy": 10
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | Must be `position_update` |
| `lat` | float | Yes | Latitude |
| `lon` | float | Yes | Longitude |
| `heading` | float | No | Device heading in degrees (0-360) |
| `accuracy` | float | No | GPS accuracy in meters |

### Set Threat Radius

Adjust the threat detection radius:

```json
{
  "type": "set_radius",
  "radius_nm": 15.0
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | Must be `set_radius` |
| `radius_nm` | float | Yes | Detection radius in nautical miles (default: 25) |

**Response:**

```json
{
  "type": "radius_updated",
  "radius_nm": 15.0
}
```

### Get Threats

Request current threats without updating position:

```json
{
  "type": "get_threats"
}
```

This returns threats based on your last known position.

## Server Messages

### Threats Response

Received after position update or get_threats request:

```json
{
  "type": "threats",
  "data": [
    {
      "hex": "A12345",
      "callsign": "N123HP",
      "category": "Law Enforcement",
      "description": "LAPD helicopter",
      "distance_nm": 2.5,
      "bearing": 45.0,
      "relative_bearing": 135.0,
      "direction": "NE",
      "altitude": 1500,
      "ground_speed": 85,
      "vertical_rate": 0,
      "trend": "approaching",
      "threat_level": "warning",
      "is_law_enforcement": true,
      "is_helicopter": true,
      "confidence": "high",
      "aircraft_type": "EC130",
      "registration": "N123HP",
      "lat": 34.0622,
      "lon": -118.2337
    }
  ],
  "count": 1,
  "position": {
    "lat": 34.0522,
    "lon": -118.2437
  },
  "timestamp": "2024-01-15T12:30:00Z"
}
```

### Threat Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `hex` | string | Aircraft ICAO hex code |
| `callsign` | string | Aircraft callsign |
| `category` | string | Threat category (e.g., "Law Enforcement", "Helicopter") |
| `description` | string | Human-readable description |
| `distance_nm` | float | Distance in nautical miles |
| `bearing` | float | Absolute bearing in degrees |
| `relative_bearing` | float | Bearing relative to device heading |
| `direction` | string | Cardinal direction (N, NE, E, SE, S, SW, W, NW) |
| `altitude` | integer | Altitude in feet |
| `ground_speed` | integer | Ground speed in knots |
| `vertical_rate` | integer | Vertical rate in feet/minute |
| `trend` | string | Movement trend: `approaching`, `departing`, `holding`, `unknown` |
| `threat_level` | string | Threat level: `info`, `warning`, `critical` |
| `is_law_enforcement` | boolean | Whether aircraft is identified as law enforcement |
| `is_helicopter` | boolean | Whether aircraft is a helicopter |
| `confidence` | string | Identification confidence level |
| `aircraft_type` | string | Aircraft type code |
| `registration` | string | Aircraft registration |
| `lat` | float | Aircraft latitude |
| `lon` | float | Aircraft longitude |

### Threat Update (Broadcast)

Received when threat data is updated server-side:

```json
{
  "type": "threats",
  "data": [...],
  "count": 3,
  "position": {...},
  "timestamp": "2024-01-15T12:30:00Z"
}
```

### Error Messages

```json
{
  "type": "error",
  "message": "lat and lon are required"
}
```

## Request/Response Pattern

You can also use a request/response pattern for specific queries:

### Request Threats

```json
{
  "type": "request",
  "request_id": "req-123",
  "request_type": "threats",
  "params": {}
}
```

**Response:**

```json
{
  "type": "response",
  "request_id": "req-123",
  "request_type": "threats",
  "data": {
    "threats": [...],
    "count": 3,
    "position": {...}
  }
}
```

### Request Session Info

```json
{
  "type": "request",
  "request_id": "req-456",
  "request_type": "session-info",
  "params": {}
}
```

**Response:**

```json
{
  "type": "response",
  "request_id": "req-456",
  "request_type": "session-info",
  "data": {
    "session_id": "550e8400-e29b-41d4-a716-446655440000",
    "position": {"lat": 34.0522, "lon": -118.2437},
    "heading": 180,
    "radius_nm": 25.0
  }
}
```

## Example: JavaScript Client

```javascript
const ws = new WebSocket('wss://api.skyspy.io/ws/cannonball/');

ws.onopen = () => {
  console.log('Connected to Cannonball');

  // Set detection radius
  ws.send(JSON.stringify({
    type: 'set_radius',
    radius_nm: 15
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);

  switch (data.type) {
    case 'session_started':
      console.log('Session:', data.session_id);
      break;

    case 'threats':
      updateThreatsDisplay(data.data);
      break;

    case 'radius_updated':
      console.log('Radius set to:', data.radius_nm);
      break;

    case 'error':
      console.error('Error:', data.message);
      break;
  }
};

// Send position updates from GPS
function sendPosition(lat, lon, heading) {
  ws.send(JSON.stringify({
    type: 'position_update',
    lat: lat,
    lon: lon,
    heading: heading
  }));
}

// Example: Update position every 5 seconds
navigator.geolocation.watchPosition((position) => {
  sendPosition(
    position.coords.latitude,
    position.coords.longitude,
    position.coords.heading
  );
}, null, { enableHighAccuracy: true });
```

## Disconnection

When disconnecting, the server automatically cleans up cached position data. No explicit disconnect message is required.

## Performance Notes

- Position updates are recommended every 5-30 seconds depending on movement speed
- Threat lists are sorted by distance (closest first) then by threat level
- The server caches position data for 60 seconds
- Threats outside the configured radius are filtered out
