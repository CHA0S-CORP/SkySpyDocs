---
title: JSON Lines Logging
description: Append aircraft data to JSON Lines files for easy processing.
hidden: false
recipe:
  color: '#FCD34D'
  icon: 📋
---
```shell Shell
echo '{"hex":"A12345","flight":"RCH419","alt":35000,"ts":"2024-01-15T10:30:00Z"}' >> aircraft.jsonl
```

```go Go
package main

import (
	"encoding/json"
	"os"
	"time"
)

type LogEntry struct {
	Hex       string    `json:"hex"`
	Flight    string    `json:"flight,omitempty"`
	Type      string    `json:"type,omitempty"`
	Alt       int       `json:"alt,omitempty"`
	Lat       float64   `json:"lat,omitempty"`
	Lon       float64   `json:"lon,omitempty"`
	Timestamp time.Time `json:"ts"`
}

func main() {
	f, _ := os.OpenFile("aircraft.jsonl", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	defer f.Close()

	entry := LogEntry{
		Hex:       "A12345",
		Flight:    "RCH419",
		Type:      "C-17",
		Alt:       35000,
		Lat:       34.05,
		Lon:       -118.25,
		Timestamp: time.Now(),
	}

	json.NewEncoder(f).Encode(entry)
}
```

```python Python
import json
from datetime import datetime

def log_aircraft(filename, aircraft):
    entry = {
        "hex": aircraft["hex"],
        "flight": aircraft.get("flight"),
        "type": aircraft.get("type"),
        "alt": aircraft.get("alt"),
        "lat": aircraft.get("lat"),
        "lon": aircraft.get("lon"),
        "ts": datetime.utcnow().isoformat() + "Z"
    }
    with open(filename, "a") as f:
        f.write(json.dumps(entry) + "\n")

# Example usage
log_aircraft("aircraft.jsonl", {
    "hex": "A12345",
    "flight": "RCH419",
    "type": "C-17",
    "alt": 35000,
    "lat": 34.05,
    "lon": -118.25
})
```

```javascript JavaScript
const fs = require('fs');

function logAircraft(filename, aircraft) {
    const entry = {
        hex: aircraft.hex,
        flight: aircraft.flight,
        type: aircraft.type,
        alt: aircraft.alt,
        lat: aircraft.lat,
        lon: aircraft.lon,
        ts: new Date().toISOString(),
    };
    fs.appendFileSync(filename, JSON.stringify(entry) + '\n');
}

// Example usage
logAircraft('aircraft.jsonl', {
    hex: 'A12345',
    flight: 'RCH419',
    type: 'C-17',
    alt: 35000,
    lat: 34.05,
    lon: -118.25,
});
```

```json Response Example
{"hex":"A12345","flight":"RCH419","type":"C-17","alt":35000,"lat":34.05,"lon":-118.25,"ts":"2024-01-15T10:30:00Z"}
```

# Define Log Entry Structure

<!-- go@9-17 -->
<!-- python@5-13 -->
<!-- javascript@4-12 -->

Each line is a complete JSON object with aircraft data and timestamp. JSON Lines format allows easy streaming and processing.

# Append to File

<!-- shell@1 -->
<!-- go@19-32 -->
<!-- python@14-15 -->
<!-- javascript@13-14 -->

Open file in append mode and write one JSON object per line. Each line is independent and parseable.

# Process Log File

<!-- go@19 -->
<!-- python@17-25 -->
<!-- javascript@16-24 -->

Read with: `cat aircraft.jsonl | jq -c 'select(.type=="C-17")'` or stream-process line by line in any language.
