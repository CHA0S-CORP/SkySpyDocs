---
title: Build a Live Dashboard
excerpt: Create a real-time aircraft monitoring dashboard with React and SSE.
hidden: true
recipe:
  color: '#10B981'
  icon: 🖥️
---
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
    "strings"
    "sync"
)

var (
    aircraft = make(map[string]interface{})
    mu       sync.RWMutex
    tmpl     = template.Must(template.New("dashboard").Parse(`
<!DOCTYPE html>
<html><head><title>SkySpy Dashboard</title></head>
<body>
<h1>Aircraft: {{len .}}</h1>
<table><tr><th>Callsign</th><th>Type</th><th>Altitude</th></tr>
{{range .}}<tr><td>{{.flight}}</td><td>{{.type}}</td><td>{{.alt}}</td></tr>{{end}}
</table>
<script>setTimeout(() => location.reload(), 5000)</script>
</body></html>`))
)

func main() {
    go func() {
        resp, _ := http.Get("http://localhost:5000/api/v1/map/sse")
        scanner := bufio.NewScanner(resp.Body)
        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data map[string]interface{}
                json.Unmarshal([]byte(line[5:]), &data)
                if ac, ok := data["aircraft"].([]interface{}); ok {
                    mu.Lock()
                    for _, a := range ac {
                        m := a.(map[string]interface{})
                        aircraft[fmt.Sprint(m["hex"])] = m
                    }
                    mu.Unlock()
                }
            }
        }
    }()

    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        mu.RLock()
        defer mu.RUnlock()
        list := make([]map[string]interface{}, 0, len(aircraft))
        for _, v := range aircraft {
            list = append(list, v.(map[string]interface{}))
        }
        tmpl.Execute(w, list)
    })
    http.ListenAndServe(":8080", nil)
}
```

```python Python
from flask import Flask, render_template_string
import json
import requests
import sseclient
import threading

app = Flask(__name__)
aircraft = {}

TEMPLATE = """
<!DOCTYPE html>
<html><head><title>SkySpy Dashboard</title>
<meta http-equiv="refresh" content="5"></head>
<body>
<h1>Aircraft: {{ aircraft|length }}</h1>
<table><tr><th>Callsign</th><th>Type</th><th>Altitude</th></tr>
{% for a in aircraft.values() %}
<tr><td>{{ a.flight }}</td><td>{{ a.type }}</td><td>{{ a.alt }}</td></tr>
{% endfor %}
</table>
</body></html>
"""

def stream_aircraft():
    response = requests.get("http://localhost:5000/api/v1/map/sse", stream=True)
    client = sseclient.SSEClient(response)
    for event in client.events():
        if event.event in ["aircraft_update", "aircraft_new"]:
            data = json.loads(event.data)
            for a in data.get("aircraft", []):
                aircraft[a["hex"]] = a

@app.route("/")
def index():
    return render_template_string(TEMPLATE, aircraft=aircraft)

if __name__ == "__main__":
    threading.Thread(target=stream_aircraft, daemon=True).start()
    app.run(port=8080)
```

```javascript JavaScript
// src/hooks/useSkySpySSE.ts
import { useEffect, useState } from 'react';

export function useSkySpySSE(url = 'http://localhost:5000/api/v1/map/sse') {
  const [aircraft, setAircraft] = useState(new Map());
  const [connected, setConnected] = useState(false);

  useEffect(() => {
    const es = new EventSource(url);
    es.onopen = () => setConnected(true);
    es.onerror = () => setConnected(false);

    es.addEventListener('aircraft_update', (e) => {
      const data = JSON.parse(e.data);
      setAircraft(prev => {
        const next = new Map(prev);
        data.aircraft.forEach(a => next.set(a.hex, { ...next.get(a.hex), ...a }));
        return next;
      });
    });

    return () => es.close();
  }, [url]);

  return { aircraft: Array.from(aircraft.values()), connected, count: aircraft.size };
}
```

```json Response Example
{"aircraft": [{"hex": "A12345", "flight": "UAL123", "type": "B738", "alt": 35000, "gs": 450}], "count": 1, "connected": true}
```

# Set Up Project

<!-- shell@1-3 -->
<!-- go@1-12 -->
<!-- python@1-8 -->
<!-- javascript@1-6 -->

Create a new project and set up the basic structure. The React version uses Vite with TypeScript, while Go and Python create simple HTTP servers.

# Connect to SSE Stream

<!-- shell@4 -->
<!-- go@29-43 -->
<!-- python@22-31 -->
<!-- javascript@8-21 -->

Establish a connection to SkySpy's SSE endpoint. Aircraft data updates in real-time as new events arrive from the stream.

# Render Dashboard UI

<!-- shell@4 -->
<!-- go@45-55 -->
<!-- python@33-35 -->
<!-- javascript@23-25 -->

Display the aircraft data in a table format. The dashboard auto-refreshes to show the latest aircraft positions and details.
