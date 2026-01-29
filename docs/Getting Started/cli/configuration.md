---
title: "Configuration"
excerpt: "Configuration file format and all available options"
---

# Configuration

The SkySpy CLI stores its configuration in a JSON file that is automatically loaded on startup and saved on exit.

## Configuration File Location

```
~/.config/skyspy/settings.json
```

On different platforms:
- **Linux**: `/home/username/.config/skyspy/settings.json`
- **macOS**: `/Users/username/.config/skyspy/settings.json`
- **Windows**: `C:\Users\username\.config\skyspy\settings.json`

## Related Directories

| Path | Description |
|------|-------------|
| `~/.config/skyspy/` | Main configuration directory |
| `~/.config/skyspy/settings.json` | Main configuration file |
| `~/.config/skyspy/overlays/` | Default overlay storage directory |
| `~/.config/skyspy/credentials/` | Encrypted authentication tokens |

## Configuration Structure

The configuration file uses the following JSON structure:

```json
{
  "display": { ... },
  "radar": { ... },
  "filters": { ... },
  "connection": { ... },
  "audio": { ... },
  "overlays": { ... },
  "export": { ... },
  "alerts": { ... },
  "recent_hosts": [ ... ]
}
```

## Display Settings

Controls visual appearance and panel visibility.

```json
{
  "display": {
    "theme": "classic",
    "show_labels": true,
    "show_trails": false,
    "refresh_rate": 10,
    "compact_mode": false,
    "show_acars": true,
    "show_target_list": true,
    "show_vu_meters": true,
    "show_spectrum": true,
    "show_frequencies": true,
    "show_stats_panel": true
  }
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `theme` | string | `"classic"` | Color theme name |
| `show_labels` | bool | `true` | Show callsign labels on radar |
| `show_trails` | bool | `false` | Show aircraft movement trails |
| `refresh_rate` | int | `10` | Display refresh rate in Hz (1-60) |
| `compact_mode` | bool | `false` | Use compact display layout |
| `show_acars` | bool | `true` | Show ACARS message panel |
| `show_target_list` | bool | `true` | Show aircraft target list |
| `show_vu_meters` | bool | `true` | Show signal VU meters |
| `show_spectrum` | bool | `true` | Show frequency spectrum |
| `show_frequencies` | bool | `true` | Show frequency display |
| `show_stats_panel` | bool | `true` | Show statistics panel |

## Radar Settings

Controls radar scope appearance and behavior.

```json
{
  "radar": {
    "default_range": 100,
    "range_rings": 4,
    "sweep_speed": 6,
    "show_compass": true,
    "show_grid": false,
    "show_overlays": true,
    "overlay_color": "cyan"
  }
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `default_range` | int | `100` | Initial range in nautical miles |
| `range_rings` | int | `4` | Number of concentric range rings (0-10) |
| `sweep_speed` | int | `6` | Radar sweep animation speed (1-20) |
| `show_compass` | bool | `true` | Display compass rose |
| `show_grid` | bool | `false` | Display coordinate grid |
| `show_overlays` | bool | `true` | Display loaded map overlays |
| `overlay_color` | string | `"cyan"` | Default color for overlays |

## Filter Settings

Controls aircraft filtering.

```json
{
  "filters": {
    "military_only": false,
    "min_altitude": null,
    "max_altitude": null,
    "min_distance": null,
    "max_distance": null,
    "hide_ground": false
  }
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `military_only` | bool | `false` | Show only military aircraft |
| `min_altitude` | int/null | `null` | Minimum altitude filter (feet) |
| `max_altitude` | int/null | `null` | Maximum altitude filter (feet) |
| `min_distance` | float/null | `null` | Minimum distance filter (nm) |
| `max_distance` | float/null | `null` | Maximum distance filter (nm) |
| `hide_ground` | bool | `false` | Hide aircraft on ground |

## Connection Settings

Controls server connection parameters.

```json
{
  "connection": {
    "host": "localhost",
    "port": 80,
    "receiver_lat": 0.0,
    "receiver_lon": 0.0,
    "auto_reconnect": true,
    "reconnect_delay": 2
  }
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `host` | string | `"localhost"` | Server hostname |
| `port` | int | `80` | Server port |
| `receiver_lat` | float | `0.0` | Receiver latitude for distance calculation |
| `receiver_lon` | float | `0.0` | Receiver longitude for distance calculation |
| `auto_reconnect` | bool | `true` | Automatically reconnect on connection loss |
| `reconnect_delay` | int | `2` | Seconds to wait before reconnecting |

## Audio Settings

Controls audio alerts and sounds.

```json
{
  "audio": {
    "enabled": false,
    "new_aircraft_sound": true,
    "emergency_sound": true,
    "military_sound": false
  }
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | bool | `false` | Master audio enable switch |
| `new_aircraft_sound` | bool | `true` | Play sound for new aircraft |
| `emergency_sound` | bool | `true` | Play sound for emergency squawks |
| `military_sound` | bool | `false` | Play sound for military aircraft |

## Overlay Settings

Controls loaded map overlays.

```json
{
  "overlays": {
    "overlays": [
      {
        "path": "/path/to/airspace.geojson",
        "enabled": true,
        "color": "cyan",
        "name": "Airspace",
        "key": "airspace_1"
      }
    ],
    "custom_range_rings": []
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `overlays` | array | List of overlay configurations |
| `overlays[].path` | string | Absolute path to overlay file |
| `overlays[].enabled` | bool | Whether overlay is displayed |
| `overlays[].color` | string | Overlay line color |
| `overlays[].name` | string | Display name for overlay |
| `overlays[].key` | string | Unique identifier |
| `custom_range_rings` | int[] | Custom range ring distances (nm) |

## Export Settings

Controls data export behavior.

```json
{
  "export": {
    "directory": ""
  }
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `directory` | string | `""` | Export directory (empty = current directory) |

## Alert Settings

Controls custom alert rules and geofences.

```json
{
  "alerts": {
    "enabled": true,
    "rules": [
      {
        "id": "rule_1",
        "name": "Military Alert",
        "description": "Alert on military aircraft",
        "enabled": true,
        "conditions": [
          { "type": "military", "value": "true" }
        ],
        "actions": [
          { "type": "notify", "message": "Military aircraft detected!" },
          { "type": "sound", "sound": "alert" }
        ],
        "cooldown_sec": 60,
        "priority": 1
      }
    ],
    "geofences": [
      {
        "id": "fence_1",
        "name": "Airport Zone",
        "type": "circle",
        "center_lat": 40.7128,
        "center_lon": -74.0060,
        "radius_nm": 10,
        "enabled": true,
        "description": "New York area"
      }
    ],
    "log_file": "",
    "sound_dir": ""
  }
}
```

### Alert Rule Conditions

| Type | Value | Description |
|------|-------|-------------|
| `military` | `"true"` | Military aircraft |
| `emergency` | `"true"` | Emergency squawk (7500, 7600, 7700) |
| `squawk` | `"7700"` | Specific squawk code |
| `callsign` | `"UAL*"` | Callsign pattern (wildcards supported) |
| `altitude_below` | `"1000"` | Below altitude (feet) |
| `altitude_above` | `"40000"` | Above altitude (feet) |
| `geofence_enter` | `"fence_id"` | Entering a geofence |
| `geofence_exit` | `"fence_id"` | Exiting a geofence |

### Alert Rule Actions

| Type | Description |
|------|-------------|
| `notify` | Display notification message |
| `sound` | Play alert sound |

### Geofence Types

| Type | Required Fields |
|------|-----------------|
| `circle` | `center_lat`, `center_lon`, `radius_nm` |
| `polygon` | `points` (array of `{lat, lon}`) |

## Environment Variables

| Variable | Description |
|----------|-------------|
| `SKYSPY_API_KEY` | API key for authentication |

## Example Configuration

Complete example with all sections:

```json
{
  "display": {
    "theme": "cyberpunk",
    "show_labels": true,
    "show_trails": true,
    "refresh_rate": 15,
    "compact_mode": false,
    "show_acars": true,
    "show_target_list": true,
    "show_vu_meters": true,
    "show_spectrum": true,
    "show_frequencies": true,
    "show_stats_panel": true
  },
  "radar": {
    "default_range": 100,
    "range_rings": 4,
    "sweep_speed": 8,
    "show_compass": true,
    "show_grid": false,
    "show_overlays": true,
    "overlay_color": "cyan"
  },
  "filters": {
    "military_only": false,
    "hide_ground": true
  },
  "connection": {
    "host": "skyspy.example.com",
    "port": 443,
    "receiver_lat": 40.7128,
    "receiver_lon": -74.0060,
    "auto_reconnect": true,
    "reconnect_delay": 5
  },
  "audio": {
    "enabled": true,
    "new_aircraft_sound": false,
    "emergency_sound": true,
    "military_sound": true
  },
  "overlays": {
    "overlays": [
      {
        "path": "/home/user/.config/skyspy/overlays/airspace.geojson",
        "enabled": true,
        "color": "yellow",
        "key": "airspace"
      }
    ],
    "custom_range_rings": [25, 75, 150]
  },
  "export": {
    "directory": "/home/user/skyspy-exports"
  },
  "alerts": {
    "enabled": true,
    "rules": [
      {
        "id": "military_1",
        "name": "Military Aircraft",
        "enabled": true,
        "conditions": [{"type": "military", "value": "true"}],
        "actions": [{"type": "notify", "message": "Military aircraft detected"}],
        "cooldown_sec": 300,
        "priority": 1
      }
    ],
    "geofences": []
  },
  "recent_hosts": ["skyspy.example.com:443", "localhost:80"]
}
```
