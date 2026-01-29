---
title: "Acknowledge Alert"
excerpt: "Acknowledge a single Cannonball alert"
hidden: false
---

## POST /api/v1/cannonball/alerts/{id}/acknowledge/

Acknowledges a single alert, marking it as reviewed by the user.

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | Alert ID to acknowledge |

## Request Body

No request body required.

## Response

```json
{
  "id": 1,
  "session": 1,
  "session_icao": "A12345",
  "session_callsign": "N123HP",
  "alert_type": "le_detected",
  "priority": "warning",
  "title": "Law Enforcement Aircraft Detected",
  "message": "LAPD helicopter N123HP detected 2.5nm to the northeast, altitude 1500ft, circling pattern detected.",
  "aircraft_lat": 34.0522,
  "aircraft_lon": -118.2437,
  "aircraft_altitude": 1500,
  "user_lat": 34.0400,
  "user_lon": -118.2500,
  "distance_nm": 2.5,
  "bearing": 45.0,
  "pattern": 1,
  "notified": true,
  "announced": true,
  "acknowledged": true,
  "acknowledged_at": "2024-01-15T12:35:00Z",
  "created_at": "2024-01-15T12:30:00Z"
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Alert ID |
| `session` | integer | Associated session ID |
| `session_icao` | string | Aircraft ICAO hex code |
| `session_callsign` | string | Aircraft callsign |
| `alert_type` | string | Type of alert |
| `priority` | string | Alert priority |
| `title` | string | Alert title |
| `message` | string | Detailed alert message |
| `aircraft_lat` | float | Aircraft latitude at time of alert |
| `aircraft_lon` | float | Aircraft longitude at time of alert |
| `aircraft_altitude` | integer | Aircraft altitude at time of alert |
| `user_lat` | float | User latitude at time of alert |
| `user_lon` | float | User longitude at time of alert |
| `distance_nm` | float | Distance from user at time of alert |
| `bearing` | float | Bearing from user at time of alert |
| `pattern` | integer | Associated pattern ID (if applicable) |
| `notified` | boolean | Whether push notification was sent |
| `announced` | boolean | Whether TTS announcement was made |
| `acknowledged` | boolean | True after acknowledgement |
| `acknowledged_at` | string | Acknowledgement timestamp |
| `created_at` | string | Alert creation timestamp |

## Example Request

```bash
curl -X POST "https://api.skyspy.io/api/v1/cannonball/alerts/1/acknowledge/" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Alert acknowledged successfully |
| 404 | Alert not found |
| 401 | Unauthorized - Invalid or missing authentication |
