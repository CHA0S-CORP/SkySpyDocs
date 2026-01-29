---
title: "Get Pattern Statistics"
excerpt: "Get statistics about detected flight patterns"
hidden: false
---

## GET /api/v1/cannonball/patterns/stats/

Returns aggregated statistics about detected flight patterns over a specified time range.

## Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `hours` | integer | Time range in hours (default: 24) |

## Response

```json
{
  "total": 45,
  "by_type": {
    "circling": 18,
    "loitering": 12,
    "grid_search": 8,
    "speed_trap": 7
  },
  "by_confidence": {
    "high": 15,
    "medium": 22,
    "low": 8
  },
  "hours": 24
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `total` | integer | Total patterns detected in time range |
| `by_type` | object | Pattern counts by type |
| `by_type.circling` | integer | Circling patterns detected |
| `by_type.loitering` | integer | Loitering patterns detected |
| `by_type.grid_search` | integer | Grid search patterns detected |
| `by_type.speed_trap` | integer | Speed trap patterns detected |
| `by_confidence` | object | Pattern counts by confidence level |
| `by_confidence.high` | integer | High confidence patterns |
| `by_confidence.medium` | integer | Medium confidence patterns |
| `by_confidence.low` | integer | Low confidence patterns |
| `hours` | integer | Time range used for statistics |

## Example Request

```bash
curl -X GET "https://api.skyspy.io/api/v1/cannonball/patterns/stats/?hours=48" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 401 | Unauthorized - Invalid or missing authentication |

## Usage Notes

- Use this endpoint for dashboard widgets and summary views
- Combine with list endpoint for detailed pattern information
- Higher confidence patterns are more reliable indicators of surveillance activity
