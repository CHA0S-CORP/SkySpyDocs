---
title: "Known LE Aircraft Database"
excerpt: "List and manage known law enforcement aircraft"
hidden: false
---

## GET /api/v1/cannonball/known-aircraft/

Returns the database of known law enforcement aircraft. This database is used for quick identification of LE aircraft.

## Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | string | Search by ICAO hex, registration, or agency name |
| `agency_type` | string | Filter by agency type |
| `agency_state` | string | Filter by US state (2-letter abbreviation) |
| `verified` | boolean | Filter by verification status |
| `source` | string | Filter by data source |

## Response

```json
{
  "aircraft": [
    {
      "id": 1,
      "icao_hex": "A12345",
      "registration": "N123HP",
      "aircraft_type": "EC130",
      "aircraft_model": "Eurocopter EC130 T2",
      "agency_name": "Los Angeles Police Department",
      "agency_type": "local",
      "agency_state": "CA",
      "agency_city": "Los Angeles",
      "source": "faa",
      "source_url": "https://registry.faa.gov/...",
      "verified": true,
      "verified_at": "2024-01-01T00:00:00Z",
      "times_detected": 150,
      "last_detected": "2024-01-15T12:30:00Z",
      "notes": "LAPD Air Support Division - primary patrol helicopter",
      "created_at": "2023-06-01T00:00:00Z",
      "updated_at": "2024-01-15T12:30:00Z"
    }
  ],
  "count": 1,
  "verified_count": 1
}
```

## Aircraft Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Database ID |
| `icao_hex` | string | Aircraft ICAO hex code |
| `registration` | string | FAA registration number |
| `aircraft_type` | string | Aircraft type code |
| `aircraft_model` | string | Full aircraft model name |
| `agency_name` | string | Law enforcement agency name |
| `agency_type` | string | Agency type classification |
| `agency_state` | string | US state (2-letter code) |
| `agency_city` | string | City of agency |
| `source` | string | Data source |
| `source_url` | string | URL to source documentation |
| `verified` | boolean | Whether entry has been verified |
| `verified_at` | string | Verification timestamp |
| `times_detected` | integer | Number of times detected |
| `last_detected` | string | Last detection timestamp |
| `notes` | string | Additional notes |
| `created_at` | string | Entry creation timestamp |
| `updated_at` | string | Last update timestamp |

## Agency Types

| Type | Description |
|------|-------------|
| `federal` | Federal agencies (FBI, DEA, CBP, etc.) |
| `state` | State police and highway patrol |
| `local` | City and county law enforcement |
| `military` | Military aircraft |
| `unknown` | Unclassified |

## Data Sources

| Source | Description |
|--------|-------------|
| `faa` | FAA Aircraft Registry |
| `opensky` | OpenSky Network database |
| `manual` | Manual entry |
| `community` | Community submission |
| `research` | Research/FOIA requests |

## Example Request

```bash
curl -X GET "https://api.skyspy.io/api/v1/cannonball/known-aircraft/?agency_state=CA&verified=true" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Response Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 401 | Unauthorized - Invalid or missing authentication |

## Additional Endpoints

### Check ICAO

```
GET /api/v1/cannonball/known-aircraft/check/{icao_hex}/
```

Check if a specific ICAO hex is in the known LE database.

**Response (Found):**
```json
{
  "found": true,
  "aircraft": { ... }
}
```

**Response (Not Found):**
```json
{
  "found": false,
  "icao_hex": "A12345"
}
```

### Database Statistics

```
GET /api/v1/cannonball/known-aircraft/stats/
```

Get statistics about the known aircraft database.

**Response:**
```json
{
  "total": 1250,
  "verified": 980,
  "by_agency_type": {
    "federal": 150,
    "state": 320,
    "local": 680,
    "military": 50,
    "unknown": 50
  },
  "by_state": {
    "CA": 180,
    "TX": 150,
    "FL": 120,
    "NY": 100,
    "IL": 80
  }
}
```
