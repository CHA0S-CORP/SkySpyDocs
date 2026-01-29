---
title: "Acknowledge All Alerts"
excerpt: "Acknowledge all unacknowledged Cannonball alerts"
hidden: false
---

## POST /api/v1/cannonball/alerts/acknowledge-all/

Acknowledges all unacknowledged alerts for the current user. This is useful for clearing alert queues after reviewing them.

## Request Body

No request body required.

## Response

```json
{
  "acknowledged": 15
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `acknowledged` | integer | Number of alerts that were acknowledged |

## Example Request

```bash
curl -X POST "https://api.skyspy.io/api/v1/cannonball/alerts/acknowledge-all/" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Alerts acknowledged successfully |
| 401 | Unauthorized - Invalid or missing authentication |

## Usage Notes

- This endpoint acknowledges all alerts matching the user's filter criteria
- Authenticated users will only acknowledge their own alerts
- The response includes the count of alerts that were actually updated
- Alerts that were already acknowledged are not included in the count
