---
title: Rare Aircraft Spotter
description: Alert on unusual aircraft types and registrations.
hidden: false
recipe:
  color: '#EC4899'
  icon: 💎
---
```shell Shell
curl -s "http://localhost:5000/api/v1/aircraft" | \
  jq '.aircraft[] | select(.type | test("^(A380|B748|AN124|C5M|AN225)"))'
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"strings"
)

var rareTypes = map[string]string{
	"A380": "Airbus A380 - Largest passenger aircraft",
	"B748": "Boeing 747-8 - Latest jumbo variant",
	"AN124": "Antonov An-124 - Giant cargo aircraft",
	"C5M":  "Lockheed C-5M Super Galaxy",
	"A225": "Antonov An-225 Mriya (if flying!)",
	"B752": "Boeing 757-200 - Increasingly rare",
	"CONC": "Concorde (historical)",
}

func main() {
	seen := make(map[string]bool)

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
		for prefix, desc := range rareTypes {
			if strings.HasPrefix(ac.Type, prefix) && !seen[ac.Hex] {
				fmt.Printf("RARE: %s\n  %s (%s) at %d ft\n", desc, ac.Flight, ac.Hex, ac.Alt)
				seen[ac.Hex] = true
			}
		}
	}
}
```

```python Python
import requests

SKYSPY_URL = "http://localhost:5000"

RARE_TYPES = {
    "A380": "Airbus A380 - Largest passenger aircraft",
    "B748": "Boeing 747-8 - Latest jumbo variant",
    "AN124": "Antonov An-124 - Giant cargo aircraft",
    "C5M": "Lockheed C-5M Super Galaxy",
    "A225": "Antonov An-225 Mriya",
    "B752": "Boeing 757-200 - Increasingly rare",
}

seen = set()

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
for ac in response.json().get("aircraft", []):
    ac_type = ac.get("type", "")
    hex_code = ac["hex"]

    for prefix, desc in RARE_TYPES.items():
        if ac_type.startswith(prefix) and hex_code not in seen:
            print(f"RARE: {desc}")
            print(f"  {ac.get('flight', hex_code)} ({hex_code}) at {ac.get('alt', 0):,} ft")
            seen.add(hex_code)
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

const RARE_TYPES = {
    'A380': 'Airbus A380 - Largest passenger aircraft',
    'B748': 'Boeing 747-8 - Latest jumbo variant',
    'AN124': 'Antonov An-124 - Giant cargo aircraft',
    'C5M': 'Lockheed C-5M Super Galaxy',
    'A225': 'Antonov An-225 Mriya',
    'B752': 'Boeing 757-200 - Increasingly rare',
};

const seen = new Set();

async function findRareAircraft() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();

    for (const ac of data.aircraft || []) {
        const type = ac.type || '';

        for (const [prefix, desc] of Object.entries(RARE_TYPES)) {
            if (type.startsWith(prefix) && !seen.has(ac.hex)) {
                console.log(`RARE: ${desc}`);
                console.log(`  ${ac.flight || ac.hex} (${ac.hex}) at ${ac.alt?.toLocaleString() || 0} ft`);
                seen.add(ac.hex);
            }
        }
    }
}

findRareAircraft();
```

```json Response Example
{"hex": "4CA87D", "flight": "UAE2", "type": "A388", "alt": 38000, "rare": true}
```

# Define Rare Types

<!-- go@10-18 -->
<!-- python@5-12 -->
<!-- javascript@3-10 -->

Create a lookup of rare aircraft type codes with descriptions. Include widebodies, military transports, and retiring types.

# Track Seen Aircraft

<!-- go@20 -->
<!-- python@14 -->
<!-- javascript@12 -->

Prevent duplicate alerts by tracking which aircraft have already been reported.

# Match and Alert

<!-- shell@1-2 -->
<!-- go@34-39 -->
<!-- python@19-25 -->
<!-- javascript@20-27 -->

Check each aircraft against the rare types list. Alert with detailed information when a match is found.