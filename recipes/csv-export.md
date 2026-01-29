---
title: CSV Export
description: Export aircraft data to CSV files.
hidden: false
recipe:
  color: '#16A34A'
  icon: 📄
---
```shell Shell
curl -s "http://localhost:5000/api/v1/aircraft" | \
  jq -r '.aircraft[] | [.hex, .flight, .type, .alt, .lat, .lon] | @csv' > aircraft.csv
```

```go Go
package main

import (
	"encoding/csv"
	"encoding/json"
	"net/http"
	"os"
	"strconv"
	"time"
)

func main() {
	filename := time.Now().Format("aircraft_2006-01-02_150405.csv")
	file, _ := os.Create(filename)
	defer file.Close()

	writer := csv.NewWriter(file)
	defer writer.Flush()

	writer.Write([]string{"hex", "flight", "type", "alt", "lat", "lon", "gs", "track"})

	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct {
		Aircraft []struct {
			Hex, Flight, Type string
			Alt, Gs, Track    int
			Lat, Lon          float64
		} `json:"aircraft"`
	}
	json.NewDecoder(resp.Body).Decode(&data)
	resp.Body.Close()

	for _, ac := range data.Aircraft {
		writer.Write([]string{
			ac.Hex, ac.Flight, ac.Type,
			strconv.Itoa(ac.Alt),
			strconv.FormatFloat(ac.Lat, 'f', 6, 64),
			strconv.FormatFloat(ac.Lon, 'f', 6, 64),
			strconv.Itoa(ac.Gs), strconv.Itoa(ac.Track),
		})
	}
}
```

```python Python
import csv
from datetime import datetime
import requests

SKYSPY_URL = "http://localhost:5000"

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

filename = datetime.now().strftime("aircraft_%Y-%m-%d_%H%M%S.csv")

with open(filename, "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["hex", "flight", "type", "alt", "lat", "lon", "gs", "track"])

    for ac in aircraft:
        writer.writerow([
            ac.get("hex"),
            ac.get("flight"),
            ac.get("type"),
            ac.get("alt"),
            ac.get("lat"),
            ac.get("lon"),
            ac.get("gs"),
            ac.get("track"),
        ])

print(f"Exported {len(aircraft)} aircraft to {filename}")
```

```javascript JavaScript
const fs = require('fs');

const SKYSPY_URL = 'http://localhost:5000';

async function exportCSV() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    const timestamp = new Date().toISOString().replace(/[:.]/g, '-').slice(0, 19);
    const filename = `aircraft_${timestamp}.csv`;

    const headers = ['hex', 'flight', 'type', 'alt', 'lat', 'lon', 'gs', 'track'];
    const rows = [headers.join(',')];

    for (const ac of data.aircraft || []) {
        rows.push([
            ac.hex, ac.flight, ac.type, ac.alt, ac.lat, ac.lon, ac.gs, ac.track
        ].join(','));
    }

    fs.writeFileSync(filename, rows.join('\n'));
    console.log(`Exported ${data.aircraft?.length || 0} aircraft to ${filename}`);
}

exportCSV();
```

```json Response Example
hex,flight,type,alt,lat,lon,gs,track
A12345,UAL123,B738,35000,34.0522,-118.2437,450,270
```

# Create Timestamped File

<!-- go@12-14 -->
<!-- python@11 -->
<!-- javascript@9-10 -->

Generate a unique filename with timestamp. This enables historical tracking with one file per export.

# Write CSV Header

<!-- shell@2 -->
<!-- go@18 -->
<!-- python@14 -->
<!-- javascript@12-13 -->

Define column headers for the CSV. Include hex, callsign, type, altitude, position, speed, and heading.

# Export Aircraft Data

<!-- go@30-37 -->
<!-- python@16-26 -->
<!-- javascript@15-21 -->

Write each aircraft as a row. Handle missing values gracefully to avoid errors.
