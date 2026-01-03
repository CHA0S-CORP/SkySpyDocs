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

# step1

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->

Create a logging system that records all aircraft sightings to CSV files. This is useful for analysis, record-keeping, and building historical datasets.

# step2

Example CSV output:

```csv
timestamp,hex,flight,type,lat,lon,alt,gs,distance,military,emergency
2024-01-15T14:32:18,A12345,UAL123,B738,47.6062,-122.3321,35000,450,12.4,False,False
2024-01-15T14:32:18,AE1234,RCH419,C17,47.5500,-122.4000,28000,420,18.2,True,False
```

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->

# step3

For daily log rotation, create a new file each day:

```python
from datetime import date
filename = f"aircraft_{date.today().isoformat()}.csv"
```

<!-- shell@ -->
<!-- go@ -->
<!-- python@ -->
<!-- javascript@ -->
