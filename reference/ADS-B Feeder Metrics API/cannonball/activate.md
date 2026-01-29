---
title: "Activate Cannonball Mode"
excerpt: "Activate Cannonball mode for the current user"
hidden: false
---

## POST /api/v1/cannonball/activate/

Activates Cannonball mode for the authenticated user. This enables enhanced threat detection and tracking features.

## Request Body

No request body required.

## Response (Authenticated User)

```json
{
  "status": "activated",
  "user_id": 123
}
```

## Response (Anonymous User)

```json
{
  "status": "activated",
  "message": "Anonymous mode - location tracking limited"
}
```

## Behavior

### Authenticated Users
- User is marked as active in Cannonball system
- Enables personalized threat tracking
- Location updates are persistently stored
- Celery task triggered for background activation

### Anonymous Users
- Limited functionality activated
- Location tracking only via cache (5-minute TTL)
- No persistent session tracking

## Example Request

```bash
curl -X POST "https://api.skyspy.io/api/v1/cannonball/activate/" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Mode activated successfully |
| 401 | Unauthorized - Invalid authentication (still activates in anonymous mode) |

## Usage Notes

- Call this endpoint when the user enters Cannonball mode in your application
- For mobile apps, call on app launch or when user navigates to threat view
- Pair with the deactivate endpoint when user exits the mode
