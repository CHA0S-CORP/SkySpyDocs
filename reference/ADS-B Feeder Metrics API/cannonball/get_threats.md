---
title: "Get Current Threats"
excerpt: "Returns real-time threat data from pattern analysis"
hidden: false
---

## GET /api/v1/cannonball/threats/

Returns the current list of detected threats from the Cannonball analysis engine. Threats are cached and updated in real-time as aircraft data is processed.

## Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `max_range` | float | Maximum distance in nautical miles to include |
| `threat_level` | string | Filter by threat level: `info`, `warning`, or `critical` |

## Response

```json
{
  "threats": [
    {
      "icao_hex": "A12345",
      "callsign": "N123HP",
      "lat": 34.0522,
      "lon": -118.2437,
      "altitude": 1500,
      "ground_speed": 85,
      "track": 270,
      "distance_nm": 2.5,
      "bearing": 45.0,
      "closing_speed": 15.0,
      "threat_level": "warning",
      "urgency_score": 75.0,
      "is_known_le": true,
      "identification_method": "database",
      "identification_reason": "Known LAPD helicopter",
      "operator_name": "Los Angeles Police Department",
      "agency_name": "LAPD Air Support Division",
      "agency_type": "local",
      "patterns": [
        {
          "type": "circling",
          "confidence": "high",
          "duration_seconds": 300
        }
      ]
    }
  ],
  "count": 1,
  "total_detected": 3,
  "timestamp": "2024-01-15T12:30:00Z"
}
```

## Threat Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `icao_hex` | string | Aircraft ICAO hex code |
| `callsign` | string | Aircraft callsign (may be null) |
| `lat` | float | Current latitude |
| `lon` | float | Current longitude |
| `altitude` | integer | Altitude in feet |
| `ground_speed` | integer | Ground speed in knots |
| `track` | integer | Track/heading in degrees |
| `distance_nm` | float | Distance from user in nautical miles |
| `bearing` | float | Bearing from user in degrees |
| `closing_speed` | float | Rate of approach in knots (negative = departing) |
| `threat_level` | string | Threat classification: `info`, `warning`, `critical` |
| `urgency_score` | float | 0-100 urgency rating |
| `is_known_le` | boolean | Whether aircraft is in known LE database |
| `identification_method` | string | How the aircraft was identified |
| `identification_reason` | string | Human-readable identification reason |
| `operator_name` | string | Aircraft operator name |
| `agency_name` | string | Law enforcement agency name |
| `agency_type` | string | Agency type: `federal`, `state`, `local`, `military` |
| `patterns` | array | Active flight patterns detected |

## Example Request

```bash
curl -X GET "https://api.skyspy.io/api/v1/cannonball/threats/?max_range=10&threat_level=warning" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 401 | Unauthorized - Invalid or missing authentication |
