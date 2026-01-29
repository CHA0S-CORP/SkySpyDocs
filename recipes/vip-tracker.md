---
title: VIP Aircraft Tracker
description: Track known VIP and government aircraft by tail number.
hidden: false
recipe:
  color: '#7C3AED'
  icon: ✈️
---
```shell Shell
curl -s "http://localhost:5000/api/v1/aircraft" | \
  jq '.aircraft[] | select(.flight | test("^(SAM|AF1|AF2|EXEC|VENUS)"))'
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"strings"
)

var vipCallsigns = map[string]string{
	"AF1":    "Air Force One",
	"AF2":    "Air Force Two",
	"SAM":    "Special Air Mission",
	"EXEC":   "Executive Flight",
	"VENUS":  "USAF VIP",
	"REACH":  "Air Mobility Command",
}

var vipHexCodes = map[string]string{
	"AE0001": "POTUS Aircraft",
	"AE0002": "VP Aircraft",
}

func main() {
	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct {
		Aircraft []struct {
			Hex    string `json:"hex"`
			Flight string `json:"flight"`
			Type   string `json:"type"`
			Alt    int    `json:"alt"`
		} `json:"aircraft"`
	}
	json.NewDecoder(resp.Body).Decode(&data)
	resp.Body.Close()

	for _, ac := range data.Aircraft {
		if desc, ok := vipHexCodes[ac.Hex]; ok {
			fmt.Printf("VIP: %s - %s (%s) at %d ft\n", desc, ac.Flight, ac.Type, ac.Alt)
			continue
		}
		for prefix, desc := range vipCallsigns {
			if strings.HasPrefix(ac.Flight, prefix) {
				fmt.Printf("VIP: %s - %s (%s) at %d ft\n", desc, ac.Flight, ac.Type, ac.Alt)
				break
			}
		}
	}
}
```

```python Python
import requests

SKYSPY_URL = "http://localhost:5000"

VIP_CALLSIGNS = {
    "AF1": "Air Force One",
    "AF2": "Air Force Two",
    "SAM": "Special Air Mission",
    "EXEC": "Executive Flight",
    "VENUS": "USAF VIP",
    "REACH": "Air Mobility Command",
}

VIP_HEX_CODES = {
    "AE0001": "POTUS Aircraft",
    "AE0002": "VP Aircraft",
}

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

for ac in aircraft:
    hex_code = ac["hex"]
    flight = ac.get("flight", "")

    if hex_code in VIP_HEX_CODES:
        print(f"VIP: {VIP_HEX_CODES[hex_code]} - {flight} ({ac.get('type')}) at {ac.get('alt'):,} ft")
        continue

    for prefix, desc in VIP_CALLSIGNS.items():
        if flight.startswith(prefix):
            print(f"VIP: {desc} - {flight} ({ac.get('type')}) at {ac.get('alt'):,} ft")
            break
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

const VIP_CALLSIGNS = {
    'AF1': 'Air Force One',
    'AF2': 'Air Force Two',
    'SAM': 'Special Air Mission',
    'EXEC': 'Executive Flight',
    'VENUS': 'USAF VIP',
    'REACH': 'Air Mobility Command',
};

const VIP_HEX_CODES = {
    'AE0001': 'POTUS Aircraft',
    'AE0002': 'VP Aircraft',
};

async function findVIPAircraft() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    for (const ac of data.aircraft || []) {
        if (VIP_HEX_CODES[ac.hex]) {
            console.log(`VIP: ${VIP_HEX_CODES[ac.hex]} - ${ac.flight} (${ac.type}) at ${ac.alt?.toLocaleString()} ft`);
            continue;
        }

        for (const [prefix, desc] of Object.entries(VIP_CALLSIGNS)) {
            if (ac.flight?.startsWith(prefix)) {
                console.log(`VIP: ${desc} - ${ac.flight} (${ac.type}) at ${ac.alt?.toLocaleString()} ft`);
                break;
            }
        }
    }
}

findVIPAircraft();
```

```json Response Example
{"hex": "AE0001", "flight": "SAM38", "type": "VC25A", "alt": 30000, "vip": true}
```

# Define VIP Lists

<!-- go@10-20 -->
<!-- python@5-17 -->
<!-- javascript@3-15 -->

Create lookup tables for known VIP callsign prefixes and specific hex codes. Common prefixes include SAM, AF1, and EXEC.

# Check Hex Codes First

<!-- shell@1-2 -->
<!-- go@34-37 -->
<!-- python@25-28 -->
<!-- javascript@21-24 -->

Match by hex code for known specific aircraft. This catches VIP aircraft even with unusual callsigns.

# Match Callsign Prefixes

<!-- go@38-43 -->
<!-- python@30-33 -->
<!-- javascript@26-32 -->

Check callsign prefixes for VIP patterns. Many government flights use predictable callsign formats.
