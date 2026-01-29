---
title: "List Cannonball Sessions"
excerpt: "List all tracking sessions for potential law enforcement aircraft"
hidden: false
---

## GET /api/v1/cannonball/sessions/

Returns a list of Cannonball tracking sessions. Each session represents a period of tracking for an aircraft identified as potential law enforcement or surveillance.

## Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `active_only` | boolean | Only show active sessions (default: false) |
| `hours` | integer | Filter by hours since last seen |
| `is_active` | boolean | Filter by active status |
| `threat_level` | string | Filter by threat level: `info`, `warning`, `critical` |
| `identification_method` | string | Filter by identification method |

## Response

```json
{
  "sessions": [
    {
      "id": 1,
      "icao_hex": "A12345",
      "callsign": "N123HP",
      "is_active": true,
      "threat_level": "warning",
      "urgency_score": 75.0,
      "distance_nm": 2.5,
      "bearing": 45.0,
      "closing_speed_kts": 15.0,
      "last_seen": "2024-01-15T12:30:00Z",
      "pattern_count": 3,
      "alert_count": 2
    }
  ],
  "count": 1,
  "active_count": 1
}
```

## Session Object Fields (List View)

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Session ID |
| `icao_hex` | string | Aircraft ICAO hex code |
| `callsign` | string | Aircraft callsign |
| `is_active` | boolean | Whether session is currently active |
| `threat_level` | string | Current threat level |
| `urgency_score` | float | 0-100 urgency rating |
| `distance_nm` | float | Distance from tracked user in nautical miles |
| `bearing` | float | Bearing from tracked user |
| `closing_speed_kts` | float | Rate of approach in knots |
| `last_seen` | string | ISO 8601 timestamp of last position update |
| `pattern_count` | integer | Number of patterns detected |
| `alert_count` | integer | Number of alerts generated |

## Example Request

```bash
curl -X GET "https://api.skyspy.io/api/v1/cannonball/sessions/?active_only=true&hours=24" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 401 | Unauthorized - Invalid or missing authentication |
