---
title: "List Detected Patterns"
excerpt: "List flight patterns detected by Cannonball analysis"
hidden: false
---

## GET /api/v1/cannonball/patterns/

Returns a list of detected flight patterns that may indicate surveillance or enforcement activity.

## Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `active_only` | boolean | Only show ongoing patterns (default: false) |
| `hours` | integer | Time range in hours (default: 24) |
| `pattern_type` | string | Filter by pattern type |
| `confidence` | string | Filter by confidence level |
| `icao_hex` | string | Filter by aircraft ICAO hex |

## Response

```json
{
  "patterns": [
    {
      "id": 1,
      "icao_hex": "A12345",
      "callsign": "N123HP",
      "pattern_type": "circling",
      "confidence": "high",
      "confidence_score": 0.92,
      "center_lat": 34.0522,
      "center_lon": -118.2437,
      "radius_nm": 0.5,
      "pattern_data": {
        "orbit_count": 5,
        "avg_radius_nm": 0.45,
        "heading_changes": 12
      },
      "started_at": "2024-01-15T12:15:00Z",
      "ended_at": null,
      "duration_seconds": 300,
      "detected_at": "2024-01-15T12:15:00Z",
      "session": 1,
      "is_active": true
    }
  ],
  "count": 15,
  "by_type": {
    "circling": 8,
    "loitering": 4,
    "grid_search": 2,
    "speed_trap": 1
  }
}
```

## Pattern Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Pattern ID |
| `icao_hex` | string | Aircraft ICAO hex code |
| `callsign` | string | Aircraft callsign |
| `pattern_type` | string | Type of pattern detected |
| `confidence` | string | Confidence level: `low`, `medium`, `high` |
| `confidence_score` | float | 0.0-1.0 confidence score |
| `center_lat` | float | Center latitude of pattern |
| `center_lon` | float | Center longitude of pattern |
| `radius_nm` | float | Radius in nautical miles (for circular patterns) |
| `pattern_data` | object | Pattern-specific data |
| `started_at` | string | When pattern behavior started |
| `ended_at` | string | When pattern ended (null if ongoing) |
| `duration_seconds` | integer | Pattern duration |
| `detected_at` | string | When pattern was detected |
| `session` | integer | Associated session ID |
| `is_active` | boolean | Whether pattern is still ongoing |

## Pattern Types

| Type | Description |
|------|-------------|
| `circling` | Repeated circular orbits over an area |
| `loitering` | Remaining stationary or slow-moving in a small area |
| `grid_search` | Systematic back-and-forth search pattern |
| `speed_trap` | Low-altitude parallel highway flight |
| `parallel_highway` | Extended parallel highway flight |
| `surveillance` | General surveillance behavior |
| `pursuit` | Active pursuit pattern |

## Example Request

```bash
curl -X GET "https://api.skyspy.io/api/v1/cannonball/patterns/?active_only=true&hours=12" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 401 | Unauthorized - Invalid or missing authentication |
