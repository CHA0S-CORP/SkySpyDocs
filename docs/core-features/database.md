---
title: Database Models
hidden: false
---

# 🗄️ Database Architecture

> **Comprehensive documentation of the SkySpy database schema, including all models, relationships, indexes, and performance considerations.**

---

## 📑 Quick Navigation

| Section | Description |
|---------|-------------|
| [Overview](#-overview) | Database engine & configuration |
| [Entity Relationships](#-entity-relationship-diagram) | Visual schema diagram |
| [Domain Groups](#-domain-model-groups) | Model organization |
| [Model Reference](#-model-reference) | Detailed field documentation |
| [Performance](#-performance-optimization) | Indexes & query tips |
| [Data Retention](#-data-retention-policies) | Archival strategies |
| [Migrations](#-migration-strategy) | Schema management |
| [Monitoring](#-database-monitoring) | Health metrics |
| [Backup & Recovery](#-backup-and-recovery) | Disaster recovery |

---

## 📊 Overview

SkySpy uses **PostgreSQL** as its primary database engine for production deployments, with SQLite supported for development and lightweight installations. The database schema is managed through Django's ORM and migrations system.

### Key Characteristics

| Aspect | Details |
|--------|---------|
| 🐘 **Primary Database** | PostgreSQL 14+ |
| 📦 **Development Database** | SQLite (optional) |
| 🔧 **ORM** | Django 5.x |
| 🔄 **Connection Pooling** | `CONN_MAX_AGE=60` |
| 📋 **Schema Management** | Django Migrations |

### Database Configuration

```python
# Production (PostgreSQL)
DATABASE_URL=postgresql://user:password@host:5432/adsb

# Development (SQLite)
DATABASE_URL=sqlite:///db.sqlite3
```

---

## 🔗 Entity Relationship Diagram

```mermaid
erDiagram
    %% ═══════════════════════════════════════════════════
    %% CORE AIRCRAFT TRACKING
    %% ═══════════════════════════════════════════════════
    AircraftSighting ||--o{ AircraftSession : "belongs to"
    AircraftSession ||--o| SessionTrackingQuality : "has quality metrics"
    AircraftSession ||--o{ PersonalRecord : "sets"
    AircraftSession ||--o{ RareSighting : "triggers"

    AircraftInfo ||--o{ AirframeSourceData : "has source data"

    %% ═══════════════════════════════════════════════════
    %% USER & AUTHENTICATION
    %% ═══════════════════════════════════════════════════
    User ||--o| SkyspyUser : "has profile"
    User ||--o{ UserRole : "has roles"
    Role ||--o{ UserRole : "assigned to"
    Role ||--o{ OIDCClaimMapping : "mapped from"
    User ||--o{ APIKey : "owns"
    User ||--o{ AircraftFavorite : "favorites"

    %% ═══════════════════════════════════════════════════
    %% ALERTS & NOTIFICATIONS
    %% ═══════════════════════════════════════════════════
    AlertRule ||--o{ AlertHistory : "triggers"
    AlertRule ||--o{ AlertSubscription : "has subscribers"
    AlertRule ||--o{ AlertAggregate : "aggregates"
    AlertRule }o--o{ NotificationChannel : "sends to"
    User ||--o{ AlertRule : "owns"

    NotificationChannel ||--o{ NotificationLog : "logs"
    NotificationChannel ||--o{ UserNotificationPreference : "preferences"

    %% ═══════════════════════════════════════════════════
    %% SAFETY & EVENTS
    %% ═══════════════════════════════════════════════════
    SafetyEvent }o--|| AircraftSession : "involves"

    %% ═══════════════════════════════════════════════════
    %% CANNONBALL MODE
    %% ═══════════════════════════════════════════════════
    CannonballSession ||--o{ CannonballPattern : "detects"
    CannonballSession ||--o{ CannonballAlert : "generates"
    CannonballKnownAircraft ||--o{ CannonballSession : "triggers"

    %% ═══════════════════════════════════════════════════
    %% AVIATION DATA
    %% ═══════════════════════════════════════════════════
    CachedAirport }o--o{ CachedPirep : "reports near"
    AirspaceAdvisory }o--o{ AirspaceBoundary : "within"
```

---

## 📁 Domain Model Groups

The database models are organized into the following domain groups:

| Domain | 📊 Tables | Purpose |
|--------|:---------:|---------|
| ✈️ **Aircraft Tracking** | 4 | Position tracking, sessions, airframe data |
| 🔐 **Authentication** | 6 | Users, roles, API keys, OIDC |
| 🔔 **Alerts** | 4 | Rules, history, subscriptions, aggregates |
| 📨 **Notifications** | 5 | Channels, templates, logs, preferences |
| ⚠️ **Safety** | 1 | TCAS events, emergencies |
| 🗺️ **Aviation Data** | 6 | Airports, NOTAMs, airspace, PIREPs |
| 🎙️ **Audio** | 1 | Radio transmissions, transcriptions |
| 📡 **Antenna** | 1 | Performance snapshots |
| ⭐ **Engagement** | 2 | Favorites, quality tracking |
| 🏆 **Gamification** | 9 | Personal records, streaks, stats |
| 🚔 **Cannonball** | 5 | LE detection, patterns, alerts |

---

## 📖 Model Reference

### ✈️ Aircraft Tracking

<details>
<summary><strong>📍 AircraftSighting</strong> - Individual position reports from ADS-B receivers</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `aircraft_sightings` |
| **Primary Key** | `id` (AutoField) |
| **Record Count** | High volume (millions+) |
| **Update Frequency** | Real-time (~1/sec per aircraft) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `timestamp` | 📅 DateTimeField | ❌ | Position report time (auto-generated) |
| `icao_hex` | 🔤 CharField(10) | ❌ | Mode S transponder code (e.g., "A12345") |
| `callsign` | 🔤 CharField(10) | ✅ | Flight callsign (e.g., "UAL123") |
| `squawk` | 🔤 CharField(4) | ✅ | Transponder squawk code |
| `latitude` | 🔢 FloatField | ✅ | WGS84 latitude |
| `longitude` | 🔢 FloatField | ✅ | WGS84 longitude |
| `altitude_baro` | 🔢 IntegerField | ✅ | Barometric altitude in feet |
| `altitude_geom` | 🔢 IntegerField | ✅ | Geometric (GPS) altitude in feet |
| `ground_speed` | 🔢 FloatField | ✅ | Ground speed in knots |
| `track` | 🔢 FloatField | ✅ | Track heading in degrees (0-360) |
| `vertical_rate` | 🔢 IntegerField | ✅ | Vertical rate in ft/min |
| `distance_nm` | 🔢 FloatField | ✅ | Distance from receiver in nautical miles |
| `rssi` | 🔢 FloatField | ✅ | Received signal strength indicator (dBm) |
| `category` | 🔤 CharField(4) | ✅ | ADS-B emitter category |
| `aircraft_type` | 🔤 CharField(10) | ✅ | ICAO aircraft type designator |
| `is_military` | ✓ BooleanField | ❌ | Military aircraft flag |
| `is_emergency` | ✓ BooleanField | ❌ | Emergency squawk active |
| `source` | 🔤 CharField(10) | ❌ | Data source (default: "1090") |

#### Indexes

```mermaid
graph LR
    subgraph "Index Strategy"
        A["idx_sightings_icao_time"] --> B["(icao_hex, timestamp)"]
        C["idx_sighting_timestamp_desc"] --> D["timestamp DESC"]
        E["idx_sighting_distance"] --> F["distance_nm WHERE NOT NULL"]
        G["idx_sighting_military"] --> H["is_military WHERE TRUE"]
        I["idx_sighting_rssi"] --> J["rssi WHERE NOT NULL"]
    end
```

| Index Name | Columns | Type |
|------------|---------|------|
| `idx_sightings_icao_time` | `(icao_hex, timestamp)` | Composite |
| `idx_sighting_timestamp_desc` | `timestamp DESC` | Descending (PostgreSQL) |
| `idx_sighting_distance` | `distance_nm` | Partial (WHERE NOT NULL) |
| `idx_sighting_icao_timestamp` | `(icao_hex, timestamp DESC)` | Composite |
| `idx_sighting_military` | `is_military` | Partial (WHERE TRUE) |
| `idx_sighting_rssi` | `rssi` | Partial (WHERE NOT NULL) |
| `idx_sighting_track` | `track` | Partial (WHERE NOT NULL) |

</details>

---

<details>
<summary><strong>📋 AircraftSession</strong> - Continuous tracking session for an aircraft within range</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `aircraft_sessions` |
| **Primary Key** | `id` (AutoField) |
| **Record Count** | Medium volume (thousands/day) |
| **Update Frequency** | On position update |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `icao_hex` | 🔤 CharField(10) | ❌ | Mode S transponder code |
| `callsign` | 🔤 CharField(10) | ✅ | Last seen callsign |
| `first_seen` | 📅 DateTimeField | ❌ | Session start time |
| `last_seen` | 📅 DateTimeField | ❌ | Last position update |
| `total_positions` | 🔢 IntegerField | ❌ | Position count in session |
| `min_altitude` | 🔢 IntegerField | ✅ | Minimum altitude observed |
| `max_altitude` | 🔢 IntegerField | ✅ | Maximum altitude observed |
| `min_distance_nm` | 🔢 FloatField | ✅ | Closest approach |
| `max_distance_nm` | 🔢 FloatField | ✅ | Maximum distance |
| `max_vertical_rate` | 🔢 IntegerField | ✅ | Maximum vertical rate |
| `min_rssi` | 🔢 FloatField | ✅ | Weakest signal |
| `max_rssi` | 🔢 FloatField | ✅ | Strongest signal |
| `is_military` | ✓ BooleanField | ❌ | Military aircraft flag |
| `category` | 🔤 CharField(4) | ✅ | ADS-B emitter category |
| `aircraft_type` | 🔤 CharField(10) | ✅ | ICAO aircraft type |

#### Indexes

| Index Name | Columns | Type |
|------------|---------|------|
| `idx_sessions_last_seen_icao` | `(last_seen, icao_hex)` | Composite |
| `idx_session_last_seen` | `last_seen DESC` | Descending |
| `idx_session_icao_lastseen` | `(icao_hex, last_seen DESC)` | Composite |

</details>

---

<details>
<summary><strong>🛩️ AircraftInfo</strong> - Cached aircraft information including registration, operator, and photos</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `aircraft_info` |
| **Primary Key** | `id` (AutoField) |
| **Unique Constraint** | `icao_hex` |
| **Record Count** | Tens of thousands |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `icao_hex` | 🔤 CharField(10) | ❌ | Mode S transponder code (unique) |
| `registration` | 🔤 CharField(20) | ✅ | Aircraft registration (e.g., "N12345") |
| `source` | 🔤 CharField(50) | ✅ | Data source |
| `type_code` | 🔤 CharField(10) | ✅ | ICAO type designator |
| `type_name` | 🔤 CharField(100) | ✅ | Full aircraft type name |
| `manufacturer` | 🔤 CharField(100) | ✅ | Aircraft manufacturer |
| `model` | 🔤 CharField(100) | ✅ | Aircraft model |
| `serial_number` | 🔤 CharField(50) | ✅ | Manufacturer serial number |
| `year_built` | 🔢 IntegerField | ✅ | Year of manufacture |
| `first_flight_date` | 🔤 CharField(20) | ✅ | First flight date |
| `delivery_date` | 🔤 CharField(20) | ✅ | Delivery date |
| `airframe_hours` | 🔢 IntegerField | ✅ | Total airframe hours |
| `operator` | 🔤 CharField(100) | ✅ | Current operator name |
| `operator_icao` | 🔤 CharField(4) | ✅ | Operator ICAO code |
| `operator_callsign` | 🔤 CharField(20) | ✅ | Operator radio callsign |
| `owner` | 🔤 CharField(200) | ✅ | Registered owner |
| `city` | 🔤 CharField(100) | ✅ | Registration city |
| `state` | 🔤 CharField(10) | ✅ | Registration state |
| `is_interesting` | ✓ BooleanField | ❌ | Interesting aircraft flag |
| `is_pia` | ✓ BooleanField | ❌ | Privacy ICAO Address |
| `is_ladd` | ✓ BooleanField | ❌ | Limiting Aircraft Data Displayed |
| `country` | 🔤 CharField(100) | ✅ | Registration country |
| `country_code` | 🔤 CharField(3) | ✅ | ISO country code |
| `category` | 🔤 CharField(20) | ✅ | Aircraft category |
| `is_military` | ✓ BooleanField | ❌ | Military aircraft flag |
| `photo_url` | 🔤 CharField(500) | ✅ | Full-size photo URL |
| `photo_thumbnail_url` | 🔤 CharField(500) | ✅ | Thumbnail URL |
| `photo_photographer` | 🔤 CharField(100) | ✅ | Photographer name |
| `photo_source` | 🔤 CharField(50) | ✅ | Photo source |
| `photo_page_link` | 🔤 CharField(500) | ✅ | Photo page link |
| `photo_local_path` | 🔤 CharField(500) | ✅ | Local cached photo path |
| `photo_thumbnail_local_path` | 🔤 CharField(500) | ✅ | Local thumbnail path |
| `extra_data` | 📦 JSONField | ✅ | Additional data as JSON |
| `created_at` | 📅 DateTimeField | ❌ | Record creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update time |
| `fetch_failed` | ✓ BooleanField | ❌ | Data fetch failed flag |

#### Indexes

| Index Name | Columns | Type |
|------------|---------|------|
| `idx_aircraft_info_reg` | `registration` | Single |
| `idx_aircraft_info_operator` | `operator_icao` | Single |

</details>

---

<details>
<summary><strong>📂 AirframeSourceData</strong> - Raw airframe data from each data source preserved separately</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `airframe_source_data` |
| **Primary Key** | `id` (AutoField) |
| **Unique Constraint** | `(aircraft_info, source)` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `aircraft_info_id` | 🔗 ForeignKey | ❌ | Reference to AircraftInfo |
| `source` | 🔤 CharField(20) | ❌ | Data source identifier |
| `raw_data` | 📦 JSONField | ❌ | Complete raw data as JSON |
| `registration` | 🔤 CharField(20) | ✅ | Extracted registration |
| `type_code` | 🔤 CharField(10) | ✅ | Extracted type code |
| `type_name` | 🔤 CharField(100) | ✅ | Extracted type name |
| `manufacturer` | 🔤 CharField(100) | ✅ | Extracted manufacturer |
| `model` | 🔤 CharField(100) | ✅ | Extracted model |
| `serial_number` | 🔤 CharField(50) | ✅ | Extracted serial number |
| `year_built` | 🔢 IntegerField | ✅ | Extracted year |
| `operator` | 🔤 CharField(100) | ✅ | Extracted operator |
| `operator_icao` | 🔤 CharField(4) | ✅ | Extracted operator ICAO |
| `owner` | 🔤 CharField(200) | ✅ | Extracted owner |
| `country` | 🔤 CharField(100) | ✅ | Extracted country |
| `city` | 🔤 CharField(100) | ✅ | Extracted city |
| `state` | 🔤 CharField(10) | ✅ | Extracted state |
| `category` | 🔤 CharField(20) | ✅ | Extracted category |
| `is_military` | ✓ BooleanField | ❌ | Military flag |
| `is_interesting` | ✓ BooleanField | ❌ | Interesting flag |
| `is_pia` | ✓ BooleanField | ❌ | PIA flag |
| `is_ladd` | ✓ BooleanField | ❌ | LADD flag |
| `fetched_at` | 📅 DateTimeField | ❌ | Data fetch time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update time |

#### Source Choices

| Source | Description |
|--------|-------------|
| `faa` | FAA Registry |
| `adsbx` | ADS-B Exchange |
| `tar1090` | tar1090-db |
| `opensky` | OpenSky Network |
| `hexdb` | HexDB API |
| `adsblol` | adsb.lol API |
| `planespotters` | Planespotters API |

</details>

---

### 🔐 Authentication & Authorization

<details>
<summary><strong>👤 SkyspyUser</strong> - Extended user profile with OIDC integration and preferences</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `skyspy_users` |
| **Primary Key** | `id` (AutoField) |
| **Relation** | OneToOne with Django User |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `user_id` | 🔗 OneToOneField | ❌ | Reference to Django User |
| `auth_provider` | 🔤 CharField(20) | ❌ | Authentication provider (local/oidc) |
| `oidc_subject` | 🔤 CharField(255) | ✅ | OIDC subject identifier (unique) |
| `oidc_issuer` | 🔤 CharField(500) | ✅ | OIDC issuer URL |
| `oidc_claims` | 📦 JSONField | ✅ | Cached OIDC claims |
| `display_name` | 🔤 CharField(100) | ✅ | Display name |
| `avatar_url` | 🔗 URLField | ✅ | Avatar URL |
| `last_active` | 📅 DateTimeField | ✅ | Last activity timestamp |
| `last_login_ip` | 🌐 GenericIPAddressField | ✅ | Last login IP |
| `preferences` | 📦 JSONField | ❌ | User preferences JSON |
| `created_at` | 📅 DateTimeField | ❌ | Creation timestamp |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

</details>

---

<details>
<summary><strong>🎭 Role</strong> - Custom role with permission arrays</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `roles` |
| **Primary Key** | `id` (AutoField) |
| **Unique Constraint** | `name` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `name` | 🔤 CharField(50) | ❌ | Role name (unique) |
| `display_name` | 🔤 CharField(100) | ❌ | Display name |
| `description` | 📝 TextField | ✅ | Role description |
| `permissions` | 📦 JSONField | ❌ | Permission array |
| `is_system` | ✓ BooleanField | ❌ | System role flag |
| `priority` | 🔢 IntegerField | ❌ | Display priority |
| `created_at` | 📅 DateTimeField | ❌ | Creation timestamp |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

#### Default Roles

| Role | Priority | Description |
|:----:|:--------:|-------------|
| 👁️ `viewer` | 10 | Read-only access |
| ⚙️ `operator` | 20 | Create/manage own alerts |
| 📊 `analyst` | 30 | Export and transcription access |
| 🔧 `admin` | 40 | Full feature access |
| 👑 `superadmin` | 100 | All permissions |

</details>

---

<details>
<summary><strong>🔗 UserRole</strong> - User-to-role assignment with optional expiration</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `user_roles` |
| **Primary Key** | `id` (AutoField) |
| **Unique Constraint** | `(user, role)` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `user_id` | 🔗 ForeignKey | ❌ | Reference to User |
| `role_id` | 🔗 ForeignKey | ❌ | Reference to Role |
| `expires_at` | 📅 DateTimeField | ✅ | Expiration time |
| `assigned_by_id` | 🔗 ForeignKey | ✅ | Assigning user |
| `created_at` | 📅 DateTimeField | ❌ | Assignment time |

#### Indexes

| Index Name | Columns |
|------------|---------|
| `idx_user_role_expiry` | `(user, expires_at)` |

</details>

---

<details>
<summary><strong>🔑 APIKey</strong> - Programmatic access keys with scope restrictions</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `api_keys` |
| **Primary Key** | `id` (AutoField) |
| **Unique Constraint** | `key_hash` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `user_id` | 🔗 ForeignKey | ❌ | Reference to User |
| `name` | 🔤 CharField(100) | ❌ | Key name |
| `key_hash` | 🔤 CharField(64) | ❌ | SHA-256 hash (unique) |
| `key_prefix` | 🔤 CharField(16) | ❌ | First 10 characters |
| `scopes` | 📦 JSONField | ❌ | Permission scopes |
| `is_active` | ✓ BooleanField | ❌ | Active status |
| `expires_at` | 📅 DateTimeField | ✅ | Expiration time |
| `last_used_at` | 📅 DateTimeField | ✅ | Last usage time |
| `last_used_ip` | 🌐 GenericIPAddressField | ✅ | Last usage IP |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |

</details>

---

<details>
<summary><strong>🚪 FeatureAccess</strong> - Per-feature public/private access configuration</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `feature_access` |
| **Primary Key** | `feature` (CharField) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `feature` | 🔤 CharField(30) | ❌ | Feature ID (primary key) |
| `read_access` | 🔤 CharField(20) | ❌ | Read access level |
| `write_access` | 🔤 CharField(20) | ❌ | Write access level |
| `is_enabled` | ✓ BooleanField | ❌ | Feature enabled flag |
| `settings` | 📦 JSONField | ❌ | Feature-specific settings |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |
| `updated_by_id` | 🔗 ForeignKey | ✅ | Last updater |

#### Access Levels

| Level | Description |
|-------|-------------|
| 🌍 `public` | No authentication required |
| 🔓 `authenticated` | Any logged-in user |
| 🔒 `permission` | Specific permission required |

</details>

---

<details>
<summary><strong>🗺️ OIDCClaimMapping</strong> - Map OIDC claims to roles for automatic role assignment</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `oidc_claim_mappings` |
| **Primary Key** | `id` (AutoField) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `name` | 🔤 CharField(100) | ❌ | Mapping name |
| `claim_name` | 🔤 CharField(100) | ❌ | OIDC claim name |
| `match_type` | 🔤 CharField(20) | ❌ | Match type (exact/contains/regex) |
| `claim_value` | 🔤 CharField(255) | ❌ | Value to match |
| `role_id` | 🔗 ForeignKey | ❌ | Role to assign |
| `priority` | 🔢 IntegerField | ❌ | Processing priority |
| `is_active` | ✓ BooleanField | ❌ | Active status |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

</details>

---

### 🔔 Alerts & Notifications

<details>
<summary><strong>📜 AlertRule</strong> - User-defined alert rules with complex conditions</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `alert_rules` |
| **Primary Key** | `id` (AutoField) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `name` | 🔤 CharField(100) | ❌ | Rule name |
| `rule_type` | 🔤 CharField(30) | ✅ | Rule type identifier |
| `operator` | 🔤 CharField(10) | ❌ | Comparison operator |
| `value` | 🔤 CharField(100) | ✅ | Comparison value |
| `conditions` | 📦 JSONField | ✅ | Complex AND/OR conditions |
| `description` | 🔤 CharField(200) | ✅ | Rule description |
| `enabled` | ✓ BooleanField | ❌ | Enabled status |
| `priority` | 🔤 CharField(20) | ❌ | Priority level |
| `starts_at` | 📅 DateTimeField | ✅ | Start time |
| `expires_at` | 📅 DateTimeField | ✅ | Expiration time |
| `api_url` | 🔤 CharField(500) | ✅ | Webhook URL |
| `cooldown_minutes` | 🔢 IntegerField | ❌ | Cooldown period |
| `last_triggered` | 📅 DateTimeField | ✅ | Last trigger time |
| `use_global_notifications` | ✓ BooleanField | ❌ | Use global config |
| `owner_id` | 🔗 ForeignKey | ✅ | Rule owner |
| `visibility` | 🔤 CharField(20) | ❌ | Visibility level |
| `is_system` | ✓ BooleanField | ❌ | System rule flag |
| `is_shared` | ✓ BooleanField | ❌ | Legacy shared flag |
| `suppression_windows` | 📦 JSONField | ❌ | Suppression windows |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

#### Operator Choices

| Operator | Symbol | Description |
|:--------:|:------:|-------------|
| `eq` | `=` | Equals |
| `neq` | `!=` | Not Equals |
| `lt` | `\<` | Less Than |
| `le` | `\<=` | Less Than or Equal |
| `gt` | `\>` | Greater Than |
| `ge` | `\>=` | Greater Than or Equal |
| `contains` | `*...*` | Contains |
| `startswith` | `^...` | Starts With |
| `endswith` | `...$` | Ends With |
| `regex` | `/.../` | Regex Match |

#### Indexes

| Index Name | Columns |
|------------|---------|
| `idx_alert_rules_type` | `(rule_type, enabled)` |
| `idx_alert_rules_vis` | `(visibility, enabled)` |
| `idx_alert_rules_owner` | `(owner, enabled)` |

</details>

---

<details>
<summary><strong>📊 AlertHistory</strong> - History of triggered alerts</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `alert_history` |
| **Primary Key** | `id` (AutoField) |
| **Volume** | High (depends on rule count) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `rule_id` | 🔗 ForeignKey | ✅ | Reference to AlertRule |
| `rule_name` | 🔤 CharField(100) | ✅ | Rule name at trigger time |
| `icao_hex` | 🔤 CharField(10) | ✅ | Triggering aircraft |
| `callsign` | 🔤 CharField(10) | ✅ | Aircraft callsign |
| `message` | 📝 TextField | ✅ | Alert message |
| `priority` | 🔤 CharField(20) | ✅ | Alert priority |
| `aircraft_data` | 📦 JSONField | ✅ | Aircraft snapshot |
| `triggered_at` | 📅 DateTimeField | ❌ | Trigger timestamp |
| `user_id` | 🔗 ForeignKey | ✅ | Rule owner |
| `session_key` | 🔤 CharField(40) | ✅ | Anonymous session key |
| `acknowledged` | ✓ BooleanField | ❌ | Acknowledged status |
| `acknowledged_by_id` | 🔗 ForeignKey | ✅ | Acknowledging user |
| `acknowledged_at` | 📅 DateTimeField | ✅ | Acknowledgment time |

#### Indexes

| Index Name | Columns |
|------------|---------|
| `idx_alert_hist_user` | `(user, triggered_at)` |
| `idx_alert_hist_ack` | `(acknowledged, triggered_at)` |
| `idx_alert_history_triggered` | `triggered_at DESC` |

</details>

---

<details>
<summary><strong>📬 AlertSubscription</strong> - User subscriptions to shared/public alert rules</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `alert_subscriptions` |
| **Unique Constraints** | `(user, rule)`, `(session_key, rule)` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `user_id` | 🔗 ForeignKey | ✅ | Subscribing user |
| `session_key` | 🔤 CharField(40) | ✅ | Anonymous session key |
| `rule_id` | 🔗 ForeignKey | ❌ | Subscribed rule |
| `notify_on_trigger` | ✓ BooleanField | ❌ | Notification enabled |
| `created_at` | 📅 DateTimeField | ❌ | Subscription time |

</details>

---

<details>
<summary><strong>📈 AlertAggregate</strong> - Aggregated alert statistics for time windows</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `alert_aggregates` |
| **Primary Key** | `id` (AutoField) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `rule_id` | 🔗 ForeignKey | ❌ | Reference to AlertRule |
| `window_start` | 📅 DateTimeField | ❌ | Window start time |
| `window_end` | 📅 DateTimeField | ❌ | Window end time |
| `trigger_count` | 🔢 IntegerField | ❌ | Number of triggers |
| `unique_aircraft` | 🔢 IntegerField | ❌ | Unique aircraft count |
| `sample_aircraft` | 📦 JSONField | ❌ | Sample aircraft list |

</details>

---

### 📨 Notification System

<details>
<summary><strong>⚙️ NotificationConfig</strong> - Singleton notification configuration</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `notification_config` |
| **Primary Key** | `id` (always 1) |
| **Pattern** | Singleton |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key (always 1) |
| `apprise_urls` | 📝 TextField | ❌ | Apprise URL configuration |
| `cooldown_seconds` | 🔢 IntegerField | ❌ | Default cooldown |
| `enabled` | ✓ BooleanField | ❌ | Global enabled flag |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

</details>

---

<details>
<summary><strong>📡 NotificationChannel</strong> - Reusable notification channel definitions</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `notification_channels` |
| **Primary Key** | `id` (AutoField) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `name` | 🔤 CharField(100) | ❌ | Channel name |
| `channel_type` | 🔤 CharField(30) | ❌ | Channel type |
| `apprise_url` | 📝 TextField | ❌ | Apprise URL |
| `description` | 🔤 CharField(200) | ✅ | Description |
| `supports_rich` | ✓ BooleanField | ❌ | Rich formatting support |
| `is_global` | ✓ BooleanField | ❌ | Global availability |
| `owner_id` | 🔗 ForeignKey | ✅ | Channel owner |
| `enabled` | ✓ BooleanField | ❌ | Enabled status |
| `verified` | ✓ BooleanField | ❌ | Verified status |
| `last_success` | 📅 DateTimeField | ✅ | Last success time |
| `last_failure` | 📅 DateTimeField | ✅ | Last failure time |
| `last_error` | 📝 TextField | ✅ | Last error message |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

#### Channel Types

| Type | Icon | Description |
|------|:----:|-------------|
| `discord` | 💬 | Discord |
| `slack` | 📱 | Slack |
| `pushover` | 📲 | Pushover |
| `telegram` | ✈️ | Telegram |
| `email` | 📧 | Email |
| `webhook` | 🔗 | Generic Webhook |
| `ntfy` | 🔔 | ntfy |
| `gotify` | 📣 | Gotify |
| `home_assistant` | 🏠 | Home Assistant |
| `twilio` | 📞 | Twilio SMS |
| `custom` | 🔧 | Custom Apprise URL |

</details>

---

<details>
<summary><strong>📝 NotificationTemplate</strong> - Message templates with variable substitution</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `notification_templates` |
| **Unique Constraint** | `name` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `name` | 🔤 CharField(100) | ❌ | Template name (unique) |
| `description` | 🔤 CharField(200) | ✅ | Description |
| `title_template` | 🔤 CharField(200) | ❌ | Title template |
| `body_template` | 📝 TextField | ❌ | Body template |
| `discord_embed` | 📦 JSONField | ✅ | Discord embed JSON |
| `slack_blocks` | 📦 JSONField | ✅ | Slack Block Kit JSON |
| `event_type` | 🔤 CharField(30) | ✅ | Matching event type |
| `priority` | 🔤 CharField(20) | ✅ | Matching priority |
| `is_default` | ✓ BooleanField | ❌ | Default template flag |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

</details>

---

<details>
<summary><strong>📋 NotificationLog</strong> - Log of sent notifications with retry tracking</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `notification_logs` |
| **Primary Key** | `id` (AutoField) |
| **Volume** | High |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `timestamp` | 📅 DateTimeField | ❌ | Log timestamp |
| `notification_type` | 🔤 CharField(50) | ✅ | Notification type |
| `icao_hex` | 🔤 CharField(10) | ✅ | Aircraft ICAO |
| `callsign` | 🔤 CharField(10) | ✅ | Aircraft callsign |
| `message` | 📝 TextField | ✅ | Notification message |
| `details` | 📦 JSONField | ✅ | Additional details |
| `channel_id` | 🔗 ForeignKey | ✅ | Channel reference |
| `channel_url` | 📝 TextField | ✅ | Actual URL used |
| `status` | 🔤 CharField(20) | ❌ | Status (pending/sent/failed/retrying) |
| `retry_count` | 🔢 IntegerField | ❌ | Retry attempt count |
| `max_retries` | 🔢 IntegerField | ❌ | Maximum retries |
| `next_retry_at` | 📅 DateTimeField | ✅ | Next retry time |
| `last_error` | 📝 TextField | ✅ | Last error message |
| `sent_at` | 📅 DateTimeField | ✅ | Send time |
| `duration_ms` | 🔢 IntegerField | ✅ | Send duration |

</details>

---

<details>
<summary><strong>🔕 UserNotificationPreference</strong> - Per-user notification preferences</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `user_notification_preferences` |
| **Unique Constraint** | `(user, channel)` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `user_id` | 🔗 ForeignKey | ❌ | User reference |
| `channel_id` | 🔗 ForeignKey | ❌ | Channel reference |
| `min_priority` | 🔤 CharField(20) | ❌ | Minimum priority |
| `event_types` | 📦 JSONField | ❌ | Event type filter |
| `quiet_hours_start` | ⏰ TimeField | ✅ | Quiet hours start |
| `quiet_hours_end` | ⏰ TimeField | ✅ | Quiet hours end |
| `critical_overrides_quiet` | ✓ BooleanField | ❌ | Critical override flag |
| `timezone` | 🔤 CharField(50) | ❌ | User timezone |
| `enabled` | ✓ BooleanField | ❌ | Enabled status |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

</details>

---

### ⚠️ Safety Events

<details>
<summary><strong>🚨 SafetyEvent</strong> - TCAS conflicts and dangerous flight parameters</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `safety_events` |
| **Primary Key** | `id` (AutoField) |
| **Criticality** | High |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `timestamp` | 📅 DateTimeField | ❌ | Event timestamp |
| `event_type` | 🔤 CharField(50) | ❌ | Event type |
| `severity` | 🔤 CharField(20) | ❌ | Severity level |
| `icao_hex` | 🔤 CharField(10) | ❌ | Primary aircraft |
| `icao_hex_2` | 🔤 CharField(10) | ✅ | Secondary aircraft |
| `callsign` | 🔤 CharField(10) | ✅ | Primary callsign |
| `callsign_2` | 🔤 CharField(10) | ✅ | Secondary callsign |
| `message` | 📝 TextField | ✅ | Event message |
| `details` | 📦 JSONField | ✅ | Event details |
| `aircraft_snapshot` | 📦 JSONField | ✅ | Primary telemetry snapshot |
| `aircraft_snapshot_2` | 📦 JSONField | ✅ | Secondary telemetry snapshot |
| `acknowledged` | ✓ BooleanField | ❌ | Acknowledged status |
| `acknowledged_at` | 📅 DateTimeField | ✅ | Acknowledgment time |

#### Event Types

| Type | Severity | Description |
|------|:--------:|-------------|
| `tcas_ra` | 🔴 Critical | TCAS Resolution Advisory |
| `tcas_ta` | 🟠 High | TCAS Traffic Advisory |
| `extreme_vs` | 🟠 High | Extreme Vertical Speed |
| `vs_reversal` | 🟡 Medium | Vertical Speed Reversal |
| `proximity_conflict` | 🔴 Critical | Proximity Conflict |
| `emergency_squawk` | 🔴 Critical | Emergency Squawk |
| `7500` | 🔴 Critical | Squawk 7500 (Hijack) |
| `7600` | 🟠 High | Squawk 7600 (Radio Failure) |
| `7700` | 🔴 Critical | Squawk 7700 (Emergency) |

#### Indexes

| Index Name | Columns |
|------------|---------|
| `idx_safety_events_type_time` | `(event_type, timestamp)` |
| `idx_safety_event_timestamp` | `timestamp DESC` |

</details>

---

### 🗺️ Aviation Data

<details>
<summary><strong>🛫 CachedAirport</strong> - Cached airport data from Aviation Weather Center</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `cached_airports` |
| **Unique Constraint** | `icao_id` |
| **Refresh Rate** | 30 days |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `fetched_at` | 📅 DateTimeField | ❌ | Fetch timestamp |
| `icao_id` | 🔤 CharField(4) | ❌ | ICAO identifier (unique) |
| `name` | 🔤 CharField(200) | ✅ | Airport name |
| `latitude` | 🔢 FloatField | ❌ | WGS84 latitude |
| `longitude` | 🔢 FloatField | ❌ | WGS84 longitude |
| `elevation_ft` | 🔢 IntegerField | ✅ | Field elevation |
| `airport_type` | 🔤 CharField(50) | ✅ | Airport type |
| `country` | 🔤 CharField(100) | ✅ | Country name |
| `region` | 🔤 CharField(100) | ✅ | Region/state |
| `source_data` | 📦 JSONField | ✅ | Raw API response |

</details>

---

<details>
<summary><strong>📡 CachedNavaid</strong> - Cached navigation aid data</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `cached_navaids` |
| **Primary Key** | `id` (AutoField) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `fetched_at` | 📅 DateTimeField | ❌ | Fetch timestamp |
| `ident` | 🔤 CharField(10) | ❌ | Navaid identifier |
| `name` | 🔤 CharField(100) | ✅ | Navaid name |
| `navaid_type` | 🔤 CharField(20) | ✅ | Type (VOR, NDB, etc.) |
| `latitude` | 🔢 FloatField | ❌ | WGS84 latitude |
| `longitude` | 🔢 FloatField | ❌ | WGS84 longitude |
| `frequency` | 🔢 FloatField | ✅ | Frequency (MHz/kHz) |
| `channel` | 🔤 CharField(10) | ✅ | TACAN channel |
| `source_data` | 📦 JSONField | ✅ | Raw API response |

</details>

---

<details>
<summary><strong>📋 CachedNotam</strong> - Cached NOTAM and TFR data</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `cached_notams` |
| **Unique Constraint** | `notam_id` |
| **Archive Support** | Yes |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `notam_id` | 🔤 CharField(50) | ❌ | NOTAM identifier (unique) |
| `notam_type` | 🔤 CharField(10) | ❌ | NOTAM type |
| `classification` | 🔤 CharField(20) | ✅ | Classification |
| `location` | 🔤 CharField(10) | ❌ | ICAO location |
| `latitude` | 🔢 FloatField | ✅ | Center latitude |
| `longitude` | 🔢 FloatField | ✅ | Center longitude |
| `radius_nm` | 🔢 FloatField | ✅ | TFR radius |
| `floor_ft` | 🔢 IntegerField | ✅ | Floor altitude |
| `ceiling_ft` | 🔢 IntegerField | ✅ | Ceiling altitude |
| `effective_start` | 📅 DateTimeField | ❌ | Start time |
| `effective_end` | 📅 DateTimeField | ✅ | End time |
| `is_permanent` | ✓ BooleanField | ❌ | Permanent flag |
| `text` | 📝 TextField | ❌ | NOTAM text |
| `raw_text` | 📝 TextField | ✅ | Raw NOTAM text |
| `keywords` | 📦 JSONField | ✅ | Extracted keywords |
| `geometry` | 📦 JSONField | ✅ | TFR GeoJSON boundary |
| `reason` | 🔤 CharField(200) | ✅ | TFR reason |
| `source_data` | 📦 JSONField | ✅ | Raw API response |
| `fetched_at` | 📅 DateTimeField | ❌ | Fetch timestamp |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `is_archived` | ✓ BooleanField | ❌ | Archived status |
| `archived_at` | 📅 DateTimeField | ✅ | Archive time |
| `archive_reason` | 🔤 CharField(50) | ✅ | Archive reason |

#### Indexes

| Index Name | Columns |
|------------|---------|
| `idx_notam_loc_start` | `(location, effective_start)` |
| `idx_notam_type_start` | `(notam_type, effective_start)` |
| `idx_notam_location` | `(latitude, longitude)` |
| `idx_notam_validity` | `(effective_end, effective_start)` |
| `idx_notam_archive` | `(is_archived, archived_at)` |

</details>

---

<details>
<summary><strong>🌤️ AirspaceAdvisory</strong> - Active airspace advisories (G-AIRMETs, SIGMETs)</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `airspace_advisories` |
| **Primary Key** | `id` (AutoField) |
| **Refresh Rate** | Hourly |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `fetched_at` | 📅 DateTimeField | ❌ | Fetch timestamp |
| `advisory_id` | 🔤 CharField(50) | ❌ | Advisory identifier |
| `advisory_type` | 🔤 CharField(20) | ❌ | Advisory type |
| `hazard` | 🔤 CharField(20) | ✅ | Hazard type |
| `severity` | 🔤 CharField(20) | ✅ | Severity level |
| `valid_from` | 📅 DateTimeField | ✅ | Valid from time |
| `valid_to` | 📅 DateTimeField | ✅ | Valid to time |
| `lower_alt_ft` | 🔢 IntegerField | ✅ | Lower altitude |
| `upper_alt_ft` | 🔢 IntegerField | ✅ | Upper altitude |
| `region` | 🔤 CharField(20) | ✅ | Geographic region |
| `polygon` | 📦 JSONField | ✅ | GeoJSON polygon |
| `raw_text` | 📝 TextField | ✅ | Raw advisory text |
| `source_data` | 📦 JSONField | ✅ | Raw API response |

</details>

---

<details>
<summary><strong>🔲 AirspaceBoundary</strong> - Static airspace boundary data (Class B/C/D, MOAs)</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `airspace_boundaries` |
| **Primary Key** | `id` (AutoField) |
| **Update Frequency** | Monthly |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `fetched_at` | 📅 DateTimeField | ❌ | Fetch timestamp |
| `name` | 🔤 CharField(100) | ❌ | Airspace name |
| `icao` | 🔤 CharField(4) | ✅ | Associated airport |
| `airspace_class` | 🔤 CharField(20) | ❌ | Airspace class |
| `floor_ft` | 🔢 IntegerField | ❌ | Floor altitude |
| `ceiling_ft` | 🔢 IntegerField | ❌ | Ceiling altitude |
| `center_lat` | 🔢 FloatField | ❌ | Center latitude |
| `center_lon` | 🔢 FloatField | ❌ | Center longitude |
| `radius_nm` | 🔢 FloatField | ✅ | Radius (circular) |
| `polygon` | 📦 JSONField | ✅ | GeoJSON polygon |
| `controlling_agency` | 🔤 CharField(100) | ✅ | Controlling agency |
| `schedule` | 🔤 CharField(200) | ✅ | Operating schedule |
| `source` | 🔤 CharField(50) | ❌ | Data source |
| `source_id` | 🔤 CharField(100) | ✅ | External ID |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

</details>

---

<details>
<summary><strong>✈️ CachedPirep</strong> - Cached Pilot Reports (PIREPs)</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `cached_pireps` |
| **Unique Constraint** | `pirep_id` |
| **Retention** | 24 hours |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `fetched_at` | 📅 DateTimeField | ❌ | Fetch timestamp |
| `pirep_id` | 🔤 CharField(100) | ❌ | PIREP identifier (unique) |
| `report_type` | 🔤 CharField(10) | ❌ | Report type (UA/UUA) |
| `latitude` | 🔢 FloatField | ✅ | Location latitude |
| `longitude` | 🔢 FloatField | ✅ | Location longitude |
| `location` | 🔤 CharField(50) | ✅ | Location string |
| `observation_time` | 📅 DateTimeField | ✅ | Observation time |
| `flight_level` | 🔢 IntegerField | ✅ | Flight level |
| `altitude_ft` | 🔢 IntegerField | ✅ | Altitude in feet |
| `aircraft_type` | 🔤 CharField(10) | ✅ | Reporting aircraft type |
| `turbulence_type` | 🔤 CharField(20) | ✅ | Turbulence intensity |
| `turbulence_freq` | 🔤 CharField(20) | ✅ | Turbulence frequency |
| `turbulence_base_ft` | 🔢 IntegerField | ✅ | Turbulence base |
| `turbulence_top_ft` | 🔢 IntegerField | ✅ | Turbulence top |
| `icing_type` | 🔤 CharField(20) | ✅ | Icing intensity |
| `icing_intensity` | 🔤 CharField(20) | ✅ | Icing type |
| `icing_base_ft` | 🔢 IntegerField | ✅ | Icing base |
| `icing_top_ft` | 🔢 IntegerField | ✅ | Icing top |
| `sky_cover` | 🔤 CharField(100) | ✅ | Sky condition |
| `visibility_sm` | 🔢 FloatField | ✅ | Visibility (SM) |
| `weather` | 🔤 CharField(100) | ✅ | Weather phenomena |
| `temperature_c` | 🔢 IntegerField | ✅ | Temperature |
| `wind_dir` | 🔢 IntegerField | ✅ | Wind direction |
| `wind_speed_kt` | 🔢 IntegerField | ✅ | Wind speed |
| `raw_text` | 📝 TextField | ✅ | Raw PIREP text |
| `source_data` | 📦 JSONField | ✅ | Raw API response |
| `is_archived` | ✓ BooleanField | ❌ | Archived status |
| `archived_at` | 📅 DateTimeField | ✅ | Archive time |
| `archive_reason` | 🔤 CharField(50) | ✅ | Archive reason |

</details>

---

### 🎙️ Audio Transmissions

<details>
<summary><strong>🔊 AudioTransmission</strong> - Audio transmissions captured for transcription</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `audio_transmissions` |
| **Primary Key** | `id` (AutoField) |
| **Storage** | S3/Local |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `created_at` | 📅 DateTimeField | ❌ | Creation timestamp |
| `filename` | 🔤 CharField(255) | ❌ | Audio filename |
| `s3_key` | 🔤 CharField(500) | ✅ | S3 object key |
| `s3_url` | 🔤 CharField(500) | ✅ | S3 URL |
| `file_size_bytes` | 🔢 IntegerField | ✅ | File size |
| `duration_seconds` | 🔢 FloatField | ✅ | Audio duration |
| `format` | 🔤 CharField(10) | ❌ | Audio format |
| `frequency_mhz` | 🔢 FloatField | ✅ | Radio frequency |
| `channel_name` | 🔤 CharField(100) | ✅ | Channel name |
| `squelch_level` | 🔢 FloatField | ✅ | Squelch level |
| `transcription_status` | 🔤 CharField(20) | ❌ | Transcription status |
| `transcription_queued_at` | 📅 DateTimeField | ✅ | Queue time |
| `transcription_started_at` | 📅 DateTimeField | ✅ | Start time |
| `transcription_completed_at` | 📅 DateTimeField | ✅ | Completion time |
| `transcription_error` | 📝 TextField | ✅ | Error message |
| `transcript` | 📝 TextField | ✅ | Transcription text |
| `transcript_confidence` | 🔢 FloatField | ✅ | Confidence score |
| `transcript_language` | 🔤 CharField(10) | ✅ | Detected language |
| `transcript_segments` | 📦 JSONField | ✅ | Word-level timestamps |
| `identified_airframes` | 📦 JSONField | ✅ | Identified aircraft |
| `extra_metadata` | 📦 JSONField | ✅ | Additional metadata |

</details>

---

### 📡 Antenna Analytics

<details>
<summary><strong>📊 AntennaAnalyticsSnapshot</strong> - Periodic antenna performance snapshots</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `antenna_analytics_snapshots` |
| **Primary Key** | `id` (AutoField) |
| **Snapshot Types** | scheduled, hourly, daily |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `timestamp` | 📅 DateTimeField | ❌ | Snapshot timestamp |
| `snapshot_type` | 🔤 CharField(20) | ❌ | Type (scheduled/hourly/daily) |
| `window_hours` | 🔢 FloatField | ❌ | Time window size |
| `max_range_nm` | 🔢 FloatField | ✅ | Maximum range |
| `avg_range_nm` | 🔢 FloatField | ✅ | Average range |
| `min_range_nm` | 🔢 FloatField | ✅ | Minimum range |
| `range_p50_nm` | 🔢 FloatField | ✅ | 50th percentile range |
| `range_p75_nm` | 🔢 FloatField | ✅ | 75th percentile range |
| `range_p90_nm` | 🔢 FloatField | ✅ | 90th percentile range |
| `range_p95_nm` | 🔢 FloatField | ✅ | 95th percentile range |
| `best_rssi` | 🔢 FloatField | ✅ | Best signal strength |
| `avg_rssi` | 🔢 FloatField | ✅ | Average signal strength |
| `worst_rssi` | 🔢 FloatField | ✅ | Worst signal strength |
| `total_positions` | 🔢 IntegerField | ❌ | Total position count |
| `unique_aircraft` | 🔢 IntegerField | ❌ | Unique aircraft count |
| `positions_per_hour` | 🔢 FloatField | ❌ | Position rate |
| `range_by_direction` | 📦 JSONField | ❌ | Directional range data |
| `sectors_with_data` | 🔢 IntegerField | ❌ | Sectors with data (0-12) |
| `coverage_percentage` | 🔢 FloatField | ❌ | Coverage percentage |
| `estimated_gain_db` | 🔢 FloatField | ✅ | Estimated antenna gain |
| `performance_score` | 🔢 FloatField | ✅ | Performance score (0-100) |

</details>

---

### ⭐ Engagement Tracking

<details>
<summary><strong>❤️ AircraftFavorite</strong> - User-favorited aircraft tracking</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `aircraft_favorites` |
| **Unique Constraints** | `(user, icao_hex)`, `(session_key, icao_hex)` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `user_id` | 🔗 ForeignKey | ✅ | User reference |
| `session_key` | 🔤 CharField(40) | ✅ | Anonymous session |
| `icao_hex` | 🔤 CharField(10) | ❌ | Aircraft ICAO |
| `registration` | 🔤 CharField(20) | ✅ | Registration |
| `callsign` | 🔤 CharField(10) | ✅ | Last callsign |
| `notes` | 📝 TextField | ✅ | User notes |
| `times_seen` | 🔢 IntegerField | ❌ | Times seen while favorited |
| `last_seen_at` | 📅 DateTimeField | ✅ | Last seen time |
| `total_tracking_minutes` | 🔢 FloatField | ❌ | Total tracking time |
| `notify_on_detection` | ✓ BooleanField | ❌ | Notification enabled |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

</details>

---

<details>
<summary><strong>📶 SessionTrackingQuality</strong> - Extended session quality metrics</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `session_tracking_quality` |
| **Relation** | OneToOne with AircraftSession |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `session_id` | 🔗 OneToOneField | ❌ | Session reference |
| `expected_positions` | 🔢 IntegerField | ❌ | Expected positions |
| `actual_positions` | 🔢 IntegerField | ❌ | Actual positions |
| `completeness_score` | 🔢 FloatField | ❌ | Completeness (0-100) |
| `avg_update_rate` | 🔢 FloatField | ❌ | Positions per minute |
| `total_gaps` | 🔢 IntegerField | ❌ | Number of gaps |
| `max_gap_seconds` | 🔢 IntegerField | ❌ | Longest gap |
| `avg_gap_seconds` | 🔢 FloatField | ❌ | Average gap |
| `gap_percentage` | 🔢 FloatField | ❌ | Gap percentage |
| `avg_rssi` | 🔢 FloatField | ✅ | Average signal |
| `rssi_variance` | 🔢 FloatField | ✅ | Signal variance |
| `quality_grade` | 🔤 CharField(20) | ❌ | Quality grade |
| `calculated_at` | 📅 DateTimeField | ❌ | Calculation time |

</details>

---

### 🏆 Gamification & Statistics

<details>
<summary><strong>🥇 PersonalRecord</strong> - All-time personal bests</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `personal_records` |
| **Unique Constraint** | `record_type` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `record_type` | 🔤 CharField(50) | ❌ | Record type (unique) |
| `icao_hex` | 🔤 CharField(10) | ❌ | Record-holding aircraft |
| `callsign` | 🔤 CharField(10) | ✅ | Callsign |
| `aircraft_type` | 🔤 CharField(10) | ✅ | Aircraft type |
| `registration` | 🔤 CharField(20) | ✅ | Registration |
| `operator` | 🔤 CharField(100) | ✅ | Operator |
| `value` | 🔢 FloatField | ❌ | Record value |
| `session_id` | 🔗 ForeignKey | ✅ | Session reference |
| `achieved_at` | 📅 DateTimeField | ❌ | Achievement time |
| `previous_value` | 🔢 FloatField | ✅ | Previous record |
| `previous_icao_hex` | 🔤 CharField(10) | ✅ | Previous holder |
| `previous_achieved_at` | 📅 DateTimeField | ✅ | Previous time |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

#### Record Types

| Record | Icon | Description |
|--------|:----:|-------------|
| `max_distance` | 📏 | Furthest Aircraft Tracked |
| `max_altitude` | ⬆️ | Highest Altitude Aircraft |
| `max_speed` | 💨 | Fastest Aircraft Tracked |
| `longest_session` | ⏱️ | Longest Tracking Session |
| `most_positions` | 📍 | Most Positions for Single Aircraft |
| `closest_approach` | 🎯 | Closest Approach |
| `max_vertical_rate` | 🚀 | Fastest Climb Rate |
| `max_descent_rate` | 📉 | Fastest Descent Rate |
| `earliest_morning` | 🌅 | Earliest Morning Sighting |
| `latest_night` | 🌙 | Latest Night Sighting |

</details>

---

<details>
<summary><strong>📅 DailyStats</strong> - Daily aggregated statistics</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `daily_stats` |
| **Unique Constraint** | `date` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `date` | 📅 DateField | ❌ | Statistics date (unique) |
| `unique_aircraft` | 🔢 IntegerField | ❌ | Unique aircraft count |
| `new_aircraft` | 🔢 IntegerField | ❌ | First-time sightings |
| `total_sessions` | 🔢 IntegerField | ❌ | Session count |
| `total_positions` | 🔢 IntegerField | ❌ | Position count |
| `military_count` | 🔢 IntegerField | ❌ | Military aircraft count |
| `max_distance_nm` | 🔢 FloatField | ✅ | Maximum distance |
| `max_altitude` | 🔢 IntegerField | ✅ | Maximum altitude |
| `max_speed` | 🔢 FloatField | ✅ | Maximum speed |
| `aircraft_types` | 📦 JSONField | ❌ | Type distribution |
| `operators` | 📦 JSONField | ❌ | Operator distribution |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

</details>

---

### 🚔 Cannonball Mode

Cannonball Mode provides law enforcement aircraft detection and pattern analysis.

```mermaid
flowchart LR
    subgraph Detection
        A[Known Aircraft DB] --> B{Match?}
        C[Pattern Analysis] --> B
    end

    subgraph Tracking
        B --> D[Cannonball Session]
        D --> E[Pattern Detection]
        D --> F[Alert Generation]
    end

    subgraph Analysis
        E --> G[Circling]
        E --> H[Loitering]
        E --> I[Grid Search]
        E --> J[Surveillance]
    end
```

<details>
<summary><strong>🔄 CannonballPattern</strong> - Detected flight patterns indicating surveillance activity</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `cannonball_patterns` |
| **Primary Key** | `id` (AutoField) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `icao_hex` | 🔤 CharField(10) | ❌ | Aircraft ICAO |
| `callsign` | 🔤 CharField(10) | ✅ | Callsign |
| `pattern_type` | 🔤 CharField(30) | ❌ | Pattern type |
| `confidence` | 🔤 CharField(10) | ❌ | Confidence level |
| `confidence_score` | 🔢 FloatField | ❌ | Confidence score (0-1) |
| `center_lat` | 🔢 FloatField | ❌ | Pattern center latitude |
| `center_lon` | 🔢 FloatField | ❌ | Pattern center longitude |
| `radius_nm` | 🔢 FloatField | ✅ | Pattern radius |
| `pattern_data` | 📦 JSONField | ❌ | Additional pattern data |
| `position_samples` | 📦 JSONField | ❌ | Position samples |
| `started_at` | 📅 DateTimeField | ❌ | Pattern start time |
| `ended_at` | 📅 DateTimeField | ✅ | Pattern end time |
| `duration_seconds` | 🔢 IntegerField | ❌ | Duration |
| `detected_at` | 📅 DateTimeField | ❌ | Detection timestamp |
| `session_id` | 🔗 ForeignKey | ✅ | Session reference |

#### Pattern Types

| Type | Icon | Description |
|------|:----:|-------------|
| `circling` | 🔄 | Circling |
| `loitering` | ⏸️ | Loitering |
| `grid_search` | 🔲 | Grid Search |
| `speed_trap` | 🚗 | Speed Trap |
| `parallel_highway` | 🛣️ | Parallel to Highway |
| `surveillance` | 👁️ | General Surveillance |
| `pursuit` | 🏃 | Pursuit Pattern |

</details>

---

<details>
<summary><strong>📋 CannonballSession</strong> - Tracking session for potential law enforcement aircraft</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `cannonball_sessions` |
| **Primary Key** | `id` (AutoField) |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `icao_hex` | 🔤 CharField(10) | ❌ | Aircraft ICAO |
| `callsign` | 🔤 CharField(10) | ✅ | Callsign |
| `registration` | 🔤 CharField(15) | ✅ | Registration |
| `identification_method` | 🔤 CharField(20) | ❌ | How identified |
| `identification_reason` | 🔤 CharField(200) | ✅ | Identification reason |
| `operator_name` | 🔤 CharField(100) | ✅ | Operator name |
| `operator_icao` | 🔤 CharField(10) | ✅ | Operator ICAO |
| `aircraft_type` | 🔤 CharField(50) | ✅ | Aircraft type |
| `is_active` | ✓ BooleanField | ❌ | Active status |
| `threat_level` | 🔤 CharField(20) | ❌ | Threat level |
| `urgency_score` | 🔢 FloatField | ❌ | Urgency score (0-100) |
| `last_lat` | 🔢 FloatField | ✅ | Last latitude |
| `last_lon` | 🔢 FloatField | ✅ | Last longitude |
| `last_altitude` | 🔢 IntegerField | ✅ | Last altitude |
| `last_ground_speed` | 🔢 IntegerField | ✅ | Last ground speed |
| `last_track` | 🔢 IntegerField | ✅ | Last track |
| `user_id` | 🔗 ForeignKey | ✅ | User reference |
| `distance_nm` | 🔢 FloatField | ✅ | Distance from user |
| `bearing` | 🔢 FloatField | ✅ | Bearing from user |
| `closing_speed_kts` | 🔢 FloatField | ✅ | Closing speed |
| `first_seen` | 📅 DateTimeField | ❌ | Session start |
| `last_seen` | 📅 DateTimeField | ❌ | Last update |
| `session_duration_seconds` | 🔢 IntegerField | ❌ | Duration |
| `pattern_count` | 🔢 IntegerField | ❌ | Pattern count |
| `alert_count` | 🔢 IntegerField | ❌ | Alert count |
| `position_count` | 🔢 IntegerField | ❌ | Position count |
| `metadata` | 📦 JSONField | ❌ | Additional metadata |

</details>

---

<details>
<summary><strong>🗃️ CannonballKnownAircraft</strong> - Database of known law enforcement aircraft</summary>

#### Model Card

| Property | Value |
|----------|-------|
| **Table Name** | `cannonball_known_aircraft` |
| **Unique Constraint** | `icao_hex` |

#### Fields

| Field | Type | ⚪ Null | Description |
|-------|:----:|:------:|-------------|
| `id` | 🔢 AutoField | ❌ | Primary key |
| `icao_hex` | 🔤 CharField(10) | ❌ | Aircraft ICAO (unique) |
| `registration` | 🔤 CharField(15) | ✅ | Registration |
| `aircraft_type` | 🔤 CharField(50) | ✅ | Aircraft type |
| `aircraft_model` | 🔤 CharField(100) | ✅ | Aircraft model |
| `agency_name` | 🔤 CharField(200) | ❌ | Agency name |
| `agency_type` | 🔤 CharField(20) | ❌ | Agency type |
| `agency_state` | 🔤 CharField(2) | ✅ | US state |
| `agency_city` | 🔤 CharField(100) | ✅ | City |
| `source` | 🔤 CharField(20) | ❌ | Data source |
| `source_url` | 🔗 URLField | ✅ | Source URL |
| `verified` | ✓ BooleanField | ❌ | Verified status |
| `verified_at` | 📅 DateTimeField | ✅ | Verification time |
| `verified_by_id` | 🔗 ForeignKey | ✅ | Verifying user |
| `times_detected` | 🔢 IntegerField | ❌ | Detection count |
| `last_detected` | 📅 DateTimeField | ✅ | Last detection |
| `notes` | 📝 TextField | ✅ | Notes |
| `created_at` | 📅 DateTimeField | ❌ | Creation time |
| `updated_at` | 📅 DateTimeField | ❌ | Last update |

</details>

---

## ⚡ Performance Optimization

### Index Strategy

SkySpy uses a comprehensive indexing strategy optimized for common query patterns.

```mermaid
graph TD
    subgraph "Query Patterns"
        A[Time-Based] --> A1["Recent sightings"]
        A --> A2["Session history"]

        B[Aircraft Lookups] --> B1["Track by ICAO"]
        B --> B2["History queries"]

        C[Filtered Queries] --> C1["Military aircraft"]
        C --> C2["Distance range"]
        C --> C3["Signal strength"]
    end

    subgraph "Index Types"
        D[Composite] --> D1["Multi-column"]
        E[Partial] --> E1["Conditional"]
        F[Descending] --> F1["Time ordering"]
    end

    A1 --> F
    A2 --> F
    B1 --> D
    B2 --> D
    C1 --> E
    C2 --> E
    C3 --> E
```

> **Tip**: Partial indexes in PostgreSQL dramatically reduce index size for sparse boolean columns like `is_military` (typically <5% of traffic).

#### Time-Based Queries

```sql
-- Most common pattern: Recent sightings ordered by time
CREATE INDEX idx_sighting_timestamp_desc ON aircraft_sightings(timestamp DESC);
CREATE INDEX idx_session_last_seen ON aircraft_sessions(last_seen DESC);
```

#### Aircraft Lookups

```sql
-- Track history for specific aircraft
CREATE INDEX idx_sighting_icao_timestamp ON aircraft_sightings(icao_hex, timestamp DESC);
CREATE INDEX idx_session_icao_lastseen ON aircraft_sessions(icao_hex, last_seen DESC);
```

#### Partial Indexes

```sql
-- Military aircraft filtering (typically <5% of traffic)
CREATE INDEX idx_sighting_military ON aircraft_sightings(is_military) WHERE is_military = TRUE;

-- Distance-based queries (exclude null values)
CREATE INDEX idx_sighting_distance ON aircraft_sightings(distance_nm) WHERE distance_nm IS NOT NULL;
```

### Query Optimization Tips

> **Performance Best Practices**

| Pattern | Recommendation |
|---------|----------------|
| 📅 Recent sightings | Use `timestamp__gte` with indexed column |
| ✈️ Aircraft history | Query by `icao_hex` first, then filter by time |
| 📏 Distance filtering | Use partial index, exclude nulls |
| 🎖️ Military aircraft | Partial index on boolean flag |
| 📋 Session lookups | Use composite `(icao_hex, last_seen)` index |

### Connection Pooling

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'CONN_MAX_AGE': 60,  # Reuse connections for 60 seconds
    }
}
```

---

## 📦 Data Retention Policies

### Active Data

| Table | Retention | Policy |
|-------|:---------:|--------|
| 📍 `aircraft_sightings` | 30 days | Archive older positions |
| 📋 `aircraft_sessions` | 90 days | Keep for history |
| 🔔 `alert_history` | 90 days | Archive older alerts |
| 📨 `notification_logs` | 30 days | Delete after retention |
| ⚠️ `safety_events` | 1 year | Archive for analysis |

### Cached Data

| Table | Retention | Policy |
|-------|:---------:|--------|
| 🛩️ `aircraft_info` | Indefinite | Update on access |
| 🛫 `cached_airports` | 30 days | Refresh periodically |
| 📋 `cached_notams` | Active + 7 days | Archive expired |
| ✈️ `cached_pireps` | 24 hours | Archive after expiry |
| 🌤️ `airspace_advisories` | Active + 24 hours | Archive expired |

### Archive Strategy

```python
# Mark expired NOTAMs as archived
CachedNotam.objects.filter(
    effective_end__lt=timezone.now() - timedelta(days=7),
    is_archived=False
).update(
    is_archived=True,
    archived_at=timezone.now(),
    archive_reason='expired'
)
```

---

## 🔄 Migration Strategy

### Running Migrations

```bash
# Apply all pending migrations
python manage.py migrate

# Show migration status
python manage.py showmigrations

# Create new migration
python manage.py makemigrations skyspy
```

### Migration Best Practices

> **Important Guidelines**

1. **Never edit applied migrations** - Create new migrations for changes
2. **Use atomic=False for index creation** - Allows `CREATE INDEX CONCURRENTLY`
3. **Test on copy of production data** - Verify performance impact
4. **Backup before major migrations** - Enable rollback capability

### PostgreSQL-Specific Migrations

```python
class Migration(migrations.Migration):
    atomic = False  # Required for CONCURRENTLY

    operations = [
        PostgreSQLOnlyRunSQL(
            sql="CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_name ON table(column);",
            reverse_sql="DROP INDEX CONCURRENTLY IF EXISTS idx_name;",
        ),
    ]
```

### Zero-Downtime Migrations

For large tables, use the following pattern:

1. Add new columns as nullable
2. Backfill data in batches
3. Add NOT NULL constraint if needed
4. Create indexes concurrently

---

## 📈 Database Monitoring

### Key Metrics

| Metric | ⚠️ Warning | 🔴 Critical |
|--------|:----------:|:-----------:|
| Connection count | 80% of max | 95% of max |
| Query duration (p99) | 500ms | 2000ms |
| Table bloat | 20% | 40% |
| Index bloat | 30% | 50% |
| Disk usage | 70% | 85% |

### Useful Queries

```sql
-- Active connections
SELECT count(*) FROM pg_stat_activity WHERE state = 'active';

-- Slow queries
SELECT query, calls, mean_time, max_time
FROM pg_stat_statements
ORDER BY mean_time DESC LIMIT 10;

-- Table sizes
SELECT relname, pg_size_pretty(pg_total_relation_size(relid))
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Index usage
SELECT indexrelname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

---

## 💾 Backup and Recovery

### Backup Strategy

```bash
# Full database backup
pg_dump -h localhost -U adsb adsb > backup_$(date +%Y%m%d).sql

# Schema-only backup
pg_dump -h localhost -U adsb --schema-only adsb > schema.sql

# Data-only backup
pg_dump -h localhost -U adsb --data-only adsb > data.sql
```

### Point-in-Time Recovery

Configure WAL archiving in PostgreSQL:

```ini
archive_mode = on
archive_command = 'cp %p /backup/wal/%f'
```

### Restore Procedure

```bash
# Restore from backup
psql -h localhost -U adsb adsb < backup.sql

# Restore to specific point in time
pg_restore --target-time="2024-01-15 10:00:00" -d adsb backup.dump
```

---

## 📜 Version History

| Version | Date | Changes |
|:-------:|:----:|---------|
| 0.1.0 | 2024-01 | Initial schema with aircraft tracking |
| 0.2.0 | 2024-06 | Added auth, alerts, notifications |
| 0.3.0 | 2024-09 | Added gamification, cannonball mode |
| 0.4.0 | 2024-12 | Performance indexes, archive support |
