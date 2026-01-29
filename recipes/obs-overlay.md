---
title: OBS Browser Overlay
excerpt: Stream overlay for OBS showing aircraft info
hidden: false
recipe:
  color: '#302E31'
  icon: 🎬
difficulty: beginner
tags: [visualization, obs, streaming, overlay]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- OBS Studio installed
- Basic HTML/CSS knowledge helpful

## What You'll Build

A transparent browser overlay for OBS that displays real-time aircraft data including total aircraft count, closest aircraft details, and military alerts. Perfect for aviation streaming, planespotting broadcasts, or adding live flight data to any stream.

```shell Shell
# Create project directory
mkdir skyspy-obs-overlay && cd skyspy-obs-overlay

# Create the HTML file (content below)
# Then add as browser source in OBS
```

```html HTML (overlay.html)
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>SkySpy OBS Overlay</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: transparent;
            font-family: 'Segoe UI', 'Roboto', sans-serif;
            color: #ffffff;
            overflow: hidden;
        }

        .overlay-container {
            position: fixed;
            bottom: 20px;
            left: 20px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            max-width: 400px;
        }

        .panel {
            background: rgba(0, 0, 0, 0.75);
            border-radius: 8px;
            padding: 15px;
            border-left: 4px solid #3b82f6;
            backdrop-filter: blur(10px);
        }

        .panel-header {
            font-size: 12px;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: #94a3b8;
            margin-bottom: 8px;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
        }

        .stat {
            text-align: center;
        }

        .stat-value {
            font-size: 32px;
            font-weight: bold;
            line-height: 1;
        }

        .stat-label {
            font-size: 11px;
            color: #94a3b8;
            text-transform: uppercase;
            margin-top: 4px;
        }

        .stat-aircraft .stat-value { color: #3b82f6; }
        .stat-military .stat-value { color: #f59e0b; }
        .stat-closest .stat-value { color: #22c55e; }

        .closest-panel {
            border-left-color: #22c55e;
        }

        .closest-info {
            display: flex;
            align-items: center;
            gap: 15px;
        }

        .closest-icon {
            font-size: 36px;
            transform: rotate(0deg);
            transition: transform 0.3s ease;
        }

        .closest-details {
            flex: 1;
        }

        .closest-callsign {
            font-size: 20px;
            font-weight: bold;
            color: #ffffff;
        }

        .closest-type {
            font-size: 14px;
            color: #94a3b8;
        }

        .closest-stats {
            display: flex;
            gap: 20px;
            margin-top: 8px;
            font-size: 13px;
        }

        .closest-stat {
            display: flex;
            align-items: center;
            gap: 5px;
        }

        .closest-stat-icon {
            color: #64748b;
        }

        .military-alert {
            border-left-color: #f59e0b;
            animation: pulse 2s infinite;
        }

        .military-alert .panel-header {
            color: #f59e0b;
        }

        .military-list {
            display: flex;
            flex-direction: column;
            gap: 8px;
        }

        .military-item {
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px;
            background: rgba(245, 158, 11, 0.1);
            border-radius: 4px;
        }

        .military-badge {
            font-size: 18px;
        }

        .military-callsign {
            font-weight: bold;
            color: #f59e0b;
        }

        .military-type {
            font-size: 12px;
            color: #94a3b8;
        }

        .hidden {
            display: none !important;
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.8; }
        }

        /* Connection status indicator */
        .connection-status {
            position: fixed;
            top: 10px;
            right: 10px;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background: #ef4444;
        }

        .connection-status.connected {
            background: #22c55e;
        }
    </style>
</head>
<body>
    <div class="connection-status" id="connectionStatus"></div>

    <div class="overlay-container">
        <!-- Stats Panel -->
        <div class="panel stats-panel">
            <div class="panel-header">SkySpy Live</div>
            <div class="stats-grid">
                <div class="stat stat-aircraft">
                    <div class="stat-value" id="totalAircraft">0</div>
                    <div class="stat-label">Aircraft</div>
                </div>
                <div class="stat stat-military">
                    <div class="stat-value" id="militaryCount">0</div>
                    <div class="stat-label">Military</div>
                </div>
                <div class="stat stat-closest">
                    <div class="stat-value" id="closestDistance">--</div>
                    <div class="stat-label">Closest (nm)</div>
                </div>
            </div>
        </div>

        <!-- Closest Aircraft Panel -->
        <div class="panel closest-panel" id="closestPanel">
            <div class="panel-header">Closest Aircraft</div>
            <div class="closest-info">
                <div class="closest-icon" id="closestIcon">✈</div>
                <div class="closest-details">
                    <div class="closest-callsign" id="closestCallsign">---</div>
                    <div class="closest-type" id="closestType">---</div>
                    <div class="closest-stats">
                        <div class="closest-stat">
                            <span class="closest-stat-icon">↑</span>
                            <span id="closestAlt">---</span> ft
                        </div>
                        <div class="closest-stat">
                            <span class="closest-stat-icon">→</span>
                            <span id="closestSpeed">---</span> kts
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Military Alert Panel -->
        <div class="panel military-alert hidden" id="militaryPanel">
            <div class="panel-header">Military Activity</div>
            <div class="military-list" id="militaryList">
                <!-- Military aircraft will be populated here -->
            </div>
        </div>
    </div>

    <script>
        // Configuration - Edit these values
        const CONFIG = {
            skyspyUrl: 'http://localhost:5000',
            showMilitaryAlerts: true,
            maxMilitaryDisplay: 3,
            staleTimeout: 60000  // Remove aircraft after 60s of no updates
        };

        // Parse URL parameters for configuration
        const urlParams = new URLSearchParams(window.location.search);
        if (urlParams.get('url')) CONFIG.skyspyUrl = urlParams.get('url');
        if (urlParams.get('military') === 'false') CONFIG.showMilitaryAlerts = false;

        // State
        const aircraft = new Map();
        let isConnected = false;

        // DOM Elements
        const elements = {
            connectionStatus: document.getElementById('connectionStatus'),
            totalAircraft: document.getElementById('totalAircraft'),
            militaryCount: document.getElementById('militaryCount'),
            closestDistance: document.getElementById('closestDistance'),
            closestPanel: document.getElementById('closestPanel'),
            closestIcon: document.getElementById('closestIcon'),
            closestCallsign: document.getElementById('closestCallsign'),
            closestType: document.getElementById('closestType'),
            closestAlt: document.getElementById('closestAlt'),
            closestSpeed: document.getElementById('closestSpeed'),
            militaryPanel: document.getElementById('militaryPanel'),
            militaryList: document.getElementById('militaryList')
        };

        function updateDisplay() {
            const aircraftList = Array.from(aircraft.values());

            // Total count
            elements.totalAircraft.textContent = aircraftList.length;

            // Military count
            const militaryAircraft = aircraftList.filter(a => a.military);
            elements.militaryCount.textContent = militaryAircraft.length;

            // Closest aircraft
            const sorted = aircraftList
                .filter(a => a.distance != null)
                .sort((a, b) => a.distance - b.distance);

            if (sorted.length > 0) {
                const closest = sorted[0];
                elements.closestDistance.textContent = closest.distance.toFixed(1);
                elements.closestCallsign.textContent = closest.flight || closest.hex;
                elements.closestType.textContent = closest.type || 'Unknown';
                elements.closestAlt.textContent = (closest.alt || 0).toLocaleString();
                elements.closestSpeed.textContent = closest.gs || 0;
                elements.closestIcon.style.transform = `rotate(${closest.track || 0}deg)`;
                elements.closestPanel.classList.remove('hidden');
            } else {
                elements.closestPanel.classList.add('hidden');
            }

            // Military alerts
            if (CONFIG.showMilitaryAlerts && militaryAircraft.length > 0) {
                elements.militaryPanel.classList.remove('hidden');
                elements.militaryList.innerHTML = militaryAircraft
                    .slice(0, CONFIG.maxMilitaryDisplay)
                    .map(a => `
                        <div class="military-item">
                            <span class="military-badge">🎖️</span>
                            <div>
                                <div class="military-callsign">${a.flight || a.hex}</div>
                                <div class="military-type">${a.type || 'Unknown'} • ${(a.distance || 0).toFixed(1)} nm</div>
                            </div>
                        </div>
                    `).join('');
            } else {
                elements.militaryPanel.classList.add('hidden');
            }
        }

        function removeStaleAircraft() {
            const now = Date.now();
            for (const [hex, ac] of aircraft) {
                if (now - ac._lastSeen > CONFIG.staleTimeout) {
                    aircraft.delete(hex);
                }
            }
            updateDisplay();
        }

        function connect() {
            const es = new EventSource(`${CONFIG.skyspyUrl}/api/v1/map/sse`);

            es.onopen = () => {
                isConnected = true;
                elements.connectionStatus.classList.add('connected');
            };

            es.onerror = () => {
                isConnected = false;
                elements.connectionStatus.classList.remove('connected');
                es.close();
                // Reconnect after 5 seconds
                setTimeout(connect, 5000);
            };

            es.addEventListener('aircraft_update', (e) => {
                const data = JSON.parse(e.data);
                for (const ac of data.aircraft || []) {
                    ac._lastSeen = Date.now();
                    aircraft.set(ac.hex, ac);
                }
                updateDisplay();
            });

            es.addEventListener('aircraft_new', (e) => {
                const data = JSON.parse(e.data);
                for (const ac of data.aircraft || []) {
                    ac._lastSeen = Date.now();
                    aircraft.set(ac.hex, ac);
                }
                updateDisplay();
            });
        }

        // Initialize
        connect();
        setInterval(removeStaleAircraft, 10000);
    </script>
</body>
</html>
```

```css Custom Themes
/* Dark theme (default) - included in HTML above */

/* Light theme overlay */
.panel {
    background: rgba(255, 255, 255, 0.9);
    color: #1e293b;
    border-left-color: #3b82f6;
}

.panel-header {
    color: #64748b;
}

.stat-label {
    color: #64748b;
}

/* Minimal theme - just stats */
.closest-panel,
.military-alert {
    display: none;
}

.stats-panel {
    background: rgba(0, 0, 0, 0.5);
    padding: 10px;
}

/* Compact corner theme */
.overlay-container {
    bottom: 10px;
    left: 10px;
    max-width: 200px;
}

.stat-value {
    font-size: 24px;
}
```

```json Response Example
{
  "aircraft": [
    {
      "hex": "A12345",
      "flight": "UAL123",
      "type": "B738",
      "alt": 35000,
      "gs": 450,
      "track": 270,
      "distance": 12.4,
      "military": false
    },
    {
      "hex": "AE1234",
      "flight": "RCH419",
      "type": "C17",
      "alt": 28000,
      "gs": 380,
      "track": 180,
      "distance": 15.2,
      "military": true
    }
  ]
}
```

## Create the Overlay HTML

<!-- html@1-70 -->

The overlay uses a transparent background with semi-transparent panels. The CSS is designed to look good over any stream content.

### Key Styling Features

- **Transparent background** - Works as OBS browser source
- **Backdrop blur** - Subtle blur behind panels for readability
- **Color-coded stats** - Blue for total, orange for military, green for closest
- **Animated military alerts** - Subtle pulse animation draws attention

## Connect to SkySpy SSE

<!-- html@170-220 -->

The JavaScript connects to SkySpy's Server-Sent Events (SSE) endpoint for real-time updates. It handles reconnection automatically if the connection drops.

## Adding to OBS

### Step 1: Create Browser Source

1. Open OBS Studio
2. In your scene, click **+** under Sources
3. Select **Browser**
4. Name it "SkySpy Overlay"

### Step 2: Configure Browser Source

| Setting | Value |
|---------|-------|
| URL | `file:///path/to/overlay.html` or serve via HTTP |
| Width | 1920 (match your canvas) |
| Height | 1080 (match your canvas) |
| Custom CSS | Leave empty |
| Shutdown source when not visible | Recommended |
| Refresh browser when scene becomes active | Recommended |

### Step 3: Position the Overlay

1. The overlay positions itself in the bottom-left by default
2. Resize/move the browser source if needed
3. Use Edit > Transform to adjust

### Using URL Parameters

Instead of editing the HTML file, pass configuration via URL parameters:

```
file:///path/to/overlay.html?url=http://192.168.1.100:5000&military=true
```

Or if serving via HTTP:

```
http://localhost:8080/overlay.html?url=http://skyspy.local:5000
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `skyspyUrl` | SkySpy server URL | `http://localhost:5000` |
| `showMilitaryAlerts` | Show military aircraft panel | `true` |
| `maxMilitaryDisplay` | Max military aircraft to show | `3` |
| `staleTimeout` | Remove aircraft after ms | `60000` |

### URL Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `url` | SkySpy server URL | `?url=http://192.168.1.100:5000` |
| `military` | Show military alerts | `?military=false` |

## Customization Options

### Change Position

Edit the `.overlay-container` CSS:

```css
/* Bottom-left (default) */
.overlay-container {
    bottom: 20px;
    left: 20px;
}

/* Bottom-right */
.overlay-container {
    bottom: 20px;
    right: 20px;
    left: auto;
}

/* Top-left */
.overlay-container {
    top: 20px;
    left: 20px;
    bottom: auto;
}

/* Top-right */
.overlay-container {
    top: 20px;
    right: 20px;
    bottom: auto;
    left: auto;
}
```

### Change Colors

Edit the color variables in the CSS:

```css
/* Panel accent colors */
.panel { border-left-color: #3b82f6; }           /* Blue */
.closest-panel { border-left-color: #22c55e; }   /* Green */
.military-alert { border-left-color: #f59e0b; }  /* Orange */

/* Stat value colors */
.stat-aircraft .stat-value { color: #3b82f6; }   /* Blue */
.stat-military .stat-value { color: #f59e0b; }   /* Orange */
.stat-closest .stat-value { color: #22c55e; }    /* Green */
```

### Change Size

```css
/* Smaller overlay */
.overlay-container {
    max-width: 300px;
    transform: scale(0.8);
    transform-origin: bottom left;
}

/* Larger overlay */
.overlay-container {
    max-width: 500px;
    transform: scale(1.2);
    transform-origin: bottom left;
}
```

### Custom Fonts

Add a Google Font:

```html
<head>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Orbitron', sans-serif;
        }
    </style>
</head>
```

### Hide Panels

```css
/* Hide closest aircraft panel */
.closest-panel { display: none; }

/* Hide military alerts */
.military-alert { display: none; }

/* Show only stats */
.closest-panel, .military-alert { display: none; }
```

## Testing & Verification

### Local Testing

1. **Open in browser** - Open `overlay.html` directly in Chrome/Firefox
2. **Check connection** - Green dot in top-right indicates connected
3. **Verify data** - Aircraft count should update when SkySpy has data
4. **Test transparency** - Background should be transparent

### OBS Testing

1. **Add browser source** with the overlay URL
2. **Check rendering** - Overlay should appear with transparent background
3. **Verify updates** - Stats should update in real-time
4. **Test visibility** - Toggle source visibility to test refresh

### Serving via HTTP

For remote SkySpy servers or team setups:

```shell
# Simple Python server
cd skyspy-obs-overlay
python -m http.server 8080

# Then use URL: http://localhost:8080/overlay.html
```

## Troubleshooting

### Overlay Not Transparent in OBS

**Problem:** Background shows as white or colored instead of transparent
**Solution:**
- Ensure the HTML has `background: transparent` on body
- In OBS browser source, make sure custom CSS is empty
- Try adding `--disable-gpu` to OBS launch options if using hardware acceleration

### No Data Showing

**Problem:** Connection dot is red, no aircraft data
**Solution:**
- Verify SkySpy is running and accessible
- Check the `skyspyUrl` in the config or URL parameter
- Open browser console (F12) to check for errors
- Test the SSE endpoint directly: `curl http://localhost:5000/api/v1/map/sse`

### CORS Errors

**Problem:** Browser console shows CORS errors
**Solution:**
- Serve the HTML file via HTTP instead of file://
- Configure SkySpy to allow CORS from your origin
- Use the same host for both overlay and SkySpy

### Overlay Looks Blurry

**Problem:** Text and panels appear blurry in OBS
**Solution:**
- Match browser source dimensions to your canvas (1920x1080)
- Avoid scaling the browser source in OBS
- Use whole-pixel positioning (no decimals in transform)

### High CPU Usage

**Problem:** OBS CPU usage spikes with overlay
**Solution:**
- Enable "Shutdown source when not visible"
- Reduce update frequency by adding throttling
- Use "Refresh browser when scene becomes active" instead of continuous

### Military Alerts Not Showing

**Problem:** Military aircraft exist but alert panel is hidden
**Solution:**
- Check `showMilitaryAlerts` is set to `true`
- Verify aircraft have `military: true` in the data
- Check `maxMilitaryDisplay` is greater than 0

## Related Recipes

- [Terminal Dashboard](/docs/terminal-dashboard) - CLI terminal interface
- [Leaflet.js Map](/docs/leaflet-map) - Interactive web map
- [Build a Live Dashboard](/docs/live-dashboard) - Full web dashboard
- [Discord Alert Bot](/docs/discord-alert-bot) - Alert notifications
