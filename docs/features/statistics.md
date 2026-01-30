---
title: Statistics & Analytics
---

# 📊 Statistics & Analytics

> **Transform your aircraft tracking data into actionable insights**

SkySpy provides comprehensive statistics and analytics capabilities for tracking aircraft activity, analyzing flight patterns, monitoring system performance, and gamification features. This guide covers all available metrics, real-time streaming, historical analysis, and data export options.

---

## 🎯 Overview

<Tabs>
<Tab title="Architecture">

```mermaid
graph TB
    subgraph "Data Sources"
        ADS[🛩️ ADS-B Receiver]
        ACARS[📡 ACARS Decoder]
    end

    subgraph "Processing Layer"
        CACHE[(🗄️ Redis Cache)]
        DB[(🐘 PostgreSQL)]
        CELERY[⚙️ Celery Workers]
    end

    subgraph "API Layer"
        REST[🌐 REST API]
        WS[🔌 WebSocket]
    end

    subgraph "Frontend"
        DASH[📈 Stats Dashboard]
        GAME[🎮 Gamification View]
    end

    ADS --> DB
    ACARS --> DB
    DB --> CELERY
    CELERY --> CACHE
    CACHE --> REST
    CACHE --> WS
    REST --> DASH
    WS --> DASH
    REST --> GAME
    WS --> GAME
```

</Tab>
<Tab title="Components">

| Component | Purpose | Technology |
|-----------|---------|------------|
| 🌐 **REST API** | Request-response statistics | Django REST Framework |
| 🔌 **WebSocket** | Real-time streaming updates | Django Channels |
| 📈 **Dashboard** | Interactive visualizations | React + Recharts |
| 🗄️ **Cache** | Performance optimization | Redis |

</Tab>
</Tabs>

> 📌 **Pro Tip:** All statistics are cached for performance with configurable TTLs, and most support customizable time ranges.

---

## ✈️ Aircraft Statistics

Core metrics about tracked aircraft within your coverage area.

### 📊 Live Metrics Dashboard

<CardGroup cols={4}>
<Card title="Total Aircraft" icon="plane">
Currently tracked aircraft count with real-time updates
</Card>
<Card title="With Position" icon="map-pin">
Aircraft transmitting valid GPS coordinates
</Card>
<Card title="Military" icon="shield">
Active military aircraft in coverage area
</Card>
<Card title="Emergencies" icon="triangle-exclamation">
Aircraft squawking 7500/7600/7700
</Card>
</CardGroup>

### 📈 Altitude Distribution

| Band | Altitude Range | Icon | Typical Traffic |
|:----:|----------------|:----:|-----------------|
| 🛬 | Ground | `ground` | Taxiing, parked aircraft |
| ⬆️ | Below 10,000 ft | `low` | Departures, arrivals, GA |
| ✈️ | 10,000 - 30,000 ft | `medium` | Climbing, regional flights |
| 🚀 | Above 30,000 ft | `high` | Cruise altitude, long-haul |

<Accordion title="Example Response">

```json
{
  "total": 42,
  "with_position": 38,
  "military": 3,
  "emergency_squawks": [],
  "altitude": {
    "ground": 2,
    "low": 8,
    "medium": 18,
    "high": 14
  }
}
```

</Accordion>

---

## 🏆 Top Aircraft Leaderboards

Real-time leaderboards computed from currently tracked aircraft.

| Rank | 🎯 Closest | 🚀 Fastest | ⬆️ Highest |
|:----:|-----------|-----------|-----------|
| 🥇 | 2.3 nm | 612 kts | 45,000 ft |
| 🥈 | 4.8 nm | 585 kts | 43,500 ft |
| 🥉 | 7.1 nm | 560 kts | 42,000 ft |

<CardGroup cols={3}>
<Card title="🎯 Closest" icon="crosshairs">
**Metric:** Distance (nm)

Nearest aircraft to your receiver location
</Card>
<Card title="🚀 Fastest" icon="gauge-high">
**Metric:** Ground Speed (kts)

Highest velocity aircraft currently tracked
</Card>
<Card title="⬆️ Highest" icon="arrow-up">
**Metric:** Altitude (ft)

Aircraft at highest cruise altitude
</Card>
</CardGroup>

---

## 📈 Flight Patterns

Flight pattern analytics provide insights into traffic patterns over time.

### 🌡️ Busiest Hours Heatmap

Hourly activity distribution for visualization as a heatmap.

```mermaid
xychart-beta
    title "Typical Daily Traffic Pattern"
    x-axis [0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22]
    y-axis "Aircraft Count" 0 --> 60
    bar [5, 3, 2, 8, 25, 42, 48, 52, 50, 45, 35, 15]
```

<Accordion title="🔗 API: GET /api/v1/stats/flight-patterns/busiest-hours">

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `busiest_hours[]` | array | Hourly data (0-23) |
| `peak_hour` | integer | Hour with most activity |
| `peak_aircraft_count` | integer | Max aircraft during peak |
| `quietest_hour` | integer | Hour with least activity |
| `day_night_ratio` | float | Daytime to nighttime ratio |

</Accordion>

### 🛫 Top Routes

Most frequent origin-destination pairs based on ACARS data and callsign analysis.

<CodeGroup>
```bash Request
GET /api/v1/flight-patterns/routes?hours=24&limit=20
```

```json Response
{
  "routes": [
    {
      "origin": "KJFK",
      "destination": "KLAX",
      "count": 45,
      "airlines": ["AAL", "UAL", "DAL"]
    }
  ],
  "total_routes": 156,
  "time_range_hours": 24
}
```
</CodeGroup>

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `hours` | integer | 24 | Time range in hours |
| `limit` | integer | 20 | Maximum routes to return |

### ✈️ Aircraft Types Distribution

<Accordion title="🔗 API: GET /api/v1/flight-patterns/aircraft-types">

**Response includes:**

| Field | Description |
|-------|-------------|
| `type_code` | ICAO type designator (e.g., "B738") |
| `type_name` | Full type name |
| `count` | Session count |
| `unique_aircraft` | Unique ICAO hex codes |
| `military_pct` | Percentage military |
| `avg_duration_min` | Average tracking duration |

</Accordion>

### ⏱️ Flight Duration by Type

<Accordion title="🔗 API: GET /api/v1/flight-patterns/duration-by-type">

```json
{
  "duration_by_type": [
    {
      "type": "B738",
      "avg_minutes": 45.2,
      "min_minutes": 5.0,
      "max_minutes": 180.5,
      "session_count": 234
    }
  ]
}
```

</Accordion>

---

## 🌍 Geographic Statistics

Geographic analytics based on aircraft registration and origin data.

### 🏳️ Countries of Origin

Breakdown by country based on registration prefix.

| Prefix | Country | Example |
|:------:|---------|---------|
| 🇺🇸 N | United States | N12345 |
| 🇬🇧 G- | United Kingdom | G-ABCD |
| 🇩🇪 D- | Germany | D-AIBC |
| 🇫🇷 F- | France | F-GHIJ |
| 🇯🇵 JA | Japan | JA8088 |

<CodeGroup>
```bash Request
GET /api/v1/stats/geographic/countries
```

```json Response
{
  "countries": [
    {
      "country": "United States",
      "country_code": "US",
      "count": 234,
      "military_count": 12,
      "military_pct": 5.1
    }
  ],
  "total_countries": 28
}
```
</CodeGroup>

### ✈️ Airlines/Operators

<Accordion title="🔗 API: GET /api/v1/stats/geographic/operators">

**Response includes:**
- `operator` - Operator name
- `operator_icao` - ICAO airline code
- `aircraft_count` - Unique aircraft
- `session_count` - Total tracking sessions

</Accordion>

### 🛫 Connected Airports

Airports most connected to your coverage area based on ACARS mentions.

<Accordion title="Example Response">

```json
{
  "airports": [
    {
      "icao": "KJFK",
      "iata": "JFK",
      "name": "John F Kennedy International Airport",
      "country": "United States",
      "count": 156
    }
  ]
}
```

</Accordion>

### ⚔️ Military vs Civilian

**Endpoint:** `GET /api/v1/stats/geographic/military-breakdown`

Returns military/civilian split by country with counts and percentages.

---

## 📡 System Performance

### 📶 Tracking Quality Metrics

Quality metrics for your ADS-B reception.

<CardGroup cols={4}>
<Card title="Quality Score" icon="gauge">
**Target:** > 80

Composite score (0-100)
</Card>
<Card title="Update Rate" icon="bolt">
**Target:** > 1.0 Hz

Position updates per second
</Card>
<Card title="Coverage" icon="signal">
**Target:** > 90%

Expected positions received
</Card>
<Card title="RSSI" icon="wave-square">
**Target:** > -20 dB

Average signal strength
</Card>
</CardGroup>

### 🏅 Quality Grades

| Grade | Badge | Completeness | Update Rate |
|-------|:-----:|--------------|-------------|
| **Excellent** | ⭐⭐⭐⭐⭐ | ≥ 90% | ≥ 10/min |
| **Good** | ⭐⭐⭐⭐ | ≥ 70% | ≥ 6/min |
| **Fair** | ⭐⭐⭐ | ≥ 50% | Any |
| **Poor** | ⭐ | < 50% | Any |

<Accordion title="🔗 API: GET /api/v1/stats/tracking-quality/session/{icao_hex}">

```json
{
  "icao_hex": "A1B2C3",
  "callsign": "UAL123",
  "session": {
    "first_seen": "2024-01-15T10:30:00Z",
    "last_seen": "2024-01-15T11:45:00Z",
    "duration_minutes": 75.0,
    "total_positions": 892
  },
  "quality": {
    "grade": "excellent",
    "update_rate_per_min": 11.89,
    "expected_positions": 900,
    "completeness_pct": 99.1
  },
  "gaps": {
    "total_count": 2,
    "total_time_seconds": 45,
    "gap_percentage": 1.0,
    "max_gap_seconds": 30
  },
  "signal": {
    "avg_rssi": -18.5,
    "min_rssi": -25.2,
    "max_rssi": -12.1
  }
}
```

</Accordion>

### 📉 Coverage Gaps Analysis

**Endpoint:** `GET /api/v1/stats/tracking-quality/gaps`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `hours` | integer | 24 | Time range to analyze |
| `limit` | integer | 100 | Max sessions to analyze |

---

## 📡 Antenna Metrics

> 🧪 **Beta Feature** - Advanced antenna performance analytics

### 🎯 Polar Coverage Plot

Reception data by bearing (direction) for antenna pattern visualization.

```mermaid
pie showData
    title "Coverage by Quadrant"
    "North (0-90°)" : 28
    "East (90-180°)" : 24
    "South (180-270°)" : 22
    "West (270-360°)" : 26
```

<Accordion title="Data Structure">

```json
{
  "bearing_data": [
    {
      "bearing_start": 0,
      "bearing_end": 10,
      "count": 1234,
      "max_distance_nm": 185.5,
      "avg_rssi": -18.2
    }
  ],
  "summary": {
    "coverage_pct": 94,
    "total_sightings": 45678,
    "sectors_with_data": 34
  }
}
```

</Accordion>

### 📊 RSSI vs Distance Correlation

Signal strength analysis by distance for antenna performance evaluation.

| Band | Distance | Avg RSSI | Status |
|------|----------|----------|:------:|
| Near | 0-25 nm | -12.5 dB | 🟢 |
| Mid | 25-50 nm | -16.2 dB | 🟢 |
| Far | 50-100 nm | -20.8 dB | 🟡 |
| Extended | 100+ nm | -25.5 dB | 🟠 |

<Accordion title="Full Data Structure">

```json
{
  "scatter_data": [
    {"distance_nm": 50.5, "rssi": -15.2},
    {"distance_nm": 120.3, "rssi": -22.8}
  ],
  "band_statistics": [
    {"band": "0-25nm", "avg_rssi": -12.5, "count": 500},
    {"band": "25-50nm", "avg_rssi": -16.2, "count": 800}
  ],
  "trend_line": {
    "slope": -0.05,
    "intercept": -10.5,
    "interpretation": "Normal signal degradation with distance"
  },
  "overall_statistics": {
    "avg_rssi": -17.5,
    "min_rssi": -28.0,
    "max_rssi": -8.5
  }
}
```

</Accordion>

---

## 🔌 Real-Time Stats Streaming

SkySpy provides WebSocket-based real-time statistics streaming.

### 📡 Connection Flow

```mermaid
sequenceDiagram
    participant Client
    participant WebSocket
    participant Cache
    participant Database

    Client->>WebSocket: Connect to ws://server/ws/stats/
    WebSocket-->>Client: stats.connected (available types)

    Client->>WebSocket: stats.subscribe (flight_patterns)
    WebSocket-->>Client: stats.subscribed

    loop Every 5 seconds
        Cache->>WebSocket: Stats updated
        WebSocket-->>Client: stats.update
    end

    Client->>WebSocket: stats.request (specific query)
    WebSocket->>Cache: Check cache
    alt Cache hit
        Cache-->>WebSocket: Cached data
    else Cache miss
        WebSocket->>Database: Query
        Database-->>WebSocket: Fresh data
        WebSocket->>Cache: Store
    end
    WebSocket-->>Client: stats.response
```

### 📋 Available Stat Types

<Tabs>
<Tab title="✈️ Flight">

| Stat Type | Description |
|-----------|-------------|
| `flight_patterns` | Flight pattern statistics |
| `geographic` | Geographic breakdown |
| `busiest_hours` | Hourly activity |
| `common_aircraft_types` | Aircraft types seen |

</Tab>
<Tab title="🌍 Geographic">

| Stat Type | Description |
|-----------|-------------|
| `countries` | Countries of origin |
| `airlines` | Airline frequency |
| `airports` | Connected airports |

</Tab>
<Tab title="📊 Session">

| Stat Type | Description |
|-----------|-------------|
| `tracking_quality` | Tracking metrics |
| `coverage_gaps` | Coverage gap analysis |
| `engagement` | Engagement statistics |

</Tab>
<Tab title="⏱️ Time">

| Stat Type | Description |
|-----------|-------------|
| `week_comparison` | Week-over-week stats |
| `seasonal_trends` | Monthly/seasonal trends |
| `day_night` | Day vs night traffic |
| `weekend_weekday` | Weekend vs weekday |
| `daily_totals` | Daily time series |
| `weekly_totals` | Weekly time series |
| `monthly_totals` | Monthly time series |

</Tab>
<Tab title="📡 ACARS">

| Stat Type | Description |
|-----------|-------------|
| `acars_stats` | ACARS message stats |
| `acars_trends` | ACARS trends |
| `acars_airlines` | ACARS by airline |

</Tab>
<Tab title="🎮 Gamification">

| Stat Type | Description |
|-----------|-------------|
| `personal_records` | Personal records |
| `rare_sightings` | Notable sightings |
| `collection_stats` | Collection progress |
| `spotted_by_type` | Spotted by type |
| `spotted_by_operator` | Spotted by operator |
| `streaks` | Sighting streaks |
| `daily_stats` | Daily gamification |
| `lifetime_stats` | Lifetime totals |

</Tab>
</Tabs>

### 📨 WebSocket Messages

<CodeGroup>
```json Subscribe
{
  "type": "stats.subscribe",
  "stat_types": ["flight_patterns", "tracking_quality"]
}
```

```json Request Data
{
  "type": "stats.request",
  "stat_type": "flight_patterns",
  "filters": {
    "hours": 24,
    "limit": 50
  },
  "request_id": "req-123"
}
```

```json Set Filters
{
  "type": "stats.set_filters",
  "filters": {
    "hours": 48,
    "aircraft_type": "B738"
  }
}
```

```json Refresh Cache
{
  "type": "stats.refresh",
  "stat_type": "geographic"
}
```
</CodeGroup>

---

## 📜 Historical Data Analysis

### 📈 Trends Over Time

<CodeGroup>
```bash Request
GET /api/v1/history/trends?hours=24&interval=hour
```

```json Response
{
  "intervals": [
    {
      "timestamp": "2024-01-15T10:00:00Z",
      "unique_aircraft": 42,
      "total_positions": 1234,
      "military_count": 3
    }
  ],
  "summary": {
    "total_unique_aircraft": 156,
    "peak_concurrent": 52,
    "total_intervals": 24
  }
}
```
</CodeGroup>

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `hours` | integer | 24 | Time range |
| `interval` | string | "hour" | Grouping (hour/day) |

### 🏆 Top Performers

**Endpoint:** `GET /api/v1/history/top`

| Category | Icon | Description |
|----------|:----:|-------------|
| `longest_tracked` | ⏱️ | Longest session duration |
| `furthest_distance` | 🎯 | Maximum distance from receiver |
| `highest_altitude` | ⬆️ | Maximum altitude reached |
| `closest_approach` | 📍 | Minimum distance to receiver |

<Accordion title="Example Response">

```json
{
  "longest_tracked": [
    {
      "icao_hex": "A1B2C3",
      "callsign": "UAL123",
      "aircraft_type": "B738",
      "duration_min": 185.5,
      "is_military": false
    }
  ],
  "furthest_distance": [...],
  "highest_altitude": [...],
  "closest_approach": [...]
}
```

</Accordion>

### 📏 Distance Analytics

**Endpoint:** `GET /api/v1/history/analytics/distance`

<Accordion title="Response Structure">

```json
{
  "statistics": {
    "mean_nm": 45.2,
    "median_nm": 38.5,
    "max_nm": 245.8,
    "percentile_90": 125.0
  },
  "distribution": {
    "0-25nm": 234,
    "25-50nm": 567,
    "50-100nm": 890,
    "100-150nm": 345,
    "150+nm": 123
  }
}
```

</Accordion>

### 🚀 Speed Analytics

**Endpoint:** `GET /api/v1/history/analytics/speed`

<Accordion title="Response Structure">

```json
{
  "statistics": {
    "mean_kt": 285,
    "max_kt": 612,
    "percentile_90": 480
  },
  "fastest_sessions": [
    {
      "icao_hex": "A1B2C3",
      "callsign": "UAL789",
      "max_speed": 612
    }
  ]
}
```

</Accordion>

### 🔗 Correlation Analysis

**Endpoint:** `GET /api/v1/history/analytics/correlation`

```mermaid
xychart-beta
    title "Altitude vs Speed Correlation"
    x-axis ["0-10k", "10-20k", "20-30k", "30k+"]
    y-axis "Avg Speed (kts)" 0 --> 500
    bar [180, 320, 420, 480]
```

---

## 👥 Engagement Statistics

### 📈 Peak Tracking Periods

<Accordion title="🔗 API: GET /api/v1/stats/engagement/peak-tracking">

```json
{
  "peak_periods": [
    {
      "hour": "2024-01-15T14:00:00Z",
      "unique_aircraft": 52,
      "position_count": 3456,
      "military_count": 4
    }
  ],
  "summary": {
    "avg_aircraft_per_hour": 35.2,
    "max_aircraft_in_hour": 52
  }
}
```

</Accordion>

### 🔄 Return Visitors

Aircraft seen multiple times within the time range.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `hours` | integer | 24 | Time range |
| `min_sessions` | integer | 2 | Minimum session count |
| `limit` | integer | 30 | Results limit |

<Accordion title="Response Example">

```json
{
  "return_visitors": [
    {
      "icao_hex": "A1B2C3",
      "registration": "N12345",
      "session_count": 5,
      "total_positions": 2345,
      "first_session": "2024-01-15T06:00:00Z",
      "last_session": "2024-01-15T18:00:00Z"
    }
  ],
  "stats": {
    "total_unique_aircraft": 156,
    "returning_aircraft": 23,
    "return_rate_pct": 14.7
  }
}
```

</Accordion>

### ⭐ Most Watched Aircraft

**Endpoint:** `GET /api/v1/stats/engagement/most-watched`

---

## 🎮 Gamification Features

> **Level up your aircraft spotting experience!**

SkySpy includes gamification elements to make aircraft spotting more engaging.

### 🏆 Personal Records

Track your best achievements across multiple categories.

<CardGroup cols={4}>
<Card title="🎯 Max Distance" icon="bullseye">
Furthest aircraft tracked

**Unit:** nautical miles
</Card>
<Card title="⬆️ Max Altitude" icon="arrow-up">
Highest aircraft tracked

**Unit:** feet
</Card>
<Card title="🚀 Max Speed" icon="gauge-high">
Fastest aircraft observed

**Unit:** knots
</Card>
<Card title="⏱️ Longest Session" icon="stopwatch">
Longest tracking duration

**Unit:** minutes
</Card>
</CardGroup>

<CardGroup cols={4}>
<Card title="📊 Most Positions" icon="chart-line">
Most positions in single session

**Unit:** count
</Card>
<Card title="📍 Closest Approach" icon="location-dot">
Nearest to receiver

**Unit:** nautical miles
</Card>
<Card title="📈 Max Climb" icon="arrow-trend-up">
Fastest climb rate

**Unit:** ft/min
</Card>
<Card title="📉 Max Descent" icon="arrow-trend-down">
Fastest descent rate

**Unit:** ft/min
</Card>
</CardGroup>

<Accordion title="API Response Example">

```json
{
  "records": [
    {
      "record_type": "max_distance",
      "record_type_display": "Maximum Distance",
      "icao_hex": "A1B2C3",
      "callsign": "UAL123",
      "aircraft_type": "B77W",
      "registration": "N12345",
      "operator": "United Airlines",
      "value": 245.8,
      "achieved_at": "2024-01-15T14:30:00Z",
      "previous_value": 220.5,
      "previous_icao_hex": "B2C3D4"
    }
  ]
}
```

</Accordion>

### 💎 Rare Sightings

Notable and rare aircraft detections.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `hours` | integer | 24 | Time range |
| `limit` | integer | 50 | Maximum results |
| `include_acknowledged` | boolean | false | Include dismissed |

#### 🏅 Rarity Types & Scores

| Type | Icon | Description | Score |
|------|:----:|-------------|:-----:|
| `first_hex` | 🆕 | First time tracking this aircraft | 3 |
| `military` | ⚔️ | Military aircraft | 4 |
| `air_ambulance` | 🚑 | Medical evacuation flights | 5 |
| `law_enforcement` | 🚔 | Police/Coast Guard aircraft | 5-6 |
| `rare_type` | 💎 | Rare aircraft type | 6-10 |
| `test_flight` | 🧪 | Manufacturer test flights | 7 |
| `government` | 🏛️ | Government aircraft (e.g., N1xx) | 9 |

#### ✨ Notable Registration Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| 🇺🇸 `N1xx` | US Government | N100, N175 |
| ✈️ `SAM`, `AF1`, `AF2` | Air Force One/Two | Executive transport |
| 🔧 `N7xx` | Boeing Test | N787BX |
| 🔧 `F-WW*` | Airbus Test | F-WWDD |
| 🚀 Contains "NASA" | NASA Research | NASA941 |

#### 🌟 Ultra-Rare Aircraft Types

| Aircraft | Rarity Score | Description |
|----------|:------------:|-------------|
| Boeing E-4B Nightwatch | ⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐ | "Doomsday Plane" |
| Lockheed U-2 | ⭐⭐⭐⭐⭐⭐⭐⭐⭐ | High-altitude reconnaissance |
| Boeing E-6B Mercury | ⭐⭐⭐⭐⭐⭐⭐⭐⭐ | Airborne command post |
| Boeing B-52 | ⭐⭐⭐⭐⭐⭐⭐⭐ | Strategic bomber |
| Airbus A380 | ⭐⭐⭐⭐⭐⭐⭐ | Superjumbo |
| Boeing 747-8 | ⭐⭐⭐⭐⭐⭐ | Jumbo variant |

### 📦 Collection Progress

**Endpoint:** `GET /api/v1/stats/gamification/collection`

<Accordion title="Response Example">

```json
{
  "total_unique_aircraft": 1234,
  "military_aircraft": 156,
  "unique_types": 89,
  "unique_operators": 234,
  "unique_countries": 45,
  "first_aircraft": {
    "icao_hex": "A1B2C3",
    "registration": "N12345",
    "first_seen": "2023-06-15T10:00:00Z"
  },
  "most_seen": [
    {
      "icao_hex": "B2C3D4",
      "registration": "N54321",
      "operator": "Southwest Airlines",
      "times_seen": 156
    }
  ]
}
```

</Accordion>

### 🔥 Sighting Streaks

Track consecutive day streaks for various categories.

| Streak Type | Icon | Qualification |
|-------------|:----:|---------------|
| `any_sighting` | 📅 | Any aircraft tracked |
| `military` | ⚔️ | At least one military aircraft |
| `unique_new` | 🆕 | A new unique aircraft |
| `high_altitude` | 🚀 | Aircraft at 40,000+ ft |
| `long_range` | 🎯 | Aircraft at 100+ nm |
| `rare_type` | 💎 | A rare aircraft type |

<Accordion title="Response Example">

```json
{
  "streaks": [
    {
      "streak_type": "any_sighting",
      "streak_type_display": "Daily Sighting",
      "current_streak_days": 15,
      "current_streak_start": "2024-01-01",
      "last_qualifying_date": "2024-01-15",
      "best_streak_days": 45,
      "best_streak_start": "2023-10-01",
      "best_streak_end": "2023-11-15"
    }
  ]
}
```

</Accordion>

### 📊 Daily Statistics

**Endpoint:** `GET /api/v1/stats/gamification/daily?days=30`

<Accordion title="Response Example">

```json
{
  "days": [
    {
      "date": "2024-01-15",
      "unique_aircraft": 156,
      "new_aircraft": 12,
      "total_sessions": 234,
      "total_positions": 45678,
      "military_count": 8,
      "max_distance_nm": 185.5,
      "max_altitude": 45000,
      "max_speed": 580,
      "top_types": {"B738": 45, "A320": 38},
      "top_operators": {"United": 25, "American": 22}
    }
  ]
}
```

</Accordion>

### ♾️ Lifetime Statistics

**Endpoint:** `GET /api/v1/stats/gamification/lifetime`

<CardGroup cols={3}>
<Card title="Total Aircraft" icon="plane">
All-time unique aircraft spotted
</Card>
<Card title="Total Sessions" icon="clock">
Cumulative tracking sessions
</Card>
<Card title="Total Positions" icon="map-pin">
Position updates received
</Card>
</CardGroup>

<Accordion title="Full Response">

```json
{
  "total_unique_aircraft": 12345,
  "total_sessions": 56789,
  "total_positions": 2345678,
  "unique_aircraft_types": 156,
  "unique_operators": 456,
  "unique_countries": 78,
  "active_tracking_days": 365,
  "total_rare_sightings": 234,
  "all_time_records": {
    "max_distance": {"value": 285.5, "icao_hex": "A1B2C3"},
    "max_altitude": {"value": 51000, "icao_hex": "B2C3D4"}
  },
  "first_sighting": {
    "icao_hex": "C3D4E5",
    "timestamp": "2023-01-01T10:00:00Z"
  }
}
```

</Accordion>

---

## 🔗 API Endpoints Reference

### 📊 Stats API

| Endpoint | Method | Description |
|----------|:------:|-------------|
| `/api/v1/stats/tracking-quality` | `GET` | 📶 Tracking quality metrics |
| `/api/v1/stats/tracking-quality/gaps` | `GET` | 📉 Coverage gaps analysis |
| `/api/v1/stats/tracking-quality/session/{icao_hex}` | `GET` | 🎯 Session-specific quality |
| `/api/v1/stats/engagement` | `GET` | 👥 Engagement statistics |
| `/api/v1/stats/engagement/most-watched` | `GET` | ⭐ Most favorited aircraft |
| `/api/v1/stats/engagement/return-visitors` | `GET` | 🔄 Returning aircraft |
| `/api/v1/stats/engagement/peak-tracking` | `GET` | 📈 Peak concurrent periods |
| `/api/v1/stats/flight-patterns` | `GET` | ✈️ Flight pattern analytics |
| `/api/v1/stats/flight-patterns/routes` | `GET` | 🛫 Top routes |
| `/api/v1/stats/flight-patterns/busiest-hours` | `GET` | 🌡️ Hourly heatmap data |
| `/api/v1/stats/flight-patterns/aircraft-types` | `GET` | 🛩️ Aircraft types breakdown |
| `/api/v1/stats/flight-patterns/duration-by-type` | `GET` | ⏱️ Duration by type |
| `/api/v1/stats/geographic` | `GET` | 🌍 All geographic stats |
| `/api/v1/stats/geographic/countries` | `GET` | 🏳️ Countries breakdown |
| `/api/v1/stats/geographic/operators` | `GET` | ✈️ Operators frequency |
| `/api/v1/stats/geographic/airports` | `GET` | 🛫 Connected airports |
| `/api/v1/stats/geographic/military-breakdown` | `GET` | ⚔️ Military vs civilian |
| `/api/v1/stats/combined` | `GET` | 📦 All stats combined |
| `/api/v1/stats/combined/summary` | `GET` | 📋 High-level summary |

### 📜 History API

| Endpoint | Method | Description |
|----------|:------:|-------------|
| `/api/v1/history/stats` | `GET` | 📊 Historical statistics |
| `/api/v1/history/trends` | `GET` | 📈 Activity trends |
| `/api/v1/history/top` | `GET` | 🏆 Top performers |
| `/api/v1/history/analytics/distance` | `GET` | 📏 Distance analytics |
| `/api/v1/history/analytics/speed` | `GET` | 🚀 Speed analytics |
| `/api/v1/history/analytics/correlation` | `GET` | 🔗 Correlation analysis |

### ⭐ Favorites API

| Endpoint | Method | Description |
|----------|:------:|-------------|
| `/api/v1/stats/favorites` | `GET` | 📋 List user favorites |
| `/api/v1/stats/favorites/toggle/{icao_hex}` | `POST` | 🔄 Add/remove favorite |
| `/api/v1/stats/favorites/check/{icao_hex}` | `GET` | ✅ Check if favorited |
| `/api/v1/stats/favorites/{id}/notes` | `PATCH` | ✏️ Update notes |

### 🎮 Gamification API

| Endpoint | Method | Description |
|----------|:------:|-------------|
| `/api/v1/stats/gamification/records` | `GET` | 🏆 Personal records |
| `/api/v1/stats/gamification/rare-sightings` | `GET` | 💎 Rare sightings |
| `/api/v1/stats/gamification/collection` | `GET` | 📦 Collection progress |
| `/api/v1/stats/gamification/spotted/types` | `GET` | 🛩️ Spotted by type |
| `/api/v1/stats/gamification/spotted/operators` | `GET` | ✈️ Spotted by operator |
| `/api/v1/stats/gamification/streaks` | `GET` | 🔥 Sighting streaks |
| `/api/v1/stats/gamification/daily` | `GET` | 📅 Daily stats |
| `/api/v1/stats/gamification/lifetime` | `GET` | ♾️ Lifetime totals |

### 🔧 Common Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `hours` | integer | 24 | ⏱️ Time range in hours |
| `refresh` | boolean | false | 🔄 Force cache refresh |
| `limit` | integer | varies | 📊 Maximum results |
| `military_only` | boolean | false | ⚔️ Filter to military only |

---

## 📤 Exporting Data

### 📄 JSON Export

All API endpoints return JSON data that can be exported directly.

<CodeGroup>
```bash Flight Patterns
# Export flight patterns for last 48 hours
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://your-server/api/v1/stats/flight-patterns?hours=48" \
  > flight_patterns.json
```

```bash Gamification
# Export gamification stats
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://your-server/api/v1/stats/gamification/lifetime" \
  > lifetime_stats.json
```

```bash Combined
# Export all stats
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://your-server/api/v1/stats/combined" \
  > all_stats.json
```
</CodeGroup>

### 📊 CSV Export

For tabular data, convert JSON to CSV:

```bash
# Using jq to convert routes to CSV
curl -s -H "Authorization: Bearer YOUR_TOKEN" \
  "https://your-server/api/v1/flight-patterns/routes" | \
  jq -r '.routes[] | [.origin, .destination, .count] | @csv' \
  > routes.csv
```

**Output format:**

```csv
"KJFK","KLAX",45
"KORD","KJFK",38
"KLAX","KSFO",32
```

### ⏰ Scheduled Exports

Use Celery tasks for scheduled data exports:

```python
from skyspy.tasks import export_daily_stats

# Schedule via Celery Beat
CELERY_BEAT_SCHEDULE = {
    'export-daily-stats': {
        'task': 'skyspy.tasks.export_daily_stats',
        'schedule': crontab(hour=0, minute=5),  # 12:05 AM daily
    },
}
```

---

## 💻 Frontend Integration

### 🪝 Using the Stats Hook

The `useStatsData` hook provides access to all statistics data:

```jsx
import { useStatsData } from '../hooks';

function MyStatsComponent({ apiBase, wsRequest, wsConnected }) {
  const data = useStatsData({
    apiBase,
    wsRequest,
    wsConnected,
    filters: {
      timeRange: '24h',
      showMilitaryOnly: false
    }
  });

  const {
    stats,              // Current aircraft stats
    top,                // Top aircraft leaderboards
    histStats,          // Historical stats
    flightPatternsData, // Flight patterns
    geographicData,     // Geographic breakdown
    trackingQualityData,// Quality metrics
    engagementData,     // Engagement stats
    antennaAnalytics,   // Antenna performance
    throughputHistory,  // Message rate history
    aircraftHistory     // Aircraft count history
  } = data;

  return (
    // Your component JSX
  );
}
```

### 🎚️ Filter Options

```javascript
const filters = {
  timeRange: '24h',       // '1h', '6h', '24h', '48h', '7d'
  showMilitaryOnly: false,
  categoryFilter: '',     // 'Commercial', 'GA', 'Military', etc.
  aircraftType: '',       // 'B738', 'A320', etc.
  minAltitude: '',
  maxAltitude: '',
  minDistance: '',
  maxDistance: ''
};
```

### ⏱️ Time Range Mappings

| Display | Hours | Use Case |
|---------|:-----:|----------|
| `1h` | 1 | Real-time monitoring |
| `6h` | 6 | Recent activity |
| `24h` | 24 | Daily overview |
| `48h` | 48 | Extended analysis |
| `7d` | 168 | Weekly trends |

---

## 🗄️ Caching Strategy

Statistics are cached at multiple levels for performance:

| Cache Key | TTL | Category |
|-----------|:---:|----------|
| `flight_patterns_stats` | 5 min | ✈️ Flight |
| `geographic_stats` | 5 min | 🌍 Geographic |
| `tracking_quality_stats` | 5 min | 📶 Quality |
| `engagement_stats` | 5 min | 👥 Engagement |
| `gamification:personal_records` | 5 min | 🏆 Gamification |
| `gamification:rare_sightings` | 2 min | 💎 Gamification |
| `gamification:collection_stats` | 5 min | 📦 Gamification |
| `gamification:streaks` | 10 min | 🔥 Gamification |
| `gamification:daily_stats` | 5 min | 📅 Gamification |
| `gamification:lifetime_stats` | 10 min | ♾️ Gamification |

### 🔄 Forcing Cache Refresh

<CodeGroup>
```bash REST API
GET /api/v1/stats/flight-patterns?refresh=true
```

```json WebSocket
{
  "type": "stats.request",
  "stat_type": "flight_patterns",
  "filters": {"force_refresh": true}
}
```
</CodeGroup>

---

## ✅ Best Practices

<CardGroup cols={2}>
<Card title="🔌 Use WebSocket for Real-Time" icon="bolt">
Subscribe to stat types you need updates for rather than polling REST endpoints.
</Card>
<Card title="🗄️ Cache Appropriately" icon="database">
Don't force-refresh unless necessary. Default cache TTLs are optimized for typical use cases.
</Card>
<Card title="🎯 Filter at the Source" icon="filter">
Use query parameters to filter data server-side rather than fetching everything and filtering client-side.
</Card>
<Card title="📦 Batch Requests" icon="boxes-stacked">
Use the combined endpoints (`/api/v1/stats/combined`) when you need multiple stat types.
</Card>
<Card title="⏱️ Time Range Optimization" icon="clock">
Shorter time ranges (1h, 6h) compute faster. Use 24h or longer for trend analysis.
</Card>
<Card title="⏳ Handle Loading States" icon="spinner">
All data fetches may take time. Show loading indicators while data is being retrieved.
</Card>
<Card title="🔄 Error Handling" icon="triangle-exclamation">
Stats endpoints return 503 if unable to calculate. Implement appropriate retry logic.
</Card>
<Card title="📊 Choose Right Charts" icon="chart-simple">
Use heatmaps for hourly data, line charts for trends, and pie charts for distributions.
</Card>
</CardGroup>

---

> 📚 **Related Documentation**
> - [WebSocket API](/docs/websocket) - Real-time streaming details
> - [Authentication](/docs/authentication) - API token management
> - [Alerts](/docs/alerts) - Set up stat-based alerts
