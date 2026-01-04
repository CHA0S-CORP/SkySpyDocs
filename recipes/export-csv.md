---
title: Export to CSV
description: Log aircraft sightings to CSV files for analysis and record-keeping.
hidden: true
recipe:
  color: '#10B981'
  icon: 📊
---
```shell Shell
pip install sseclient-py requests
python csv_logger.py
```

```go Go
package main

import (
    "bufio"
    "encoding/csv"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "strings"
    "time"
)

func main() {
    file, _ := os.Create("aircraft.csv")
    writer := csv.NewWriter(file)
    writer.Write([]string{"timestamp", "hex", "flight", "type", "alt", "gs", "distance"})

    resp, _ := http.Get("http://localhost:5000/api/v1/map/sse")
    defer resp.Body.Close()

    scanner := bufio.NewScanner(resp.Body)
    for scanner.Scan() {
        line := scanner.Text()
        if strings.HasPrefix(line, "data:") {
            var data map[string]interface{}
            json.Unmarshal([]byte(line[5:]), &data)
            if aircraft, ok := data["aircraft"].([]interface{}); ok {
                for _, a := range aircraft {
                    ac := a.(map[string]interface{})
                    writer.Write([]string{
                        time.Now().Format(time.RFC3339),
                        fmt.Sprint(ac["hex"]),
                        fmt.Sprint(ac["flight"]),
                        fmt.Sprint(ac["type"]),
                        fmt.Sprint(ac["alt"]),
                        fmt.Sprint(ac["gs"]),
                        fmt.Sprint(ac["distance"]),
                    })
                }
            }
        }
        writer.Flush()
    }
}
```

```python Python
import csv
import json
import requests
import sseclient
from datetime import datetime

SKYSPY_URL = "http://localhost:5000"
FIELDS = ["timestamp", "hex", "flight", "type", "alt", "gs", "distance", "military"]

with open("aircraft.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=FIELDS)
    writer.writeheader()

    response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True)
    client = sseclient.SSEClient(response)

    for event in client.events():
        if event.event in ["aircraft_update", "aircraft_new"]:
            data = json.loads(event.data)
            for aircraft in data.get("aircraft", []):
                writer.writerow({
                    "timestamp": datetime.now().isoformat(),
                    "hex": aircraft.get("hex", ""),
                    "flight": aircraft.get("flight", ""),
                    "type": aircraft.get("type", ""),
                    "alt": aircraft.get("alt", ""),
                    "gs": aircraft.get("gs", ""),
                    "distance": aircraft.get("distance", ""),
                    "military": aircraft.get("military", False),
                })
                f.flush()
```

```javascript JavaScript
const EventSource = require('eventsource');
const fs = require('fs');

const fields = ['timestamp', 'hex', 'flight', 'type', 'alt', 'gs', 'distance'];
const stream = fs.createWriteStream('aircraft.csv');
stream.write(fields.join(',') + '\n');

const es = new EventSource('http://localhost:5000/api/v1/map/sse');

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);
  data.aircraft.forEach(a => {
    const row = [
      new Date().toISOString(),
      a.hex || '',
      a.flight || '',
      a.type || '',
      a.alt || '',
      a.gs || '',
      a.distance || ''
    ];
    stream.write(row.join(',') + '\n');
  });
});
```

```json Response Example
{"timestamp": "2024-01-15T14:32:18", "hex": "A12345", "flight": "UAL123", "type": "B738", "alt": 35000, "gs": 450, "distance": 12.4, "military": false}
```

# Create CSV File & Headers

<!-- shell@1-2 -->
<!-- go@14-17 -->
<!-- python@9-12 -->
<!-- javascript@4-6 -->

Create the CSV file and write the header row with column names for timestamp, aircraft identifier, flight info, and position data.

# Connect to SSE Stream

<!-- shell@2 -->
<!-- go@19-21 -->
<!-- python@14-16 -->
<!-- javascript@8-9 -->

Connect to SkySpy's Server-Sent Events endpoint to receive real-time aircraft updates. The stream will push new data as aircraft are detected.

# Write Aircraft Data

<!-- shell@2 -->
<!-- go@28-38 -->
<!-- python@20-31 -->
<!-- javascript@11-22 -->

For each aircraft update, extract the relevant fields and write a new row to the CSV file. Flush after each write to ensure data is saved immediately.
