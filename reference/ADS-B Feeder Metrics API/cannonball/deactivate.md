---
title: "Deactivate Cannonball Mode"
excerpt: "Deactivate Cannonball mode for the current user"
hidden: false
---

## DELETE /api/v1/cannonball/activate/

Deactivates Cannonball mode for the authenticated user. This stops active threat tracking for the user.

## Request Body

No request body required.

## Response

```json
{
  "status": "deactivated"
}
```

## Behavior

### Authenticated Users
- User is removed from active Cannonball users list
- Celery task triggered for background cleanup
- Cached location data is cleared

### Anonymous Users
- Returns success response
- No persistent state to clear

## Example Request

```bash
curl -X DELETE "https://api.skyspy.io/api/v1/cannonball/activate/" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Mode deactivated successfully |

## Usage Notes

- Call this endpoint when the user exits Cannonball mode
- Recommended to call on app backgrounding or explicit mode exit
- Helps conserve server resources by cleaning up inactive sessions
