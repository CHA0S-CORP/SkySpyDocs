---
title: Build a Live Dashboard
excerpt: Create a real-time aircraft monitoring dashboard with React and SSE.
hidden: false
recipe:
  color: '#10B981'
  icon: 🖥️
difficulty: intermediate
tags: [visualization, react, dashboard, sse]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Node.js 18+ (for React dashboard)
- Python 3.8+ with Flask (for Python dashboard)
- Go 1.19+ (for Go dashboard)

## What You'll Build

A real-time dashboard that displays aircraft currently in your coverage area. The dashboard updates automatically as new aircraft appear and existing ones update their positions.

```shell Shell
npm create vite@latest skyspy-dashboard -- --template react-ts
cd skyspy-dashboard
npm install
npm run dev
```

```go Go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "html/template"
    "net/http"
    "os"
    "strings"
    "sync"
    "time"
)

type Aircraft struct {
    Hex      string  `json:"hex"`
    Flight   string  `json:"flight"`
    Type     string  `json:"type"`
    Alt      int     `json:"alt"`
    Speed    int     `json:"gs"`
    Track    float64 `json:"track"`
    Lat      float64 `json:"lat"`
    Lon      float64 `json:"lon"`
    Distance float64 `json:"distance"`
    LastSeen time.Time
}

var (
    aircraft = make(map[string]*Aircraft)
    mu       sync.RWMutex
    skyspyURL string
)

const dashboardHTML = `
<!DOCTYPE html>
<html>
<head>
    <title>SkySpy Dashboard</title>
    <meta http-equiv="refresh" content="5">
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; margin: 20px; background: #1a1a2e; color: #eee; }
        h1 { color: #10b981; }
        .stats { display: flex; gap: 20px; margin-bottom: 20px; }
        .stat { background: #16213e; padding: 15px 25px; border-radius: 8px; }
        .stat-value { font-size: 2em; font-weight: bold; color: #10b981; }
        table { width: 100%; border-collapse: collapse; background: #16213e; border-radius: 8px; overflow: hidden; }
        th, td { padding: 12px 15px; text-align: left; border-bottom: 1px solid #0f3460; }
        th { background: #0f3460; color: #10b981; }
        tr:hover { background: #0f3460; }
        .military { color: #f59e0b; }
    </style>
</head>
<body>
    <h1>SkySpy Live Dashboard</h1>
    <div class="stats">
        <div class="stat"><div class="stat-value">{{len .}}</div>Aircraft</div>
    </div>
    <table>
        <tr><th>Callsign</th><th>Type</th><th>Altitude</th><th>Speed</th><th>Distance</th></tr>
        {{range .}}
        <tr>
            <td>{{.Flight}}</td>
            <td>{{.Type}}</td>
            <td>{{.Alt}} ft</td>
            <td>{{.Speed}} kts</td>
            <td>{{printf "%.1f" .Distance}} nm</td>
        </tr>
        {{end}}
    </table>
</body>
</html>`

func main() {
    skyspyURL = os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    tmpl := template.Must(template.New("dashboard").Parse(dashboardHTML))

    // Start SSE listener in background
    go streamAircraft()

    // Cleanup stale aircraft every 30 seconds
    go func() {
        for range time.Tick(30 * time.Second) {
            mu.Lock()
            for hex, ac := range aircraft {
                if time.Since(ac.LastSeen) > 60*time.Second {
                    delete(aircraft, hex)
                }
            }
            mu.Unlock()
        }
    }()

    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        mu.RLock()
        defer mu.RUnlock()
        list := make([]*Aircraft, 0, len(aircraft))
        for _, v := range aircraft {
            list = append(list, v)
        }
        tmpl.Execute(w, list)
    })

    fmt.Println("Dashboard running at http://localhost:8080")
    http.ListenAndServe(":8080", nil)
}

func streamAircraft() {
    for {
        resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
        if err != nil {
            fmt.Printf("SSE connection failed: %v, retrying...\n", err)
            time.Sleep(5 * time.Second)
            continue
        }

        scanner := bufio.NewScanner(resp.Body)
        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data struct {
                    Aircraft []Aircraft `json:"aircraft"`
                }
                if err := json.Unmarshal([]byte(line[5:]), &data); err != nil {
                    continue
                }
                mu.Lock()
                for i := range data.Aircraft {
                    ac := &data.Aircraft[i]
                    ac.LastSeen = time.Now()
                    aircraft[ac.Hex] = ac
                }
                mu.Unlock()
            }
        }
        resp.Body.Close()
    }
}
```

```python Python
from flask import Flask, render_template_string, jsonify
import json
import os
import requests
import sseclient
import threading
from datetime import datetime, timedelta

app = Flask(__name__)
aircraft = {}
lock = threading.Lock()

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>SkySpy Dashboard</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; margin: 20px; background: #1a1a2e; color: #eee; }
        h1 { color: #10b981; }
        .stats { display: flex; gap: 20px; margin-bottom: 20px; }
        .stat { background: #16213e; padding: 15px 25px; border-radius: 8px; }
        .stat-value { font-size: 2em; font-weight: bold; color: #10b981; }
        table { width: 100%; border-collapse: collapse; background: #16213e; border-radius: 8px; overflow: hidden; }
        th, td { padding: 12px 15px; text-align: left; border-bottom: 1px solid #0f3460; }
        th { background: #0f3460; color: #10b981; }
        tr:hover { background: #0f3460; }
        .military { color: #f59e0b; }
        .refresh-notice { color: #666; font-size: 0.9em; }
    </style>
    <script>
        setTimeout(() => location.reload(), 5000);
    </script>
</head>
<body>
    <h1>SkySpy Live Dashboard</h1>
    <p class="refresh-notice">Auto-refreshes every 5 seconds</p>
    <div class="stats">
        <div class="stat"><div class="stat-value">{{ aircraft|length }}</div>Aircraft</div>
        <div class="stat"><div class="stat-value">{{ military_count }}</div>Military</div>
    </div>
    <table>
        <tr><th>Callsign</th><th>Type</th><th>Altitude</th><th>Speed</th><th>Distance</th><th>Military</th></tr>
        {% for a in aircraft.values() %}
        <tr class="{{ 'military' if a.get('military') else '' }}">
            <td>{{ a.get('flight', a.get('hex', 'Unknown')) }}</td>
            <td>{{ a.get('type', 'Unknown') }}</td>
            <td>{{ '{:,}'.format(a.get('alt', 0)) }} ft</td>
            <td>{{ a.get('gs', 0) }} kts</td>
            <td>{{ '{:.1f}'.format(a.get('distance', 0)) }} nm</td>
            <td>{{ '🎖️' if a.get('military') else '' }}</td>
        </tr>
        {% endfor %}
    </table>
</body>
</html>
"""

def stream_aircraft():
    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)
            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    with lock:
                        for a in data.get("aircraft", []):
                            a["_last_seen"] = datetime.now()
                            aircraft[a["hex"]] = a
        except Exception as e:
            print(f"SSE connection failed: {e}, reconnecting...")
            import time
            time.sleep(5)

def cleanup_stale():
    import time
    while True:
        time.sleep(30)
        cutoff = datetime.now() - timedelta(seconds=60)
        with lock:
            stale = [k for k, v in aircraft.items() if v.get("_last_seen", datetime.now()) < cutoff]
            for k in stale:
                del aircraft[k]

@app.route("/")
def index():
    with lock:
        military_count = sum(1 for a in aircraft.values() if a.get("military"))
        return render_template_string(TEMPLATE, aircraft=aircraft, military_count=military_count)

@app.route("/api/aircraft")
def api_aircraft():
    with lock:
        return jsonify(list(aircraft.values()))

if __name__ == "__main__":
    threading.Thread(target=stream_aircraft, daemon=True).start()
    threading.Thread(target=cleanup_stale, daemon=True).start()
    print("Dashboard running at http://localhost:8080")
    app.run(host="0.0.0.0", port=8080)
```

```javascript JavaScript
// src/App.tsx - Complete React Dashboard
import { useEffect, useState, useCallback } from 'react';

interface Aircraft {
  hex: string;
  flight?: string;
  type?: string;
  alt?: number;
  gs?: number;
  track?: number;
  lat?: number;
  lon?: number;
  distance?: number;
  military?: boolean;
}

function useSkySpySSE(url = 'http://localhost:5000/api/v1/map/sse') {
  const [aircraft, setAircraft] = useState<Map<string, Aircraft>>(new Map());
  const [connected, setConnected] = useState(false);
  const [lastUpdate, setLastUpdate] = useState<Date | null>(null);

  useEffect(() => {
    const es = new EventSource(url);

    es.onopen = () => setConnected(true);
    es.onerror = () => setConnected(false);

    es.addEventListener('aircraft_update', (e) => {
      const data = JSON.parse(e.data);
      setAircraft(prev => {
        const next = new Map(prev);
        data.aircraft.forEach((a: Aircraft) => {
          next.set(a.hex, { ...next.get(a.hex), ...a, _lastSeen: Date.now() });
        });
        return next;
      });
      setLastUpdate(new Date());
    });

    // Cleanup stale aircraft every 30 seconds
    const cleanup = setInterval(() => {
      setAircraft(prev => {
        const next = new Map(prev);
        const cutoff = Date.now() - 60000;
        for (const [hex, ac] of next) {
          if ((ac as any)._lastSeen < cutoff) {
            next.delete(hex);
          }
        }
        return next;
      });
    }, 30000);

    return () => {
      es.close();
      clearInterval(cleanup);
    };
  }, [url]);

  return {
    aircraft: Array.from(aircraft.values()),
    connected,
    count: aircraft.size,
    lastUpdate
  };
}

export default function App() {
  const skyspyUrl = import.meta.env.VITE_SKYSPY_URL || 'http://localhost:5000';
  const { aircraft, connected, count, lastUpdate } = useSkySpySSE(`${skyspyUrl}/api/v1/map/sse`);

  const militaryCount = aircraft.filter(a => a.military).length;

  return (
    <div style={{ fontFamily: 'system-ui', padding: 20, background: '#1a1a2e', minHeight: '100vh', color: '#eee' }}>
      <h1 style={{ color: '#10b981' }}>SkySpy Live Dashboard</h1>

      <div style={{ display: 'flex', gap: 20, marginBottom: 20 }}>
        <div style={{ background: '#16213e', padding: '15px 25px', borderRadius: 8 }}>
          <div style={{ fontSize: '2em', fontWeight: 'bold', color: '#10b981' }}>{count}</div>
          <div>Aircraft</div>
        </div>
        <div style={{ background: '#16213e', padding: '15px 25px', borderRadius: 8 }}>
          <div style={{ fontSize: '2em', fontWeight: 'bold', color: '#f59e0b' }}>{militaryCount}</div>
          <div>Military</div>
        </div>
        <div style={{ background: '#16213e', padding: '15px 25px', borderRadius: 8 }}>
          <div style={{ fontSize: '2em', fontWeight: 'bold', color: connected ? '#10b981' : '#ef4444' }}>
            {connected ? '●' : '○'}
          </div>
          <div>{connected ? 'Connected' : 'Disconnected'}</div>
        </div>
      </div>

      <table style={{ width: '100%', borderCollapse: 'collapse', background: '#16213e', borderRadius: 8 }}>
        <thead>
          <tr style={{ background: '#0f3460' }}>
            <th style={{ padding: '12px 15px', textAlign: 'left', color: '#10b981' }}>Callsign</th>
            <th style={{ padding: '12px 15px', textAlign: 'left', color: '#10b981' }}>Type</th>
            <th style={{ padding: '12px 15px', textAlign: 'left', color: '#10b981' }}>Altitude</th>
            <th style={{ padding: '12px 15px', textAlign: 'left', color: '#10b981' }}>Speed</th>
            <th style={{ padding: '12px 15px', textAlign: 'left', color: '#10b981' }}>Distance</th>
            <th style={{ padding: '12px 15px', textAlign: 'left', color: '#10b981' }}>Military</th>
          </tr>
        </thead>
        <tbody>
          {aircraft.map(a => (
            <tr key={a.hex} style={{ borderBottom: '1px solid #0f3460' }}>
              <td style={{ padding: '12px 15px', color: a.military ? '#f59e0b' : 'inherit' }}>
                {a.flight || a.hex}
              </td>
              <td style={{ padding: '12px 15px' }}>{a.type || 'Unknown'}</td>
              <td style={{ padding: '12px 15px' }}>{a.alt?.toLocaleString() || 0} ft</td>
              <td style={{ padding: '12px 15px' }}>{a.gs || 0} kts</td>
              <td style={{ padding: '12px 15px' }}>{a.distance?.toFixed(1) || 0} nm</td>
              <td style={{ padding: '12px 15px' }}>{a.military ? '🎖️' : ''}</td>
            </tr>
          ))}
        </tbody>
      </table>

      <p style={{ marginTop: 20, color: '#666', fontSize: '0.9em' }}>
        Last update: {lastUpdate?.toLocaleTimeString() || 'Never'}
      </p>
    </div>
  );
}
```

```json Response Example
{
  "aircraft": [
    {"hex": "A12345", "flight": "UAL123", "type": "B738", "alt": 35000, "gs": 450, "distance": 12.4, "military": false},
    {"hex": "AE1234", "flight": "RCH419", "type": "C17", "alt": 28000, "gs": 380, "distance": 8.2, "military": true}
  ],
  "count": 2,
  "connected": true
}
```

## Set Up Project

<!-- shell@1-4 -->
<!-- go@1-16 -->
<!-- python@1-12 -->
<!-- javascript@1-12 -->

Create a new project with your preferred framework. The React version uses Vite with TypeScript for a modern development experience.

## Connect to SSE Stream

<!-- go@97-120 -->
<!-- python@50-63 -->
<!-- javascript@13-47 -->

Establish a persistent connection to SkySpy's SSE endpoint. Aircraft data updates in real-time as new events arrive from the stream. The connection automatically reconnects if it drops.

## Render Dashboard UI

<!-- go@76-95 -->
<!-- python@72-77 -->
<!-- javascript@49-105 -->

Display the aircraft data in a styled table format. The dashboard shows key information like callsign, type, altitude, speed, and distance. Military aircraft are highlighted.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `VITE_SKYSPY_URL` | React env variable | `http://localhost:5000` |

### Customizing the Dashboard

Add additional columns by modifying the table:

```javascript
// Add more fields to display
<td>{a.lat?.toFixed(4)}, {a.lon?.toFixed(4)}</td>
<td>{a.squawk || '----'}</td>
<td>{a.track?.toFixed(0)}°</td>
```

### Filtering Aircraft

Show only specific aircraft types:

```javascript
const filtered = aircraft.filter(a =>
  a.military ||
  (a.alt && a.alt < 5000) ||
  (a.distance && a.distance < 10)
);
```

## Testing & Verification

1. **Start the dashboard** using the shell commands for your chosen language
2. **Open in browser** - Navigate to `http://localhost:8080`
3. **Verify live updates** - Aircraft should appear and update automatically
4. **Check connection status** - The React version shows a connection indicator

### Development Mode

For React development with hot reload:

```shell
npm run dev
```

The development server runs on `http://localhost:5173` with hot module replacement.

## Troubleshooting

### No Aircraft Appearing

**Problem:** Dashboard is blank with no aircraft
**Solution:**
- Verify SkySpy is running: `curl http://localhost:5000/api/v1/map/sse`
- Check browser console for CORS errors
- Ensure aircraft are in range of your receiver

### CORS Errors

**Problem:** Browser blocks SSE connection
**Solution:** Configure SkySpy to allow CORS from your dashboard origin, or run the dashboard from the same origin.

### Aircraft Disappearing Too Quickly

**Problem:** Aircraft vanish before they should
**Solution:** Increase the stale timeout in the cleanup function:

```python
cutoff = datetime.now() - timedelta(seconds=120)  # 2 minutes
```

### High Memory Usage

**Problem:** Memory grows over time
**Solution:** The cleanup function should remove stale aircraft. Verify it's running and the timeout is appropriate.

## Related Recipes

- [Leaflet.js Map](/docs/leaflet-map) - Add a map to your dashboard
- [Terminal Dashboard](/docs/terminal-dashboard) - CLI-based display
- [OBS Browser Source](/docs/obs-overlay) - Stream overlay
- [Grafana Dashboard](/docs/grafana-dashboard) - Advanced metrics
