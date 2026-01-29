---
title: "Get Active Sessions"
excerpt: "Get currently active Cannonball tracking sessions"
hidden: false
---

## GET /api/v1/cannonball/sessions/active/

Returns only the currently active tracking sessions. This is a convenience endpoint for quickly retrieving sessions that are still being tracked.

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
  "count": 1
}
```

## Session Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Session ID |
| `icao_hex` | string | Aircraft ICAO hex code |
| `callsign` | string | Aircraft callsign |
| `is_active` | boolean | Always true for this endpoint |
| `threat_level` | string | Current threat level |
| `urgency_score` | float | 0-100 urgency rating |
| `distance_nm` | float | Distance from tracked user in nautical miles |
| `bearing` | float | Bearing from tracked user |
| `closing_speed_kts` | float | Rate of approach in knots |
| `last_seen` | string | ISO 8601 timestamp of last position update |
| `pattern_count` | integer | Number of patterns detected in session |
| `alert_count` | integer | Number of alerts generated in session |

## Example Request

```bash
curl -X GET "https://api.skyspy.io/api/v1/cannonball/sessions/active/" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 401 | Unauthorized - Invalid or missing authentication |

## Usage Notes

- Use this endpoint for real-time dashboards showing current threats
- Sessions become inactive when the aircraft leaves the area or stops transmitting
- Typically paired with the threats endpoint for complete situational awareness
