---
title: "Cannonball Statistics"
excerpt: "Get aggregated statistics for Cannonball detections"
hidden: false
---

## GET /api/v1/cannonball/stats/

Returns aggregated statistics for Cannonball detections over time. Statistics are stored at different time granularities for trend analysis.

## Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `period` | string | Period type: `hourly`, `daily`, `weekly`, `monthly` (default: hourly) |
| `days` | integer | Number of days to include (default: 7) |
| `user_only` | boolean | Only show user-specific stats (default: false) |
| `period_type` | string | Filter by period type |

## Response

```json
{
  "stats": [
    {
      "id": 1,
      "period_type": "hourly",
      "period_start": "2024-01-15T12:00:00Z",
      "period_end": "2024-01-15T13:00:00Z",
      "total_detections": 25,
      "unique_aircraft": 8,
      "critical_alerts": 2,
      "warning_alerts": 5,
      "info_alerts": 12,
      "circling_patterns": 4,
      "loitering_patterns": 2,
      "grid_search_patterns": 1,
      "speed_trap_patterns": 1,
      "top_aircraft": [
        {"icao_hex": "A12345", "count": 5},
        {"icao_hex": "A67890", "count": 3}
      ],
      "top_agencies": [
        {"agency": "LAPD", "count": 8},
        {"agency": "CHP", "count": 4}
      ],
      "created_at": "2024-01-15T13:00:00Z"
    }
  ],
  "count": 168,
  "summary": {
    "total_detections": 450,
    "total_alerts": 125
  }
}
```

## Stats Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Stats record ID |
| `period_type` | string | Time period granularity |
| `period_start` | string | Period start timestamp |
| `period_end` | string | Period end timestamp |
| `total_detections` | integer | Total LE aircraft detections |
| `unique_aircraft` | integer | Unique aircraft detected |
| `critical_alerts` | integer | Critical alerts generated |
| `warning_alerts` | integer | Warning alerts generated |
| `info_alerts` | integer | Info alerts generated |
| `circling_patterns` | integer | Circling patterns detected |
| `loitering_patterns` | integer | Loitering patterns detected |
| `grid_search_patterns` | integer | Grid search patterns detected |
| `speed_trap_patterns` | integer | Speed trap patterns detected |
| `top_aircraft` | array | Most frequently detected aircraft |
| `top_agencies` | array | Most frequently detected agencies |
| `created_at` | string | Stats record creation timestamp |

## Example Request

```bash
curl -X GET "https://api.skyspy.io/api/v1/cannonball/stats/?period=daily&days=30" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 401 | Unauthorized - Invalid or missing authentication |

## Summary Endpoint

### GET /api/v1/cannonball/stats/summary/

Get a current summary of Cannonball activity.

**Response:**

```json
{
  "current": {
    "active_sessions": 3,
    "threats": 5
  },
  "today": {
    "alerts": 25,
    "patterns": 12
  },
  "week": {
    "sessions": 150,
    "alerts": 320
  },
  "timestamp": "2024-01-15T12:30:00Z"
}
```

## Summary Fields

| Field | Type | Description |
|-------|------|-------------|
| `current.active_sessions` | integer | Currently active tracking sessions |
| `current.threats` | integer | Current cached threat count |
| `today.alerts` | integer | Alerts generated today |
| `today.patterns` | integer | Patterns detected today |
| `week.sessions` | integer | Sessions started this week |
| `week.alerts` | integer | Alerts generated this week |
| `timestamp` | string | Current server timestamp |
