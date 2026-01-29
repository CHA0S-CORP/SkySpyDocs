---
title: Helicopter Watch
description: Monitor helicopter activity in your area.
hidden: false
recipe:
  color: '#DC2626'
  icon: 🚁
---
```shell Shell
curl -s "http://localhost:5000/api/v1/aircraft" | \
  jq '.aircraft[] | select(.category == "A7")'
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"strings"
)

var heliTypes = []string{"R22", "R44", "B206", "B407", "EC35", "EC45", "H125", "H145", "S76", "AW139"}

func isHelicopter(category, acType string) bool {
	if category == "A7" {
		return true
	}
	for _, t := range heliTypes {
		if strings.Contains(acType, t) {
			return true
		}
	}
	return false
}

func main() {
	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct {
		Aircraft []struct {
			Hex      string `json:"hex"`
			Flight   string `json:"flight"`
			Type     string `json:"type"`
			Category string `json:"category"`
			Alt      int    `json:"alt"`
		} `json:"aircraft"`
	}
	json.NewDecoder(resp.Body).Decode(&data)
	resp.Body.Close()

	fmt.Println("Helicopters overhead:")
	for _, ac := range data.Aircraft {
		if isHelicopter(ac.Category, ac.Type) {
			fmt.Printf("  %s (%s) - %d ft\n", ac.Flight, ac.Type, ac.Alt)
		}
	}
}
```

```python Python
import requests

SKYSPY_URL = "http://localhost:5000"
HELI_TYPES = {"R22", "R44", "B206", "B407", "EC35", "EC45", "H125", "H145", "S76", "AW139"}

def is_helicopter(aircraft):
    if aircraft.get("category") == "A7":
        return True
    ac_type = aircraft.get("type", "")
    return any(t in ac_type for t in HELI_TYPES)

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

print("Helicopters overhead:")
for ac in aircraft:
    if is_helicopter(ac):
        print(f"  {ac.get('flight', ac['hex'])} ({ac.get('type', 'Unknown')}) - {ac.get('alt', 0):,} ft")
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';
const HELI_TYPES = ['R22', 'R44', 'B206', 'B407', 'EC35', 'EC45', 'H125', 'H145', 'S76', 'AW139'];

function isHelicopter(aircraft) {
    if (aircraft.category === 'A7') return true;
    const type = aircraft.type || '';
    return HELI_TYPES.some(t => type.includes(t));
}

async function findHelicopters() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    console.log('Helicopters overhead:');
    for (const ac of data.aircraft || []) {
        if (isHelicopter(ac)) {
            console.log(`  ${ac.flight || ac.hex} (${ac.type || 'Unknown'}) - ${ac.alt?.toLocaleString() || 0} ft`);
        }
    }
}

findHelicopters();
```

```json Response Example
{"hex": "A12345", "flight": "N123TV", "type": "EC35", "category": "A7", "alt": 1500}
```

# Define Helicopter Detection

<!-- go@11-21 -->
<!-- python@4-10 -->
<!-- javascript@2-8 -->

Identify helicopters by ADS-B category A7 (rotorcraft) or by matching common helicopter type codes.

# Fetch Aircraft Data

<!-- shell@1-2 -->
<!-- go@23-34 -->
<!-- python@12-13 -->
<!-- javascript@10-13 -->

Query the SkySpy API for all aircraft. Each aircraft includes type and category information.

# Filter and Display

<!-- go@36-40 -->
<!-- python@15-17 -->
<!-- javascript@15-19 -->

List all helicopters with their callsign, type, and current altitude. Helicopters typically fly at lower altitudes.
