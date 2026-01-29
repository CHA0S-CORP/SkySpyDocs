---
title: Leaflet.js Map
description: Display aircraft on a lightweight web map.
hidden: false
recipe:
  color: '#199900'
  icon: 🗺️
---
```shell Shell
python -m http.server 8080
```

```go Go
package main

import (
	"encoding/json"
	"net/http"
)

func main() {
	http.HandleFunc("/aircraft", func(w http.ResponseWriter, r *http.Request) {
		resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
		var data interface{}
		json.NewDecoder(resp.Body).Decode(&data)
		resp.Body.Close()

		w.Header().Set("Content-Type", "application/json")
		w.Header().Set("Access-Control-Allow-Origin", "*")
		json.NewEncoder(w).Encode(data)
	})

	http.Handle("/", http.FileServer(http.Dir(".")))
	http.ListenAndServe(":8080", nil)
}
```

```python Python
from flask import Flask, jsonify
from flask_cors import CORS
import requests

app = Flask(__name__)
CORS(app)

@app.route('/aircraft')
def get_aircraft():
    response = requests.get("http://localhost:5000/api/v1/aircraft")
    return jsonify(response.json())

if __name__ == '__main__':
    app.run(port=8080)
```

```javascript JavaScript
// Save as map.html and serve with any web server
const html = `
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <style>#map { height: 100vh; width: 100%; }</style>
</head>
<body>
    <div id="map"></div>
    <script>
        const map = L.map('map').setView([34.05, -118.25], 10);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

        const markers = {};
        async function update() {
            const resp = await fetch('http://localhost:5000/api/v1/aircraft');
            const data = await resp.json();
            for (const ac of data.aircraft || []) {
                if (!markers[ac.hex]) {
                    markers[ac.hex] = L.marker([ac.lat, ac.lon]).addTo(map);
                }
                markers[ac.hex].setLatLng([ac.lat, ac.lon])
                    .bindPopup(ac.flight + '<br>' + ac.alt + ' ft');
            }
        }
        setInterval(update, 2000);
        update();
    </script>
</body>
</html>
`;
require('fs').writeFileSync('map.html', html);
```

```json Response Example
{"aircraft": [{"hex": "A12345", "lat": 34.05, "lon": -118.25, "flight": "UAL123"}]}
```

# Set Up Map Container

<!-- javascript@4-10 -->
<!-- go@8-9 -->
<!-- python@5-6 -->

Include Leaflet CSS and JS. Create a full-screen map div. Initialize centered on your area.

# Create Aircraft Markers

<!-- javascript@14-22 -->
<!-- go@10-17 -->
<!-- python@8-11 -->

Fetch aircraft positions and create markers. Store markers by hex code for efficient updates.

# Update Positions

<!-- shell@1 -->
<!-- javascript@23-26 -->
<!-- go@19-20 -->
<!-- python@13-14 -->

Refresh markers every 2 seconds. Update existing marker positions instead of recreating them.
