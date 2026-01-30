---
title: Map & Aviation Data
hidden: false
---

# ✈️ Map and Aviation Data

SkySpy provides a comprehensive real-time aircraft tracking map with rich aviation data overlays. The map system supports two display modes and integrates multiple aviation data sources for professional-grade situational awareness.

> 📸 **Screenshot Placeholder**: *Main map interface showing live aircraft tracking with weather overlays*

---

## 🗺️ Map Features Overview

### Display Modes

SkySpy offers two distinct map visualization modes optimized for different use cases:

> 🖼️ **Screenshot Placeholder**: *Side-by-side comparison of CRT Mode vs Pro Mode*

[block:callout]
{
  "type": "info",
  "title": "💡 Mode Selection Tip",
  "body": "CRT Mode works great on mobile and touch devices. Pro Mode is recommended for desktop power users who want ATC-style precision."
}
[/block]

[block:parameters]
{
  "data": {
    "h-0": "Mode",
    "h-1": "Description",
    "h-2": "Best For",
    "0-0": "🖥️ **CRT Mode**",
    "0-1": "Classic radar-style display with Leaflet-based interactive maps",
    "0-2": "General use, touch devices",
    "1-0": "🎯 **Pro Mode**",
    "1-1": "Professional ATC-style canvas-based radar display",
    "1-2": "Desktop power users, ATC simulation"
  },
  "cols": 3,
  "rows": 2
}
[/block]

---

## 🎯 Pro Mode Features

[block:callout]
{
  "type": "success",
  "title": "⚡ High-Performance Radar Display",
  "body": "Pro Mode renders aircraft using HTML5 Canvas for smooth 60fps performance even with hundreds of targets."
}
[/block]

> 📸 **Screenshot Placeholder**: *Pro Mode interface showing compass rose, data blocks, and velocity vectors*

### Feature Cards

[block:html]
{
  "html": "<div style=\"display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 16px; margin: 20px 0;\">\n  <div style=\"border: 1px solid #e0e0e0; border-radius: 8px; padding: 16px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);\">\n    <h4 style=\"margin: 0 0 8px 0; color: #00d4ff;\">🎨 Color Themes</h4>\n    <p style=\"margin: 0; color: #a0a0a0; font-size: 14px;\">Classic Cyan, Amber/Gold, Green Phosphor, High Contrast</p>\n  </div>\n  <div style=\"border: 1px solid #e0e0e0; border-radius: 8px; padding: 16px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);\">\n    <h4 style=\"margin: 0 0 8px 0; color: #00d4ff;\">🧭 Compass Rose</h4>\n    <p style=\"margin: 0; color: #a0a0a0; font-size: 14px;\">Toggleable directional reference overlay with cardinal markers</p>\n  </div>\n  <div style=\"border: 1px solid #e0e0e0; border-radius: 8px; padding: 16px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);\">\n    <h4 style=\"margin: 0 0 8px 0; color: #00d4ff;\">📊 Data Blocks</h4>\n    <p style=\"margin: 0; color: #a0a0a0; font-size: 14px;\">Customizable aircraft information callouts</p>\n  </div>\n  <div style=\"border: 1px solid #e0e0e0; border-radius: 8px; padding: 16px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);\">\n    <h4 style=\"margin: 0 0 8px 0; color: #00d4ff;\">➡️ Velocity Vectors</h4>\n    <p style=\"margin: 0; color: #a0a0a0; font-size: 14px;\">Prediction lines showing future aircraft positions</p>\n  </div>\n  <div style=\"border: 1px solid #e0e0e0; border-radius: 8px; padding: 16px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);\">\n    <h4 style=\"margin: 0 0 8px 0; color: #00d4ff;\">🌈 Speed Coloring</h4>\n    <p style=\"margin: 0; color: #a0a0a0; font-size: 14px;\">Visual speed differentiation by color gradient</p>\n  </div>\n  <div style=\"border: 1px solid #e0e0e0; border-radius: 8px; padding: 16px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);\">\n    <h4 style=\"margin: 0 0 8px 0; color: #00d4ff;\">📏 Measurement Tools</h4>\n    <p style=\"margin: 0; color: #a0a0a0; font-size: 14px;\">Distance and bearing between any two points</p>\n  </div>\n  <div style=\"border: 1px solid #e0e0e0; border-radius: 8px; padding: 16px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);\">\n    <h4 style=\"margin: 0 0 8px 0; color: #00d4ff;\">⚠️ Conflict Detection</h4>\n    <p style=\"margin: 0; color: #a0a0a0; font-size: 14px;\">Automatic proximity alerts visualization</p>\n  </div>\n  <div style=\"border: 1px solid #e0e0e0; border-radius: 8px; padding: 16px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);\">\n    <h4 style=\"margin: 0 0 8px 0; color: #00d4ff;\">📐 Grid Controls</h4>\n    <p style=\"margin: 0; color: #a0a0a0; font-size: 14px;\">Adjustable lat/lon grid opacity</p>\n  </div>\n</div>"
}
[/block]

---

## ⌨️ Keyboard Shortcuts (Pro Mode)

[block:callout]
{
  "type": "warning",
  "title": "🎮 Pro Tip",
  "body": "Master these shortcuts to navigate the radar display like a professional controller!"
}
[/block]

| Shortcut | Action | Description |
|:--------:|--------|-------------|
| ![P](https://img.shields.io/badge/P-4A90D9?style=for-the-badge&logoColor=white) | **Toggle Compass Rose** | Show/hide directional overlay |
| ![G](https://img.shields.io/badge/G-4A90D9?style=for-the-badge&logoColor=white) | **Cycle Grid Opacity** | Adjust lat/lon grid visibility |
| ![L](https://img.shields.io/badge/L-4A90D9?style=for-the-badge&logoColor=white) | **Toggle Data Blocks** | Show/hide aircraft info callouts |
| ![V](https://img.shields.io/badge/V-4A90D9?style=for-the-badge&logoColor=white) | **Toggle Velocity Vectors** | Show/hide prediction lines |
| ![S](https://img.shields.io/badge/S-4A90D9?style=for-the-badge&logoColor=white) | **Toggle Speed Coloring** | Enable/disable speed gradients |
| ![A](https://img.shields.io/badge/A-4A90D9?style=for-the-badge&logoColor=white) | **Toggle Altitude Trails** | Show/hide historical tracks |
| ![C](https://img.shields.io/badge/C-4A90D9?style=for-the-badge&logoColor=white) | **Toggle Conflict Viz** | Show/hide proximity alerts |

---

## 🛩️ Aircraft Tracking

### Real-Time Data Flow

```mermaid
flowchart LR
    subgraph Sources["📡 Data Sources"]
        ADS["ADS-B Receiver"]
        MLAT["MLAT Network"]
    end

    subgraph Backend["⚙️ Backend"]
        WS["WebSocket Server"]
        REST["REST API"]
        REDIS["Redis Cache"]
    end

    subgraph Frontend["🖥️ Frontend"]
        MAP["Map Display"]
        LIST["Aircraft List"]
    end

    ADS --> WS
    MLAT --> WS
    WS --> REDIS
    REDIS --> MAP
    REST --> MAP
    MAP --> LIST

    style ADS fill:#2ecc71,stroke:#27ae60
    style WS fill:#3498db,stroke:#2980b9
    style MAP fill:#9b59b6,stroke:#8e44ad
```

### 📡 Position Data Structure

Each aircraft broadcasts comprehensive telemetry data:

```json
{
  "hex": "A12345",
  "flight": "UAL123",
  "type": "B738",
  "lat": 40.7128,
  "lon": -74.0060,
  "alt": 35000,
  "gs": 450,
  "track": 270,
  "vr": 0,
  "squawk": "1234",
  "category": "A3",
  "military": false,
  "emergency": false,
  "distance_nm": 25.3,
  "rssi": -28.5
}
```

### Aircraft Properties Reference

| Property | Type | Description |
|----------|:----:|-------------|
| `hex` | `string` | 🔑 ICAO 24-bit hex identifier |
| `flight` | `string` | ✈️ Callsign or flight number |
| `type` | `string` | 🏷️ ICAO aircraft type code |
| `lat` / `lon` | `float` | 📍 Position coordinates |
| `alt` | `integer` | ⬆️ Barometric altitude (feet) |
| `gs` | `float` | 💨 Ground speed (knots) |
| `track` | `float` | 🧭 Ground track (degrees) |
| `vr` | `integer` | ↕️ Vertical rate (ft/min) |
| `squawk` | `string` | 📻 Transponder squawk code |
| `category` | `string` | 📊 ADS-B emitter category |
| `military` | `boolean` | 🎖️ Military aircraft flag |
| `emergency` | `boolean` | 🚨 Emergency status |
| `distance_nm` | `float` | 📏 Distance from feeder |
| `rssi` | `float` | 📶 Signal strength (dBFS) |

---

## 🚨 Emergency Squawk Codes

[block:callout]
{
  "type": "danger",
  "title": "⚠️ Emergency Detection",
  "body": "SkySpy automatically highlights aircraft transmitting emergency transponder codes with visual and audible alerts."
}
[/block]

| Squawk | Meaning | Visual Display |
|:------:|---------|----------------|
| ![7500](https://img.shields.io/badge/7500-DC3545?style=for-the-badge) | **Hijack** | 🔴 Red highlight, HIJACK badge |
| ![7600](https://img.shields.io/badge/7600-FD7E14?style=for-the-badge) | **Radio Failure** | 🟠 Orange highlight, RADIO badge |
| ![7700](https://img.shields.io/badge/7700-DC3545?style=for-the-badge) | **General Emergency** | 🔴 Red pulsing, EMERGENCY badge |

> 📸 **Screenshot Placeholder**: *Emergency aircraft display with highlighting and badge*

---

## 🎨 Aircraft Visual Categories

### Category Markers

| Category | Marker | Visual Style |
|----------|:------:|--------------|
| ✈️ Civil Aircraft | △ | Standard triangle marker |
| 🎖️ Military Aircraft | △ + MIL | Triangle with badge, distinct color |
| 🚗 Ground Vehicles | ○ | Gray circle for surface movement |
| 🚁 Helicopters | ◯ | Circular marker |

### 🌈 Altitude Color Coding

[block:html]
{
  "html": "<div style=\"display: flex; flex-direction: column; gap: 8px; margin: 16px 0;\">\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; background: linear-gradient(90deg, #00CED1 0%, #00CED1 100%);\">\n    <span style=\"font-weight: bold; color: white; width: 150px;\">FL350+ (35,000+ ft)</span>\n    <span style=\"color: white;\">🔵 Cyan — High altitude cruise</span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; background: linear-gradient(90deg, #32CD32 0%, #32CD32 100%);\">\n    <span style=\"font-weight: bold; color: white; width: 150px;\">FL180-FL350</span>\n    <span style=\"color: white;\">🟢 Green — Medium altitude</span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; background: linear-gradient(90deg, #FFD700 0%, #FFD700 100%);\">\n    <span style=\"font-weight: bold; color: #333; width: 150px;\">0-FL180</span>\n    <span style=\"color: #333;\">🟡 Yellow — Low altitude</span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; background: linear-gradient(90deg, #808080 0%, #808080 100%);\">\n    <span style=\"font-weight: bold; color: white; width: 150px;\">Ground/Unknown</span>\n    <span style=\"color: white;\">⚪ Gray — Surface or no data</span>\n  </div>\n</div>"
}
[/block]

---

## 🗃️ Aviation Data Integration

### Data Sources Overview

```mermaid
flowchart TB
    subgraph External["🌐 External Sources"]
        AWC["Aviation Weather Center"]
        HEXDB["HexDB"]
        OSKY["OpenSky Network"]
        JP["JetPhotos"]
    end

    subgraph SkySpy["🛰️ SkySpy Backend"]
        CACHE["Redis Cache"]
        DB["PostgreSQL"]
        API["REST API"]
    end

    subgraph Data["📊 Data Types"]
        METAR["METARs"]
        PIREP["PIREPs"]
        AIRFRAME["Airframes"]
        PHOTOS["Photos"]
    end

    AWC --> METAR
    AWC --> PIREP
    HEXDB --> AIRFRAME
    OSKY --> AIRFRAME
    JP --> PHOTOS

    METAR --> CACHE
    PIREP --> CACHE
    AIRFRAME --> DB
    PHOTOS --> DB

    CACHE --> API
    DB --> API

    style AWC fill:#e74c3c,stroke:#c0392b
    style HEXDB fill:#3498db,stroke:#2980b9
    style OSKY fill:#2ecc71,stroke:#27ae60
    style JP fill:#9b59b6,stroke:#8e44ad
```

### Data Source Badges

| Source | Status | Data Provided |
|--------|:------:|---------------|
| ![aviationweather.gov](https://img.shields.io/badge/aviationweather.gov-Official-2ecc71?style=flat-square) | ✅ Active | METARs, TAFs, PIREPs, SIGMETs |
| ![HexDB](https://img.shields.io/badge/HexDB-Primary-3498db?style=flat-square) | ✅ Active | Aircraft registrations |
| ![OpenSky](https://img.shields.io/badge/OpenSky-Secondary-9b59b6?style=flat-square) | ✅ Active | Aircraft metadata |
| ![JetPhotos](https://img.shields.io/badge/JetPhotos-Photos-e74c3c?style=flat-square) | ✅ Active | Aircraft images |
| ![Natural Earth](https://img.shields.io/badge/Natural_Earth-GeoJSON-f39c12?style=flat-square) | ✅ Active | Terrain boundaries |

---

## ✈️ Airframe Database

[block:callout]
{
  "type": "info",
  "title": "📷 Photo Caching",
  "body": "Aircraft photos are automatically fetched and cached locally or in S3 with a 24-hour refresh cycle."
}
[/block]

### Aircraft Info Fields

| Field | Description |
|-------|-------------|
| `icao_hex` | 🔑 ICAO 24-bit hex identifier |
| `registration` | 🏷️ Aircraft tail number |
| `type_code` | ✈️ ICAO type designator |
| `type_name` | 📝 Full aircraft type name |
| `manufacturer` | 🏭 Aircraft manufacturer |
| `model` | 📋 Specific model designation |
| `serial_number` | 🔢 Manufacturer serial number |
| `year_built` | 📅 Year of manufacture |
| `operator` | 🏢 Operating airline/company |
| `operator_icao` | 🔤 Operator ICAO code |
| `owner` | 👤 Registered owner |
| `country` | 🌍 Registration country |
| `is_military` | 🎖️ Military aircraft flag |

### Photo Endpoints

| Endpoint | Description |
|----------|-------------|
| `/api/v1/photos/{icao}` | 🖼️ Full-size aircraft photo |
| `/api/v1/photos/{icao}/thumb` | 📷 Thumbnail image |

---

## 🏛️ Airports

> 📸 **Screenshot Placeholder**: *Airport markers with class badges on map*

```json
{
  "icao": "KJFK",
  "name": "John F. Kennedy International Airport",
  "city": "New York",
  "state": "NY",
  "lat": 40.6398,
  "lon": -73.7789,
  "elev": 13,
  "class": "B",
  "rwy_length": 14511
}
```

### Airport Display Features

- 🏷️ Airport markers with class badges (Class B/C/D)
- 📋 Popup with detailed airport information
- 🔗 External links to AirNav and SkyVector
- 📏 Distance and bearing from feeder location

---

## 📍 NAVAIDs

Navigation aids are displayed as diamond markers on the map:

```json
{
  "ident": "JFK",
  "name": "Kennedy",
  "navaid_type": "VOR-DME",
  "lat": 40.6398,
  "lon": -73.7789,
  "frequency": 115.9,
  "channel": "106X"
}
```

---

## 🔘 Map Layers and Customization

### Standard Layer Toggles

[block:html]
{
  "html": "<div style=\"display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 12px; margin: 16px 0;\">\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; border: 1px solid #27ae60; background: rgba(39, 174, 96, 0.1);\">\n    <span style=\"color: #27ae60; margin-right: 8px;\">✅</span>\n    <span><strong>Aircraft</strong> — Default ON</span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; border: 1px solid #95a5a6; background: rgba(149, 165, 166, 0.1);\">\n    <span style=\"color: #95a5a6; margin-right: 8px;\">⬜</span>\n    <span><strong>VORs & NAVAIDs</strong></span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; border: 1px solid #95a5a6; background: rgba(149, 165, 166, 0.1);\">\n    <span style=\"color: #95a5a6; margin-right: 8px;\">⬜</span>\n    <span><strong>Airports</strong></span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; border: 1px solid #95a5a6; background: rgba(149, 165, 166, 0.1);\">\n    <span style=\"color: #95a5a6; margin-right: 8px;\">⬜</span>\n    <span><strong>Airspace</strong></span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; border: 1px solid #95a5a6; background: rgba(149, 165, 166, 0.1);\">\n    <span style=\"color: #95a5a6; margin-right: 8px;\">⬜</span>\n    <span><strong>METARs</strong></span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; border: 1px solid #95a5a6; background: rgba(149, 165, 166, 0.1);\">\n    <span style=\"color: #95a5a6; margin-right: 8px;\">⬜</span>\n    <span><strong>PIREPs</strong></span>\n  </div>\n</div>"
}
[/block]

### Pro Mode Terrain Overlays

| Overlay | Description |
|---------|-------------|
| 🌍 Countries | International boundaries |
| 🗺️ States | US state boundaries |
| 📍 Counties | US county boundaries |
| 💧 Water Bodies | Lakes and rivers |

### Pro Mode Aviation Overlays

| Overlay | Description |
|---------|-------------|
| 🇺🇸 US ARTCC Boundaries | Air Route Traffic Control Center regions |
| ⛽ US Refueling Tracks | Air-to-air refueling tracks |
| 🇬🇧 UK Military Zones | UK military airspace |
| 🔄 EU AWACS Orbits | European AWACS operating areas |
| 🎯 Training Areas | Military training airspace |

### Layer Opacity Controls

Each overlay supports individual opacity adjustment (0-100%):

```javascript
layerOpacities: {
  usArtcc: 0.5,
  usRefueling: 0.5,
  ukMilZones: 0.5,
  water: 0.5
}
```

### Data Block Configuration

| Field | Description | Icon |
|-------|-------------|:----:|
| Callsign | Flight number/callsign | ✈️ |
| Altitude | Current altitude | ⬆️ |
| Speed | Ground speed | 💨 |
| Heading | Track direction | 🧭 |
| Vertical Speed | Climb/descent rate | ↕️ |
| Aircraft Type | Type code | 🏷️ |
| Compact Mode | Condensed display | 📦 |

---

## 🔍 Filtering and Search

### Traffic Filters

[block:parameters]
{
  "data": {
    "h-0": "Filter",
    "h-1": "Options",
    "h-2": "Default",
    "0-0": "🚨 Safety Events Only",
    "0-1": "Show only safety event aircraft",
    "0-2": "⬜ Off",
    "1-0": "🎖️ Military",
    "1-1": "Show military aircraft",
    "1-2": "✅ On",
    "2-0": "✈️ Civil",
    "2-1": "Show civil aircraft",
    "2-2": "✅ On",
    "3-0": "🛩️ GA / Light",
    "3-1": "General aviation aircraft",
    "3-2": "✅ On",
    "4-0": "🛫 Airliners / Heavy",
    "4-1": "Commercial aircraft",
    "4-2": "✅ On",
    "5-0": "☁️ Airborne",
    "5-1": "Aircraft in flight",
    "5-2": "✅ On",
    "6-0": "🛬 On Ground",
    "6-1": "Surface vehicles/aircraft",
    "6-2": "⬜ Off",
    "7-0": "📻 With Squawk",
    "7-1": "Mode A/C transponder",
    "7-2": "✅ On",
    "8-0": "📡 No Squawk (ADS-B)",
    "8-1": "ADS-B only aircraft",
    "8-2": "✅ On"
  },
  "cols": 3,
  "rows": 9
}
[/block]

### Altitude Filter

Set minimum and maximum altitude bounds (0-60,000 ft) to filter aircraft by flight level.

### 🔎 Search Capabilities

The Pro mode search bar supports:

- ✈️ **Callsign** — e.g., "UAL123"
- 📻 **Squawk code** — e.g., "7700"
- 🔑 **ICAO hex** — e.g., "A12345"

---

## 🌐 Geographic Features

### 📏 Range Rings

Concentric circles showing distance from the feeder:

- 🔧 Configurable range (default: 50 nm)
- 📐 Automatic scaling based on zoom level
- 🏷️ Distance labels at each ring

### 🧭 Compass Rose

Directional reference overlay showing:

- 🧭 Cardinal directions (N, E, S, W)
- 📐 30-degree increment marks
- 🧲 Magnetic heading reference

### 📏 Measurement Tools

Pro mode supports distance/bearing measurement:

- 👆 Click two points to measure
- 📏 Shows great-circle distance in nautical miles
- 🧭 Shows magnetic bearing between points

> 📸 **Screenshot Placeholder**: *Measurement tool showing distance between two points*

### Map Bounds

The system tracks aircraft bounding box:

```json
{
  "bounds": {
    "min_lat": 39.5,
    "max_lat": 41.5,
    "min_lon": -75.0,
    "max_lon": -73.0
  },
  "center": {
    "latitude": 40.5,
    "longitude": -74.0
  },
  "aircraft_count": 145
}
```

---

## 🌤️ Weather Data

### METARs

[block:callout]
{
  "type": "info",
  "title": "🌡️ Real-Time Weather",
  "body": "METAR observations are fetched from aviationweather.gov and cached for 2-5 minutes."
}
[/block]

| Field | Description | Icon |
|-------|-------------|:----:|
| `stationId` | ICAO station identifier | 🏛️ |
| `fltCat` | Flight category | 🎨 |
| Temperature | Temp and dewpoint | 🌡️ |
| Wind | Direction, speed, gusts | 💨 |
| Visibility | Statute miles | 👁️ |
| Clouds | Layer coverage and heights | ☁️ |
| Altimeter | Barometric pressure | 📊 |
| Weather | Precipitation/phenomena | 🌧️ |
| Raw METAR | Original encoded report | 📝 |

### 🎨 Flight Category Visualization

[block:html]
{
  "html": "<div style=\"display: flex; flex-direction: column; gap: 8px; margin: 16px 0;\">\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; background: #27ae60;\">\n    <span style=\"font-weight: bold; color: white; width: 80px;\">VFR</span>\n    <span style=\"color: white; flex: 1;\">🟢 Green — Ceiling >3000 ft AGL, Visibility >5 SM</span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; background: #3498db;\">\n    <span style=\"font-weight: bold; color: white; width: 80px;\">MVFR</span>\n    <span style=\"color: white; flex: 1;\">🔵 Blue — Ceiling 1000-3000 ft, Visibility 3-5 SM</span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; background: #e74c3c;\">\n    <span style=\"font-weight: bold; color: white; width: 80px;\">IFR</span>\n    <span style=\"color: white; flex: 1;\">🔴 Red — Ceiling 500-999 ft, Visibility 1-3 SM</span>\n  </div>\n  <div style=\"display: flex; align-items: center; padding: 12px; border-radius: 6px; background: #9b59b6;\">\n    <span style=\"font-weight: bold; color: white; width: 80px;\">LIFR</span>\n    <span style=\"color: white; flex: 1;\">🟣 Magenta — Ceiling <500 ft, Visibility <1 SM</span>\n  </div>\n</div>"
}
[/block]

> 📸 **Screenshot Placeholder**: *Map showing METAR station markers colored by flight category*

### PIREPs (Pilot Reports)

| Field | Description | Icon |
|-------|-------------|:----:|
| `report_type` | UA (routine) or UUA (urgent) | 📋 |
| `location` | Position reference | 📍 |
| `flight_level` | Altitude of report | ⬆️ |
| `aircraft_type` | Reporting aircraft | ✈️ |
| `turbulence_type` | Clear air or convective | 🌀 |
| `turbulence_freq` | Frequency | ⏱️ |
| `icing_type` | Rime, clear, or mixed | ❄️ |
| `icing_intensity` | Light/moderate/severe | 📊 |
| `sky_cover` | Cloud observations | ☁️ |
| `weather` | Precipitation/phenomena | 🌧️ |

### Weather Icon Reference

| Icon | Meaning |
|:----:|---------|
| ☀️ | Clear skies |
| ⛅ | Partly cloudy |
| ☁️ | Overcast |
| 🌧️ | Rain |
| 🌨️ | Snow |
| ⛈️ | Thunderstorms |
| 🌫️ | Fog/Mist |
| 💨 | Strong winds |
| ❄️ | Icing conditions |
| 🌀 | Turbulence |

### Airspace Advisories

Active SIGMETs, AIRMETs, and G-AIRMETs display:

- 📐 Polygon boundaries
- ⚠️ Hazard type (turbulence, icing, convection)
- ⏱️ Valid time range
- ⬆️ Altitude range affected

---

## 🔌 API Endpoints

### Map Data Endpoints

[block:api-header]
{
  "title": "🗺️ Get GeoJSON Aircraft Data"
}
[/block]

```http
GET /api/v1/map/geojson/
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns all aircraft as a GeoJSON FeatureCollection.

**Response:**
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "A12345",
      "geometry": {
        "type": "Point",
        "coordinates": [-74.006, 40.7128]
      },
      "properties": {
        "hex": "A12345",
        "flight": "UAL123",
        "type": "B738",
        "altitude": 35000,
        "speed": 450,
        "track": 270,
        "military": false,
        "emergency": false
      }
    }
  ],
  "metadata": {
    "count": 145,
    "timestamp": "2024-01-15T12:30:00Z",
    "feeder_location": {
      "latitude": 40.7128,
      "longitude": -74.006
    }
  }
}
```

---

[block:api-header]
{
  "title": "📍 Get Map Bounds"
}
[/block]

```http
GET /api/v1/map/bounds/
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns bounding box of current aircraft positions.

**Response:**
```json
{
  "bounds": {
    "min_lat": 39.5,
    "max_lat": 41.5,
    "min_lon": -75.0,
    "max_lon": -73.0
  },
  "center": {
    "latitude": 40.5,
    "longitude": -74.0
  },
  "aircraft_count": 145
}
```

---

[block:api-header]
{
  "title": "🔗 Get Clustered Aircraft"
}
[/block]

```http
GET /api/v1/map/cluster/?zoom=8
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns aircraft clustered by location for dense traffic areas.

**Parameters:**

| Parameter | Type | Description |
|-----------|:----:|-------------|
| `zoom` | `integer` | Map zoom level (affects cluster size) |
| `cluster_distance` | `float` | Custom clustering distance |

---

[block:api-header]
{
  "title": "📡 Get SSE/WebSocket Status"
}
[/block]

```http
GET /api/v1/map/sse/status/
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns real-time streaming service status.

---

### Aviation Data Endpoints

[block:api-header]
{
  "title": "🗺️ Get GeoJSON Overlay Data"
}
[/block]

```http
GET /api/v1/aviation/geojson/{data_type}/?lat={lat}&lon={lon}&radius_nm={radius}
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns GeoJSON data for map overlays.

**Available Data Types:**

| Data Type | Description |
|-----------|-------------|
| `us_artcc` | 🇺🇸 US ARTCC boundaries |
| `us_a2a_refueling` | ⛽ US refueling tracks |
| `uk_mil_awacs` | 🇬🇧 UK military AWACS zones |
| `uk_mil_aar` | ⛽ UK air-to-air refueling areas |
| `de_mil_awacs` | 🇩🇪 German military zones |
| `ift_training_areas` | 🎯 Training airspace |

---

[block:api-header]
{
  "title": "🌤️ Get METARs"
}
[/block]

```http
GET /api/v1/aviation/metars/?lat={lat}&lon={lon}&radius_nm={radius}
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns METAR weather observations.

**Alternative:**
```http
GET /api/v1/aviation/metars/?icao={icao}&hours={hours}
```

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|:----:|:-------:|-------------|
| `lat` | `float` | — | Center latitude |
| `lon` | `float` | — | Center longitude |
| `radius_nm` | `float` | `200` | Search radius (nm) |
| `icao` | `string` | — | Airport ICAO code |
| `hours` | `integer` | `2` | Hours of history |

---

[block:api-header]
{
  "title": "📋 Get PIREPs"
}
[/block]

```http
GET /api/v1/aviation/pireps/?lat={lat}&lon={lon}&radius_nm={radius}&hours={hours}
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns pilot reports.

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|:----:|:-------:|-------------|
| `lat` | `float` | — | Center latitude |
| `lon` | `float` | — | Center longitude |
| `radius_nm` | `float` | `500` | Search radius (nm) |
| `hours` | `integer` | `6` | Time range |

---

[block:api-header]
{
  "title": "⚠️ Get SIGMETs & AIRMETs"
}
[/block]

```http
GET /api/v1/aviation/sigmets/
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns active SIGMETs and AIRMETs.

---

[block:api-header]
{
  "title": "🛫 Get Airports"
}
[/block]

```http
GET /api/v1/aviation/airports/?lat={lat}&lon={lon}&radius_nm={radius}&type={type}&limit={limit}
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns airports within search area.

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|:----:|:-------:|-------------|
| `lat` | `float` | — | Center latitude |
| `lon` | `float` | — | Center longitude |
| `radius_nm` | `float` | `100` | Search radius |
| `type` | `string` | — | Airport type filter |
| `limit` | `integer` | `500` | Max results |

---

[block:api-header]
{
  "title": "📍 Get NAVAIDs"
}
[/block]

```http
GET /api/v1/aviation/navaids/?lat={lat}&lon={lon}&radius_nm={radius}&type={type}
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns navigation aids.

---

### Airframe Data Endpoints

[block:api-header]
{
  "title": "✈️ Get Aircraft Info"
}
[/block]

```http
GET /api/v1/airframes/{icao}/
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns detailed aircraft registration information.

**Response:**
```json
{
  "icao_hex": "A12345",
  "registration": "N123AB",
  "type_code": "B738",
  "type_name": "Boeing 737-800",
  "manufacturer": "Boeing",
  "model": "737-8H4",
  "serial_number": "12345",
  "year_built": 2015,
  "age_years": 9,
  "operator": "United Airlines",
  "operator_icao": "UAL",
  "owner": "Wells Fargo Trust",
  "country": "United States",
  "country_code": "US",
  "is_military": false,
  "photo_url": "/api/v1/photos/A12345",
  "photo_thumbnail_url": "/api/v1/photos/A12345/thumb",
  "photo_photographer": "John Smith",
  "photo_source": "JetPhotos"
}
```

---

[block:api-header]
{
  "title": "📦 Bulk Aircraft Lookup"
}
[/block]

```http
GET /api/v1/airframes/bulk/?icao={icao1},{icao2},{icao3}
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Look up multiple aircraft (max 100) in a single request.

---

[block:api-header]
{
  "title": "🔍 Search Aircraft"
}
[/block]

```http
GET /api/v1/airframes/search/?q={query}&operator={operator}&type={type}&limit={limit}
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Search aircraft database.

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|:----:|:-------:|-------------|
| `q` | `string` | — | Search query |
| `operator` | `string` | — | Filter by operator |
| `type` | `string` | — | Filter by type code |
| `limit` | `integer` | `50` | Max results (max: 500) |

---

[block:api-header]
{
  "title": "🔄 Refresh Aircraft Info"
}
[/block]

```http
POST /api/v1/airframes/{icao}/refresh/
```

![POST](https://img.shields.io/badge/POST-3498db?style=flat-square) Force refresh aircraft info from external sources.

---

[block:api-header]
{
  "title": "📊 Get Cache Statistics"
}
[/block]

```http
GET /api/v1/airframes/cache/stats/
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Returns aircraft info cache statistics.

---

### Photo Endpoints

[block:api-header]
{
  "title": "🖼️ Get Full Photo"
}
[/block]

```http
GET /api/v1/photos/{icao}/
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Serves the full-size cached aircraft photo. Returns 404 if not cached.

---

[block:api-header]
{
  "title": "📷 Get Thumbnail"
}
[/block]

```http
GET /api/v1/photos/{icao}/thumb/
```

![GET](https://img.shields.io/badge/GET-2ecc71?style=flat-square) Serves the thumbnail-size cached aircraft photo.

---

## 🔌 WebSocket Integration

```mermaid
sequenceDiagram
    participant Client as 🖥️ Client
    participant WS as 📡 WebSocket
    participant Redis as 💾 Redis
    participant API as ⚙️ Backend

    Client->>WS: Connect
    WS-->>Client: Connection ACK

    Client->>WS: Subscribe (aircraft)
    WS->>Redis: Register subscriber

    loop Real-time Updates
        API->>Redis: Publish positions
        Redis->>WS: Broadcast
        WS-->>Client: Position update
    end

    Client->>WS: Request (airports)
    WS->>API: Fetch data
    API-->>WS: Airport data
    WS-->>Client: Response
```

### WebSocket Request Types

| Type | Description |
|------|-------------|
| `navaids` | 📍 Request NAVAIDs in viewport |
| `airports` | 🛫 Request airports in viewport |
| `airspace-boundaries` | 📐 Request static airspace |
| `airspaces` | ⚠️ Request active advisories |
| `metars` | 🌤️ Request METAR observations |
| `pireps` | 📋 Request pilot reports |
| `metar` | 🌡️ Request single station METAR |
| `taf` | 📊 Request single station TAF |
| `aircraft-info` | ✈️ Request aircraft info by ICAO |

### Example WebSocket Request

```javascript
wsRequest('airports', {
  lat: 40.7128,
  lon: -74.006,
  radius: 100,
  limit: 50
}, 10000);  // 10 second timeout
```

---

## ⚙️ Configuration

### Environment Variables

```bash
# 📍 Feeder location (required for distance calculations)
FEEDER_LAT=40.7128
FEEDER_LON=-74.006

# 📷 Photo caching
PHOTO_CACHE_ENABLED=true
PHOTO_CACHE_DIR=/var/lib/skyspy/photos
S3_ENABLED=false

# ⏱️ Cache settings
AIRPORT_CACHE_TIMEOUT=300  # 5 minutes
METAR_CACHE_TIMEOUT=120    # 2 minutes
PIREP_CACHE_TIMEOUT=120    # 2 minutes
```

### Frontend Configuration (localStorage)

| Key | Description |
|-----|-------------|
| `adsb-overlays` | 🗺️ Enabled overlay layers |
| `adsb-layer-opacities` | 🎨 Layer opacity settings |
| `adsb-traffic-filters` | 🔍 Traffic filter configuration |
| `adsb-pro-theme` | 🎨 Pro mode color theme |
| `adsb-pro-grid-opacity` | 📐 Grid line opacity |
| `adsb-pro-compass-rose` | 🧭 Compass rose visibility |
| `adsb-pro-datablock-config` | 📊 Data block field configuration |

---

## ⚡ Performance Considerations

### Data Loading Strategy

```mermaid
flowchart LR
    subgraph Strategy["📊 Loading Strategy"]
        VP["Viewport Detection"]
        DB["Debounce (300ms)"]
        PAR["Parallel Requests"]
        CACHE["Server Cache"]
    end

    VP --> DB --> PAR --> CACHE

    style VP fill:#3498db
    style DB fill:#e74c3c
    style PAR fill:#2ecc71
    style CACHE fill:#f39c12
```

1. **📍 Viewport-Based Loading** — Aviation data loads based on current map viewport, not globally
2. **⏱️ Debouncing** — Requests are debounced to prevent excessive API calls during pan/zoom
3. **⚡ Parallel Requests** — Multiple data types are fetched in parallel for faster loading
4. **💾 Caching** — Server-side caching reduces load on external APIs

### 💡 Recommended Practices

[block:callout]
{
  "type": "success",
  "title": "✅ Performance Tips",
  "body": "• Enable only necessary overlays to reduce rendering overhead\n• Use clustering for areas with dense traffic\n• Consider shorter trail lengths in high-traffic scenarios\n• Use Pro mode's canvas rendering for better performance with many aircraft"
}
[/block]

### ⏱️ Timeouts

| Request Type | Timeout |
|--------------|:-------:|
| 🗃️ Database queries | 10 seconds |
| 🌐 External API calls | 20 seconds |
| 📡 WebSocket requests | 10-20 seconds |
