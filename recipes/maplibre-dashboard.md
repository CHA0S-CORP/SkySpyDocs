---
title: MapLibre Dashboard
excerpt: Open-source map visualization with MapLibre GL
hidden: false
recipe:
  color: '#4264FB'
  icon: 🗺️
difficulty: intermediate
tags: [visualization, map, maplibre, dashboard]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Web browser with WebGL support
- Basic HTML/CSS/JavaScript knowledge

## What You'll Build

An interactive map dashboard using MapLibre GL JS, a free open-source fork of Mapbox GL. Features include rotated aircraft markers, popup information panels, flight trails, and smooth animations.

```shell Shell
# Create project directory
mkdir skyspy-maplibre && cd skyspy-maplibre

# Create the HTML file (content below)
# Then open index.html in your browser
# Or serve with a local server for best results
python -m http.server 8080
```

```html HTML (index.html)
<!DOCTYPE html>
<html>
<head>
    <title>SkySpy MapLibre Dashboard</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://unpkg.com/maplibre-gl@3.6.2/dist/maplibre-gl.css" rel="stylesheet" />
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: system-ui, -apple-system, sans-serif; }
        #map { width: 100vw; height: 100vh; }

        .dashboard-panel {
            position: absolute;
            top: 10px;
            right: 10px;
            background: rgba(255, 255, 255, 0.95);
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
            z-index: 1000;
            min-width: 200px;
        }

        .dashboard-panel h2 {
            font-size: 14px;
            color: #666;
            margin-bottom: 10px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .stat-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 8px 0;
            border-bottom: 1px solid #eee;
        }

        .stat-row:last-child { border-bottom: none; }
        .stat-label { color: #666; }
        .stat-value { font-size: 24px; font-weight: bold; color: #333; }
        .stat-military { color: #f59e0b; }

        .aircraft-marker {
            width: 24px;
            height: 24px;
            cursor: pointer;
        }

        .maplibregl-popup-content {
            padding: 15px;
            border-radius: 8px;
            font-family: system-ui;
        }

        .popup-title {
            font-size: 16px;
            font-weight: bold;
            margin-bottom: 8px;
        }

        .popup-row {
            display: flex;
            justify-content: space-between;
            padding: 4px 0;
            font-size: 13px;
        }

        .popup-label { color: #666; }
        .popup-value { font-weight: 500; }
        .military-badge {
            background: #f59e0b;
            color: white;
            padding: 2px 8px;
            border-radius: 4px;
            font-size: 11px;
            margin-top: 8px;
            display: inline-block;
        }
    </style>
</head>
<body>
    <div id="map"></div>
    <div class="dashboard-panel">
        <h2>Aircraft Tracker</h2>
        <div class="stat-row">
            <span class="stat-label">Total Aircraft</span>
            <span id="total-count" class="stat-value">0</span>
        </div>
        <div class="stat-row">
            <span class="stat-label">Military</span>
            <span id="military-count" class="stat-value stat-military">0</span>
        </div>
        <div class="stat-row">
            <span class="stat-label">With Position</span>
            <span id="position-count" class="stat-value">0</span>
        </div>
    </div>

    <script src="https://unpkg.com/maplibre-gl@3.6.2/dist/maplibre-gl.js"></script>
    <script>
        const SKYSPY_URL = 'http://localhost:5000';
        const CENTER = [-74.0060, 40.7128]; // [lon, lat] for MapLibre
        const ZOOM = 9;

        // Initialize map
        const map = new maplibregl.Map({
            container: 'map',
            style: {
                version: 8,
                sources: {
                    'osm': {
                        type: 'raster',
                        tiles: ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'],
                        tileSize: 256,
                        attribution: '&copy; OpenStreetMap contributors'
                    }
                },
                layers: [{
                    id: 'osm-tiles',
                    type: 'raster',
                    source: 'osm',
                    minzoom: 0,
                    maxzoom: 19
                }]
            },
            center: CENTER,
            zoom: ZOOM
        });

        // Add navigation controls
        map.addControl(new maplibregl.NavigationControl(), 'top-left');
        map.addControl(new maplibregl.ScaleControl(), 'bottom-left');

        // Store markers and trails
        const markers = new Map();
        const trails = new Map();
        const trailData = new Map();

        // Create SVG aircraft icon
        function createAircraftSvg(rotation, isMilitary) {
            const color = isMilitary ? '#f59e0b' : '#3b82f6';
            return `
                <svg width="24" height="24" viewBox="0 0 24 24" style="transform: rotate(${rotation}deg)">
                    <path fill="${color}" stroke="#fff" stroke-width="1" d="M12 2L4 12l8 2 8-2L12 2zM12 14l-4-1v4l4 5 4-5v-4l-4 1z"/>
                </svg>
            `;
        }

        // Create popup content
        function createPopupContent(aircraft) {
            const militaryBadge = aircraft.military
                ? '<span class="military-badge">MILITARY</span>'
                : '';

            return `
                <div class="popup-title">${aircraft.flight || aircraft.hex}</div>
                <div class="popup-row">
                    <span class="popup-label">ICAO</span>
                    <span class="popup-value">${aircraft.hex}</span>
                </div>
                <div class="popup-row">
                    <span class="popup-label">Type</span>
                    <span class="popup-value">${aircraft.type || 'Unknown'}</span>
                </div>
                <div class="popup-row">
                    <span class="popup-label">Altitude</span>
                    <span class="popup-value">${(aircraft.alt || 0).toLocaleString()} ft</span>
                </div>
                <div class="popup-row">
                    <span class="popup-label">Speed</span>
                    <span class="popup-value">${aircraft.gs || 0} kts</span>
                </div>
                <div class="popup-row">
                    <span class="popup-label">Heading</span>
                    <span class="popup-value">${aircraft.track || 0}°</span>
                </div>
                ${militaryBadge}
            `;
        }

        // Update or create aircraft marker
        function updateAircraft(aircraft) {
            if (!aircraft.lat || !aircraft.lon) return;

            const existing = markers.get(aircraft.hex);
            const rotation = aircraft.track || 0;

            if (existing) {
                // Update existing marker
                existing.setLngLat([aircraft.lon, aircraft.lat]);
                existing.getElement().innerHTML = createAircraftSvg(rotation, aircraft.military);
                existing.setPopup(new maplibregl.Popup({ offset: 15 })
                    .setHTML(createPopupContent(aircraft)));
            } else {
                // Create new marker
                const el = document.createElement('div');
                el.className = 'aircraft-marker';
                el.innerHTML = createAircraftSvg(rotation, aircraft.military);

                const marker = new maplibregl.Marker({ element: el })
                    .setLngLat([aircraft.lon, aircraft.lat])
                    .setPopup(new maplibregl.Popup({ offset: 15 })
                        .setHTML(createPopupContent(aircraft)))
                    .addTo(map);

                markers.set(aircraft.hex, marker);
            }

            // Update trail
            updateTrail(aircraft);
        }

        // Update aircraft trail
        function updateTrail(aircraft) {
            const hex = aircraft.hex;
            const coords = trailData.get(hex) || [];

            coords.push([aircraft.lon, aircraft.lat]);

            // Keep last 100 points
            if (coords.length > 100) {
                coords.shift();
            }

            trailData.set(hex, coords);

            const sourceId = `trail-${hex}`;
            const layerId = `trail-layer-${hex}`;

            if (map.getSource(sourceId)) {
                map.getSource(sourceId).setData({
                    type: 'Feature',
                    geometry: {
                        type: 'LineString',
                        coordinates: coords
                    }
                });
            } else if (coords.length > 1) {
                map.addSource(sourceId, {
                    type: 'geojson',
                    data: {
                        type: 'Feature',
                        geometry: {
                            type: 'LineString',
                            coordinates: coords
                        }
                    }
                });

                map.addLayer({
                    id: layerId,
                    type: 'line',
                    source: sourceId,
                    paint: {
                        'line-color': aircraft.military ? '#f59e0b' : '#3b82f6',
                        'line-width': 2,
                        'line-opacity': 0.6
                    }
                });

                trails.set(hex, { sourceId, layerId });
            }
        }

        // Remove stale aircraft
        function removeStale() {
            const now = Date.now();
            for (const [hex, marker] of markers) {
                if (now - (marker._lastUpdate || 0) > 60000) {
                    marker.remove();
                    markers.delete(hex);

                    // Remove trail
                    const trail = trails.get(hex);
                    if (trail) {
                        if (map.getLayer(trail.layerId)) {
                            map.removeLayer(trail.layerId);
                        }
                        if (map.getSource(trail.sourceId)) {
                            map.removeSource(trail.sourceId);
                        }
                        trails.delete(hex);
                        trailData.delete(hex);
                    }
                }
            }
        }

        // Connect to SSE when map is ready
        map.on('load', () => {
            const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

            es.addEventListener('aircraft_update', (e) => {
                const data = JSON.parse(e.data);
                let militaryCount = 0;
                let positionCount = 0;

                for (const aircraft of data.aircraft || []) {
                    updateAircraft(aircraft);

                    const marker = markers.get(aircraft.hex);
                    if (marker) {
                        marker._lastUpdate = Date.now();
                        positionCount++;
                    }
                    if (aircraft.military) militaryCount++;
                }

                document.getElementById('total-count').textContent = data.aircraft?.length || 0;
                document.getElementById('military-count').textContent = militaryCount;
                document.getElementById('position-count').textContent = positionCount;
            });

            es.onerror = () => {
                console.error('SSE connection error');
            };
        });

        // Cleanup every 30 seconds
        setInterval(removeStale, 30000);
    </script>
</body>
</html>
```

```javascript JavaScript (Vanilla)
// Standalone JavaScript version for integration
class MapLibreAircraftMap {
    constructor(container, options = {}) {
        this.skyspyUrl = options.skyspyUrl || 'http://localhost:5000';
        this.center = options.center || [-74.0060, 40.7128];
        this.zoom = options.zoom || 9;
        this.markers = new Map();
        this.trails = new Map();
        this.trailData = new Map();

        this.map = new maplibregl.Map({
            container,
            style: options.style || this.getDefaultStyle(),
            center: this.center,
            zoom: this.zoom
        });

        this.map.on('load', () => this.connect());
    }

    getDefaultStyle() {
        return {
            version: 8,
            sources: {
                osm: {
                    type: 'raster',
                    tiles: ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'],
                    tileSize: 256
                }
            },
            layers: [{ id: 'osm', type: 'raster', source: 'osm' }]
        };
    }

    connect() {
        const es = new EventSource(`${this.skyspyUrl}/api/v1/map/sse`);
        es.addEventListener('aircraft_update', (e) => {
            const data = JSON.parse(e.data);
            for (const aircraft of data.aircraft || []) {
                this.updateAircraft(aircraft);
            }
        });
    }

    updateAircraft(aircraft) {
        if (!aircraft.lat || !aircraft.lon) return;
        // Implementation similar to HTML version
    }
}

// Usage
const aircraftMap = new MapLibreAircraftMap('map', {
    skyspyUrl: 'http://localhost:5000',
    center: [-74.0060, 40.7128],
    zoom: 10
});
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
      "gs": 450,
      "track": 270,
      "type": "B738",
      "military": false
    },
    {
      "hex": "AE1234",
      "flight": "RCH501",
      "lat": 40.8000,
      "lon": -73.9500,
      "alt": 28000,
      "gs": 380,
      "track": 45,
      "type": "C17",
      "military": true
    }
  ]
}
```

## Initialize MapLibre

<!-- html@90-120 -->

MapLibre GL JS uses WebGL for smooth, GPU-accelerated rendering. The style object defines both tile sources and layer configurations.

### Alternative Tile Sources

```javascript
// OpenStreetMap (free, no API key)
{
    type: 'raster',
    tiles: ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'],
    tileSize: 256
}

// Carto Dark (free, great for dashboards)
{
    type: 'raster',
    tiles: ['https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png'],
    tileSize: 256
}

// Stadia Maps (free tier available)
{
    type: 'raster',
    tiles: ['https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}.png'],
    tileSize: 256
}

// MapTiler (vector tiles, requires API key)
`https://api.maptiler.com/maps/streets/style.json?key=YOUR_KEY`
```

## Create Aircraft Markers with Rotation

<!-- html@125-145 -->

MapLibre markers use HTML elements, allowing full CSS control including smooth rotation via SVG transforms.

### Custom Aircraft Icons by Type

```javascript
function getAircraftIcon(aircraft) {
    const icons = {
        helicopter: `<path d="M12 4v4m-4-2h8M8 12h8l-4 8-4-8z"/>`,
        jet: `<path d="M12 2L4 12l8 2 8-2L12 2zM12 14l-4-1v4l4 5 4-5v-4l-4 1z"/>`,
        prop: `<path d="M12 3L6 12l6 2 6-2L12 3zM12 14l-3-1v3l3 4 3-4v-3l-3 1z"/>`,
        default: `<path d="M12 2L4 12l8 2 8-2L12 2z"/>`
    };

    const type = aircraft.type?.toLowerCase() || '';
    let iconPath = icons.default;

    if (type.includes('h60') || type.includes('heli')) iconPath = icons.helicopter;
    else if (type.includes('c17') || type.includes('kc')) iconPath = icons.jet;
    else if (type.includes('c130')) iconPath = icons.prop;

    const color = aircraft.military ? '#f59e0b' : '#3b82f6';
    return `<svg width="24" height="24" viewBox="0 0 24 24"
        style="transform: rotate(${aircraft.track || 0}deg)">
        ${iconPath.replace('d=', `fill="${color}" stroke="#fff" stroke-width="1" d=`)}
    </svg>`;
}
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `CENTER` | Map center [lon, lat] | `[-74.0060, 40.7128]` |
| `ZOOM` | Initial zoom level | `9` |

### Dark Mode Style

```javascript
const darkStyle = {
    version: 8,
    sources: {
        'carto-dark': {
            type: 'raster',
            tiles: ['https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png'],
            tileSize: 256
        }
    },
    layers: [{
        id: 'carto-dark-tiles',
        type: 'raster',
        source: 'carto-dark'
    }]
};

// Update CSS for dark mode
document.documentElement.style.setProperty('--panel-bg', 'rgba(30, 30, 30, 0.95)');
document.documentElement.style.setProperty('--text-color', '#fff');
```

### Altitude-Based Trail Colors

```javascript
function getAltitudeColor(altitude) {
    if (altitude < 5000) return '#22c55e';   // Green - low
    if (altitude < 15000) return '#eab308';  // Yellow - medium
    if (altitude < 30000) return '#f97316';  // Orange - high
    return '#ef4444';                         // Red - cruise altitude
}

// Apply to trail layer
map.setPaintProperty(`trail-layer-${hex}`, 'line-color', getAltitudeColor(aircraft.alt));
```

## Testing & Verification

1. **Open the HTML file** via a local server (recommended)
2. **Check WebGL support** - map should render smoothly
3. **Verify aircraft markers** appear with correct rotation
4. **Test popups** by clicking on aircraft
5. **Watch trails** form as aircraft move
6. **Check dashboard stats** update in real-time

### WebGL Check

```javascript
const canvas = document.createElement('canvas');
const hasWebGL = !!(canvas.getContext('webgl') || canvas.getContext('webgl2'));
if (!hasWebGL) {
    alert('WebGL is required for MapLibre GL');
}
```

## Troubleshooting

### Map Not Rendering

**Problem:** Blank screen or no map tiles
**Solution:**
- Verify WebGL is enabled in your browser
- Check browser console for errors
- Ensure tile URL is accessible (try in new tab)
- Check for CORS issues with tile server

### Aircraft Not Appearing

**Problem:** Map loads but no aircraft markers
**Solution:**
- Check SkySpy URL is correct
- Verify SSE connection in Network tab
- Look for CORS errors in console
- Ensure aircraft have lat/lon data

### Performance Issues

**Problem:** Map stutters with many aircraft
**Solution:**
- Use clustering for dense areas:
  ```javascript
  map.addSource('aircraft', {
      type: 'geojson',
      data: featureCollection,
      cluster: true,
      clusterRadius: 50
  });
  ```
- Reduce trail point count
- Use `requestAnimationFrame` for updates

### Trails Not Showing

**Problem:** Aircraft move but no trails appear
**Solution:**
- Ensure `map.on('load')` fires before adding sources
- Check that trail coordinates array has > 1 point
- Verify layer is added after source

## Related Recipes

- [Leaflet.js Map](/docs/leaflet-map) - Lighter weight alternative
- [Build a Live Dashboard](/docs/live-dashboard) - Full dashboard with table
- [OBS Browser Source](/docs/obs-overlay) - Stream overlay
