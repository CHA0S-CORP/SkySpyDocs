---
title: Altitude Filter
description: Filter aircraft by altitude ranges.
hidden: false
recipe:
  color: '#0EA5E9'
  icon: ⬆️
---
```shell Shell
curl -s "http://localhost:5000/api/v1/aircraft" | \
  jq '.aircraft[] | select(.alt >= 30000 and .alt <= 40000)'
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type AltitudeFilter struct {
	Min, Max int
	Name     string
}

var altitudeBands = []AltitudeFilter{
	{0, 5000, "Low altitude (surface traffic, helicopters)"},
	{5000, 18000, "Mid altitude (regional flights, GA)"},
	{18000, 30000, "High altitude (climbing/descending jets)"},
	{30000, 45000, "Cruise altitude (airliners)"},
}

func main() {
	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct {
		Aircraft []struct {
			Flight string `json:"flight"`
			Alt    int    `json:"alt"`
		} `json:"aircraft"`
	}
	json.NewDecoder(resp.Body).Decode(&data)
	resp.Body.Close()

	for _, band := range altitudeBands {
		count := 0
		for _, ac := range data.Aircraft {
			if ac.Alt >= band.Min && ac.Alt < band.Max {
				count++
			}
		}
		fmt.Printf("%s: %d aircraft\n", band.Name, count)
	}
}
```

```python Python
import requests

SKYSPY_URL = "http://localhost:5000"

ALTITUDE_BANDS = [
    (0, 5000, "Low altitude (surface traffic, helicopters)"),
    (5000, 18000, "Mid altitude (regional flights, GA)"),
    (18000, 30000, "High altitude (climbing/descending jets)"),
    (30000, 45000, "Cruise altitude (airliners)"),
]

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

for min_alt, max_alt, name in ALTITUDE_BANDS:
    count = sum(1 for ac in aircraft if min_alt <= ac.get("alt", 0) < max_alt)
    print(f"{name}: {count} aircraft")
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

const ALTITUDE_BANDS = [
    { min: 0, max: 5000, name: 'Low altitude (surface traffic, helicopters)' },
    { min: 5000, max: 18000, name: 'Mid altitude (regional flights, GA)' },
    { min: 18000, max: 30000, name: 'High altitude (climbing/descending jets)' },
    { min: 30000, max: 45000, name: 'Cruise altitude (airliners)' },
];

async function analyzeAltitudes() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    for (const band of ALTITUDE_BANDS) {
        const count = (data.aircraft || []).filter(
            ac => ac.alt >= band.min && ac.alt < band.max
        ).length;
        console.log(`${band.name}: ${count} aircraft`);
    }
}

analyzeAltitudes();
```

```json Response Example
{"aircraft": [{"flight": "UAL123", "alt": 35000}, {"flight": "N12345", "alt": 3500}]}
```

# Define Altitude Bands

<!-- go@10-17 -->
<!-- python@5-10 -->
<!-- javascript@3-8 -->

Create altitude ranges for different flight phases. Low altitude catches helicopters and departures, cruise altitude catches airliners.

# Fetch Aircraft Data

<!-- shell@1 -->
<!-- go@19-28 -->
<!-- python@12-13 -->
<!-- javascript@10-12 -->

Get all aircraft from the SkySpy API. Each aircraft includes current barometric altitude in feet.

# Count by Band

<!-- shell@2 -->
<!-- go@30-38 -->
<!-- python@15-17 -->
<!-- javascript@14-20 -->

Group aircraft by altitude band and count. Useful for understanding traffic patterns and airspace utilization.
