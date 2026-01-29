---
title: "Commands"
excerpt: "Complete reference for all SkySpy CLI commands and flags"
---

# Commands

The SkySpy CLI uses a hierarchical command structure with a root command and several subcommands.

## Global Flags

These flags are available to all commands:

| Flag | Type | Description |
|------|------|-------------|
| `--host` | string | Server hostname |
| `--port` | int | Server port |

---

## Commands

<Accordion title="skyspy (root command)" icon="fa-radar">

The main command launches the full-featured radar display.

```bash
skyspy [flags]
```

**Description:** SkySpy Radar Pro - Full-Featured Aircraft Display. Interactive radar with overlays, VU meters, spectrum, and themes. Settings are automatically saved to `~/.config/skyspy/settings.json`.

**Flags:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--lat` | float | 0 | Receiver latitude for distance/bearing calculations |
| `--lon` | float | 0 | Receiver longitude for distance/bearing calculations |
| `--range` | int | 100 | Initial radar range in nautical miles |
| `--theme` | string | classic | Color theme (use `--list-themes` to see options) |
| `--overlay` | string[] | [] | Load overlay file (GeoJSON/Shapefile). Can be repeated. |
| `--list-themes` | bool | false | List available themes and exit |
| `--api-key` | string | | API key for authentication (or use `SKYSPY_API_KEY` env) |
| `--export-dir` | string | . | Directory for export files |
| `--no-audio` | bool | false | Disable audio alerts |

**Examples:**

```bash
# Basic connection
skyspy --host myserver.com --port 80

# With theme and location
skyspy --theme cyberpunk --lat 40.7128 --lon -74.0060 --range 50

# With overlays
skyspy --overlay airspace.geojson --overlay coastline.shp
```

</Accordion>

<Accordion title="login" icon="fa-sign-in-alt">

Authenticate with the SkySpy server using OIDC.

```bash
skyspy login [flags]
```

**Description:** Opens your web browser for authentication. After successful login, credentials are stored securely and used for subsequent connections.

**Behavior:**
1. Connects to the server to check authentication requirements
2. If OIDC is enabled, opens a browser for authentication
3. Waits for the authentication callback (5-minute timeout)
4. Stores tokens securely in `~/.config/skyspy/credentials/`

**Examples:**

```bash
# Login to default server
skyspy login

# Login to specific server
skyspy login --host myserver.com --port 443
```

</Accordion>

<Accordion title="logout" icon="fa-sign-out-alt">

Clear stored credentials for the SkySpy server.

```bash
skyspy logout [flags]
```

**Description:** Removes stored authentication tokens for the specified server.

**Examples:**

```bash
# Logout from default server
skyspy logout

# Logout from specific server
skyspy logout --host myserver.com --port 443
```

</Accordion>

<Accordion title="auth status" icon="fa-user-check">

Show current authentication status.

```bash
skyspy auth status [flags]
```

**Description:** Displays detailed information about the current authentication state including server configuration, current status, token expiration, and username.

**Sample Output:**

```
Server: myserver.com:443

Server Configuration:
  Auth Mode: oidc
  Auth Required: true
  OIDC: enabled (via Google)

Authentication Status:
  Status: Authenticated
  User: user@example.com
  Token Expires: 2024-01-15T10:30:00Z
```

</Accordion>

<Accordion title="radio" icon="fa-radio">

Launch the retro-styled radio monitor interface.

```bash
skyspy radio [flags]
```

**Description:** SkySpy Radio - Old School Aircraft Monitor. A retro terminal interface for live ADS-B and ACARS tracking.

**Flags:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--frequency` | string | | Monitor specific frequency (e.g., 1090, 136.9) |
| `--scan` | bool | false | Enable frequency scanning mode |

**Examples:**

```bash
# Basic radio mode
skyspy radio

# Scan mode
skyspy radio --scan

# Monitor specific frequency
skyspy radio --frequency 1090
```

</Accordion>

<Accordion title="radio-pro" icon="fa-broadcast-tower">

Launch the enhanced radio monitor with VU meters and spectrum display.

```bash
skyspy radio-pro [flags]
```

**Description:** SkySpy Radio PRO - Ultimate Aircraft Monitor. A fully immersive retro terminal interface with VU meters, spectrum display, and waterfall visualization.

**Features:**
- Live aircraft tracking table with detailed info
- ACARS/VDL2 message feed
- Real-time VU meters
- Spectrum analyzer display
- Frequency scanning visualization
- Signal history and waterfall display

**Flags:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--frequency` | string | | Monitor specific frequency (e.g., 1090, 136.9) |
| `--scan` | bool | false | Enable frequency scanning mode |

**Examples:**

```bash
# Launch radio pro mode
skyspy radio-pro

# With frequency scanning
skyspy radio-pro --scan
```

</Accordion>

<Accordion title="configure" icon="fa-cog">

Launch the interactive configuration wizard.

```bash
skyspy configure
```

**Description:** An interactive terminal wizard that guides you through configuring all SkySpy settings: Connection, Display, Radar, and Audio.

Settings are saved to `~/.config/skyspy/settings.json`.

**Navigation:**

| Key | Action |
|-----|--------|
| `Tab` / `Down` | Next field |
| `Shift+Tab` / `Up` | Previous field |
| `Space` | Toggle boolean fields |
| `Left` / `Right` | Cycle through select options |
| `Enter` | Confirm and continue |
| `Esc` | Go back |
| `q` | Quit (on welcome/summary screens) |

</Accordion>

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `SKYSPY_API_KEY` | API key for authentication (alternative to `--api-key` flag) |

### Example

```bash
export SKYSPY_API_KEY=sk_live_xxxxxxxxxxxxx
skyspy  # Will use the API key automatically
```
