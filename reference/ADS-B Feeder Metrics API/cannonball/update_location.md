---
title: "Update User Location"
excerpt: "Send GPS location for threat distance calculations"
hidden: false
---

## POST /api/v1/cannonball/location/

Updates the user's GPS location for distance-based threat calculations. Location is stored in cache for real-time threat filtering.

## Request Body

```json
{
  "lat": 34.0522,
  "lon": -118.2437,
  "heading": 180,
  "speed": 65
}
```

## Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `lat` | float | Yes | Latitude (-90 to 90) |
| `lon` | float | Yes | Longitude (-180 to 180) |
| `heading` | float | No | Device heading in degrees (0-360) |
| `speed` | float | No | Device speed in mph |

## Response

```json
{
  "status": "ok",
  "location": {
    "lat": 34.0522,
    "lon": -118.2437
  }
}
```

## Behavior

### Authenticated Users
- Location is stored persistently and associated with user account
- Updates trigger Celery task for background processing
- Enables personalized threat tracking across sessions

### Anonymous Users
- Location is stored in cache with session key
- Cache TTL: 300 seconds (5 minutes)
- Limited tracking functionality

## Example Request

```bash
curl -X POST "https://api.skyspy.io/api/v1/cannonball/location/" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "lat": 34.0522,
    "lon": -118.2437,
    "heading": 180
  }'
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Location updated successfully |
| 400 | Invalid coordinates or request body |
| 401 | Unauthorized - Invalid or missing authentication |

## Usage Notes

- Update location frequently (every 5-30 seconds) for accurate distance calculations
- Include heading for relative bearing calculations in threat responses
- For mobile apps, use high-accuracy GPS mode for best results
