---
title: Leaflet.js Map
excerpt: Display aircraft on an interactive web map.
hidden: false
recipe:
  color: '#199900'
  icon: 🗺️
difficulty: beginner
tags: [visualization, map, leaflet, web]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Web browser with JavaScript enabled
- Basic HTML/CSS knowledge helpful

## What You'll Build

An interactive web map showing real-time aircraft positions with icons, popups, and trails. Lighter weight than full mapping libraries like MapLibre or Deck.gl.

```shell Shell
# Create project directory
mkdir skyspy-map && cd skyspy-map

# Create the HTML file (content below)
# Then open index.html in your browser
```

```html HTML (index.html)
<!DOCTYPE html>
<html>
<head>
    <title>SkySpy Map</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        body { margin: 0; padding: 0; }
        #map { width: 100vw; height: 100vh; }
        .aircraft-icon {
            font-size: 20px;
            text-shadow: 2px 2px 2px rgba(0,0,0,0.5);
        }
        .military { color: #f59e0b; }
        .civilian { color: #3b82f6; }
        .info-panel {
            position: absolute;
            top: 10px;
            right: 10px;
            background: rgba(255,255,255,0.9);
            padding: 15px;
            border-radius: 8px;
            z-index: 1000;
            font-family: system-ui;
        }
        .stat { font-size: 24px; font-weight: bold; }
    </style>
</head>
<body>
    <div id="map"></div>
    <div class="info-panel">
        <div>Aircraft: <span id="count" class="stat">0</span></div>
        <div>Military: <span id="military" class="stat">0</span></div>
    </div>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
        const SKYSPY_URL = 'http://localhost:5000';

        // Initialize map
        const map = L.map('map').setView([40.7128, -74.0060], 10);

        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors'
        }).addTo(map);

        const aircraftMarkers = new Map();

        function createIcon(aircraft) {
            const rotation = aircraft.track || 0;
            const color = aircraft.military ? 'military' : 'civilian';
            return L.divIcon({
                html: `<div class="aircraft-icon ${color}" style="transform: rotate(${rotation}deg)">✈</div>`,
                iconSize: [24, 24],
                iconAnchor: [12, 12],
                className: ''
            });
        }

        function updateAircraft(aircraft) {
            if (!aircraft.lat || !aircraft.lon) return;

            const marker = aircraftMarkers.get(aircraft.hex);

            if (marker) {
                marker.setLatLng([aircraft.lat, aircraft.lon]);
                marker.setIcon(createIcon(aircraft));
            } else {
                const newMarker = L.marker([aircraft.lat, aircraft.lon], {
                    icon: createIcon(aircraft)
                }).addTo(map);

                newMarker.bindPopup(`
                    <b>${aircraft.flight || aircraft.hex}</b><br>
                    Type: ${aircraft.type || 'Unknown'}<br>
                    Altitude: ${(aircraft.alt || 0).toLocaleString()} ft<br>
                    Speed: ${aircraft.gs || 0} kts<br>
                    ${aircraft.military ? '🎖️ Military' : ''}
                `);

                aircraftMarkers.set(aircraft.hex, newMarker);
            }
        }

        function removeStale() {
            const now = Date.now();
            for (const [hex, marker] of aircraftMarkers) {
                if (now - marker._lastUpdate > 60000) {
                    map.removeLayer(marker);
                    aircraftMarkers.delete(hex);
                }
            }
        }

        // Connect to SSE
        const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

        es.addEventListener('aircraft_update', (e) => {
            const data = JSON.parse(e.data);
            let militaryCount = 0;

            for (const aircraft of data.aircraft || []) {
                updateAircraft(aircraft);
                const marker = aircraftMarkers.get(aircraft.hex);
                if (marker) marker._lastUpdate = Date.now();
                if (aircraft.military) militaryCount++;
            }

            document.getElementById('count').textContent = aircraftMarkers.size;
            document.getElementById('military').textContent = militaryCount;
        });

        // Cleanup every 30 seconds
        setInterval(removeStale, 30000);
    </script>
</body>
</html>
```

```python Python (Flask server)
from flask import Flask, render_template_string

app = Flask(__name__)

HTML = """
<!DOCTYPE html>
<html>
<head>
    <title>SkySpy Map</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        body { margin: 0; }
        #map { width: 100vw; height: 100vh; }
    </style>
</head>
<body>
    <div id="map"></div>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
        const map = L.map('map').setView([{{ lat }}, {{ lon }}], 10);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

        const markers = new Map();
        const es = new EventSource('{{ skyspy_url }}/api/v1/map/sse');

        es.addEventListener('aircraft_update', (e) => {
            const data = JSON.parse(e.data);
            for (const ac of data.aircraft || []) {
                if (!ac.lat || !ac.lon) continue;
                if (markers.has(ac.hex)) {
                    markers.get(ac.hex).setLatLng([ac.lat, ac.lon]);
                } else {
                    markers.set(ac.hex, L.marker([ac.lat, ac.lon]).addTo(map)
                        .bindPopup(`${ac.flight || ac.hex}<br>${ac.type || ''}`));
                }
            }
        });
    </script>
</body>
</html>
"""

@app.route("/")
def index():
    return render_template_string(HTML,
        skyspy_url="http://localhost:5000",
        lat=40.7128,
        lon=-74.0060
    )

if __name__ == "__main__":
    app.run(port=8080)
```

```javascript JavaScript (React Component)
import { useEffect, useRef, useState } from 'react';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

export default function AircraftMap({ skyspyUrl = 'http://localhost:5000' }) {
  const mapRef = useRef(null);
  const markersRef = useRef(new Map());
  const [count, setCount] = useState(0);

  useEffect(() => {
    // Initialize map
    const map = L.map(mapRef.current).setView([40.7128, -74.0060], 10);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

    // Connect to SSE
    const es = new EventSource(`${skyspyUrl}/api/v1/map/sse`);

    es.addEventListener('aircraft_update', (e) => {
      const data = JSON.parse(e.data);

      for (const aircraft of data.aircraft || []) {
        if (!aircraft.lat || !aircraft.lon) continue;

        const existing = markersRef.current.get(aircraft.hex);
        if (existing) {
          existing.setLatLng([aircraft.lat, aircraft.lon]);
        } else {
          const marker = L.marker([aircraft.lat, aircraft.lon])
            .addTo(map)
            .bindPopup(`${aircraft.flight || aircraft.hex}`);
          markersRef.current.set(aircraft.hex, marker);
        }
      }

      setCount(markersRef.current.size);
    });

    return () => {
      es.close();
      map.remove();
    };
  }, [skyspyUrl]);

  return (
    <div>
      <div ref={mapRef} style={{ width: '100%', height: '500px' }} />
      <p>Aircraft: {count}</p>
    </div>
  );
}
```

```json Response Example
{
  "aircraft": [
    {
      "hex": "A12345",
      "flight": "UAL123",
      "lat": 40.7128,
      "lon": -74.0060,
      "alt": 35000,
      "track": 270,
      "military": false
    }
  ]
}
```

## Initialize Map

<!-- html@25-35 -->

Create a Leaflet map centered on your location. OpenStreetMap tiles are free and don't require an API key.

### Alternative Tile Providers

```javascript
// Carto Light (minimal)
L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png')

// Carto Dark (for dark mode)
L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png')

// Satellite (requires Mapbox token)
L.tileLayer('https://api.mapbox.com/styles/v1/mapbox/satellite-v9/tiles/{z}/{x}/{y}?access_token=YOUR_TOKEN')
```

## Create Aircraft Markers

<!-- html@37-52 -->

Each aircraft gets a rotated plane icon. Military aircraft are colored orange, civilians blue.

### Custom SVG Icons

```javascript
function createSvgIcon(aircraft) {
  const color = aircraft.military ? '#f59e0b' : '#3b82f6';
  const svg = `
    <svg width="24" height="24" viewBox="0 0 24 24" style="transform: rotate(${aircraft.track}deg)">
      <path fill="${color}" d="M12 2L4 12l8 2 8-2L12 2z"/>
    </svg>
  `;
  return L.divIcon({
    html: svg,
    iconSize: [24, 24],
    className: ''
  });
}
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| Initial lat/lon | Map center | Your location |
| Zoom level | Initial zoom | `10` |

### Add Aircraft Trails

Track aircraft paths over time:

```javascript
const trails = new Map();

function updateTrail(aircraft) {
  const trail = trails.get(aircraft.hex) || L.polyline([], {
    color: aircraft.military ? '#f59e0b' : '#3b82f6',
    weight: 2,
    opacity: 0.6
  }).addTo(map);

  trail.addLatLng([aircraft.lat, aircraft.lon]);

  // Keep last 100 points
  if (trail.getLatLngs().length > 100) {
    trail.setLatLngs(trail.getLatLngs().slice(-100));
  }

  trails.set(aircraft.hex, trail);
}
```

### Altitude-Based Colors

```javascript
function getAltitudeColor(altitude) {
  if (altitude < 5000) return '#22c55e';   // Green - low
  if (altitude < 15000) return '#eab308';  // Yellow - medium
  if (altitude < 30000) return '#f97316';  // Orange - high
  return '#ef4444';                         // Red - very high
}
```

## Testing & Verification

1. **Open the HTML file** in a browser
2. **Check map loads** with tiles displaying
3. **Verify aircraft appear** as plane icons
4. **Click markers** to see popup info
5. **Watch icons rotate** as aircraft change heading

### CORS Issues

If running locally, you may need to:
- Serve the file via a local server
- Or configure SkySpy to allow CORS

```shell
# Simple Python server
python -m http.server 8080
# Then open http://localhost:8080
```

## Troubleshooting

### No Aircraft Showing

**Problem:** Map loads but no planes appear
**Solution:**
- Check browser console for errors
- Verify SkySpy URL is correct
- Check for CORS errors

### Icons Not Rotating

**Problem:** All planes point the same direction
**Solution:**
- Verify `track` field is present in data
- Check CSS transform is being applied
- Some aircraft don't report track

### Map Performance Issues

**Problem:** Map becomes slow with many aircraft
**Solution:**
- Use marker clustering:
  ```javascript
  const markers = L.markerClusterGroup();
  map.addLayer(markers);
  ```
- Limit visible aircraft
- Reduce update frequency

## Related Recipes

- [Build a Live Dashboard](/docs/live-dashboard) - Full dashboard with table
- [MapLibre Dashboard](/docs/maplibre-dashboard) - Vector tiles
- [MapLibre Dashboard](/docs/maplibre-dashboard) - Vector tiles and advanced mapping
- [OBS Browser Source](/docs/obs-overlay) - Stream overlay
