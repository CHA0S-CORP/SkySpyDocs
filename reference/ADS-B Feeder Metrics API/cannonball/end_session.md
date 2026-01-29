---
title: "End Session"
excerpt: "Manually end a Cannonball tracking session"
hidden: false
---

## POST /api/v1/cannonball/sessions/{id}/end/

Manually ends a tracking session. Use this to close sessions that are no longer relevant or to mark threats as cleared.

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | Session ID to end |

## Request Body

No request body required.

## Response

```json
{
  "id": 1,
  "icao_hex": "A12345",
  "callsign": "N123HP",
  "registration": "N123HP",
  "identification_method": "database",
  "identification_reason": "Known LAPD helicopter",
  "operator_name": "Los Angeles Police Department",
  "operator_icao": null,
  "aircraft_type": "EC130",
  "is_active": false,
  "threat_level": "warning",
  "threat_level_display": "Warning",
  "urgency_score": 75.0,
  "last_lat": 34.0522,
  "last_lon": -118.2437,
  "last_altitude": 1500,
  "last_ground_speed": 85,
  "last_track": 270,
  "distance_nm": 2.5,
  "bearing": 45.0,
  "closing_speed_kts": 15.0,
  "first_seen": "2024-01-15T12:00:00Z",
  "last_seen": "2024-01-15T12:30:00Z",
  "session_duration_seconds": 1800,
  "pattern_count": 3,
  "alert_count": 2,
  "position_count": 150,
  "patterns": [
    {
      "id": 1,
      "pattern_type": "circling",
      "confidence": "high",
      "confidence_score": 0.92,
      "detected_at": "2024-01-15T12:15:00Z",
      "duration_seconds": 300
    }
  ],
  "metadata": {}
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Session ID |
| `icao_hex` | string | Aircraft ICAO hex code |
| `callsign` | string | Aircraft callsign |
| `registration` | string | Aircraft registration number |
| `identification_method` | string | How aircraft was identified |
| `identification_reason` | string | Human-readable identification reason |
| `operator_name` | string | Aircraft operator name |
| `operator_icao` | string | Operator ICAO code |
| `aircraft_type` | string | Aircraft type designation |
| `is_active` | boolean | False after ending |
| `threat_level` | string | Threat level at time of ending |
| `threat_level_display` | string | Human-readable threat level |
| `urgency_score` | float | Final urgency score |
| `last_lat` | float | Last known latitude |
| `last_lon` | float | Last known longitude |
| `last_altitude` | integer | Last known altitude in feet |
| `last_ground_speed` | integer | Last known ground speed in knots |
| `last_track` | integer | Last known track in degrees |
| `distance_nm` | float | Last distance from user |
| `bearing` | float | Last bearing from user |
| `closing_speed_kts` | float | Last closing speed |
| `first_seen` | string | Session start timestamp |
| `last_seen` | string | Session end timestamp |
| `session_duration_seconds` | integer | Total session duration |
| `pattern_count` | integer | Total patterns detected |
| `alert_count` | integer | Total alerts generated |
| `position_count` | integer | Total position updates received |
| `patterns` | array | List of patterns detected during session |
| `metadata` | object | Additional session metadata |

## Example Request

```bash
curl -X POST "https://api.skyspy.io/api/v1/cannonball/sessions/1/end/" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Session ended successfully |
| 404 | Session not found |
| 401 | Unauthorized - Invalid or missing authentication |
