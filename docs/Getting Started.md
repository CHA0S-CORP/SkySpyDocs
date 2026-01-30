---
title: "SkySpy CLI"
excerpt: "A native Go terminal-based aircraft tracking application"
---

# SkySpy CLI

The SkySpy CLI is a powerful, native Go terminal application for real-time aircraft tracking and monitoring. Built with modern terminal UI libraries, it provides an immersive radar-style experience directly in your terminal.

## Features

- **Interactive Radar Display** - Full-screen radar scope with animated sweep, range rings, and compass rose
- **Real-time Aircraft Tracking** - Live ADS-B data with position, altitude, speed, and heading
- **ACARS/VDL2 Message Feed** - Decode and display aircraft data link messages
- **VU Meters & Spectrum Analyzer** - Visual signal strength indicators and frequency spectrum display
- **10 Color Themes** - Classic green phosphor, amber, cyberpunk, military, and more
- **GeoJSON/Shapefile Overlays** - Load custom map overlays for airspace boundaries, coastlines, etc.
- **Custom Alert Rules** - Define alerts for military aircraft, emergencies, geofence entry, and more
- **Export Capabilities** - Export data to CSV, JSON, or HTML screenshots
- **Authentication Support** - OIDC and API key authentication for secure server connections

## Quick Start

```bash
# Connect to a local SkySpy server
skyspy --host localhost --port 80

# Use a specific theme
skyspy --theme cyberpunk

# Set receiver location for distance/bearing calculations
skyspy --lat 40.7128 --lon -74.0060

# Load map overlays
skyspy --overlay airspace.geojson --overlay coastline.shp
```

## Installation

See [Installation](cli/installation) for detailed installation instructions including:
- Building from source
- Cross-platform compilation
- Binary distribution

## Documentation

<Cards columns={3}>
  <Card title="Installation" icon="fa-download" href="/docs/cli/installation">
    Build from source or download binaries
  </Card>
  <Card title="Commands" icon="fa-terminal" href="/docs/cli/commands">
    Complete command and flag reference
  </Card>
  <Card title="Configuration" icon="fa-cog" href="/docs/cli/configuration">
    Configuration file format and options
  </Card>
  <Card title="Keyboard Controls" icon="fa-keyboard" href="/docs/cli/keyboard-controls">
    Interactive keyboard shortcuts
  </Card>
  <Card title="Themes" icon="fa-palette" href="/docs/cli/themes">
    10 color themes available
  </Card>
  <Card title="Authentication" icon="fa-lock" href="/docs/cli/authentication">
    OIDC and API key authentication
  </Card>
</Cards>

## System Requirements

- **Terminal**: Any modern terminal with 256-color support
- **Size**: Minimum 80x24 characters recommended (larger terminals show more detail)
- **OS**: Linux, macOS, Windows (with Windows Terminal recommended)

## Screenshots

The CLI provides multiple interface modes:

### Main Radar View
The default view shows a radar scope with aircraft positions, an aircraft list panel, ACARS message feed, and signal meters.

### Radio Mode
A retro-styled interface focused on aircraft monitoring with a classic radio aesthetic.

### Radio Pro Mode
Enhanced radio interface with VU meters, spectrum analyzer, and waterfall display.

## Getting Help

Press `?` or `h` while running the CLI to view the built-in help screen with all keyboard shortcuts.

```bash
# List available themes
skyspy --list-themes

# Show command help
skyspy --help
skyspy radio --help
skyspy login --help
```
