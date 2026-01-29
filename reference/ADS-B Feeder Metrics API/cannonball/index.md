---
title: "Cannonball Mode"
excerpt: "Law enforcement aircraft detection and tracking"
hidden: false
---

Cannonball Mode is a specialized feature for detecting and tracking potential law enforcement and surveillance aircraft in real-time. It analyzes aircraft behavior patterns, cross-references against a known LE aircraft database, and provides distance-based threat assessments.

## Features

- **Real-time Threat Detection**: Identifies helicopters and aircraft exhibiting surveillance patterns
- **Pattern Analysis**: Detects circling, loitering, grid search, and speed trap patterns
- **Known LE Database**: Cross-references aircraft against a curated database of law enforcement aircraft
- **Distance-based Alerts**: Calculates distance from your position and alerts on proximity
- **Session Tracking**: Groups detections into sessions for historical analysis

## Endpoints

### Threat Detection
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | [`/api/v1/cannonball/threats/`](./get_threats) | Get current threats |
| POST | [`/api/v1/cannonball/location/`](./update_location) | Update GPS location |

### Mode Activation
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | [`/api/v1/cannonball/activate/`](./activate) | Activate Cannonball mode |
| DELETE | [`/api/v1/cannonball/activate/`](./deactivate) | Deactivate Cannonball mode |

### Sessions
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | [`/api/v1/cannonball/sessions/`](./list_sessions) | List all sessions |
| GET | [`/api/v1/cannonball/sessions/active/`](./active_sessions) | Get active sessions |
| POST | [`/api/v1/cannonball/sessions/{id}/end/`](./end_session) | End a session |

### Patterns
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | [`/api/v1/cannonball/patterns/`](./list_patterns) | List detected patterns |
| GET | [`/api/v1/cannonball/patterns/stats/`](./pattern_stats) | Get pattern statistics |

### Alerts
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | [`/api/v1/cannonball/alerts/`](./list_alerts) | List alerts |
| POST | [`/api/v1/cannonball/alerts/{id}/acknowledge/`](./acknowledge_alert) | Acknowledge an alert |
| POST | [`/api/v1/cannonball/alerts/acknowledge-all/`](./acknowledge_all) | Acknowledge all alerts |

### Known Aircraft
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | [`/api/v1/cannonball/known-aircraft/`](./known_aircraft) | List known LE aircraft |

### Statistics
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | [`/api/v1/cannonball/stats/`](./stats) | Get statistics |

### WebSocket
| Endpoint | Description |
|----------|-------------|
| [`/ws/cannonball/`](./ws) | Real-time threat updates |

## Threat Levels

| Level | Description |
|-------|-------------|
| `info` | Aircraft of interest detected at distance |
| `warning` | Aircraft approaching or exhibiting patterns |
| `critical` | Confirmed LE aircraft in close proximity |

## Pattern Types

| Pattern | Description |
|---------|-------------|
| `circling` | Aircraft making repeated circular orbits |
| `loitering` | Aircraft remaining in a small area |
| `grid_search` | Systematic back-and-forth pattern |
| `speed_trap` | Low-altitude parallel highway flight |
| `parallel_highway` | Extended flight parallel to highway |
| `surveillance` | General surveillance behavior |
| `pursuit` | Active pursuit pattern |

## Authentication

All endpoints support both JWT authentication and API key authentication. Anonymous access is allowed with limited functionality.
