---
title: Speed Filter
description: Filter aircraft by ground speed.
hidden: false
recipe:
  color: '#F97316'
  icon: ⚡
---
```shell Shell
curl -s "http://localhost:5000/api/v1/aircraft" | \
  jq '.aircraft[] | select(.gs >= 500)'
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

func main() {
	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct {
		Aircraft []struct {
			Flight string `json:"flight"`
			Type   string `json:"type"`
			Gs     int    `json:"gs"`
			Alt    int    `json:"alt"`
		} `json:"aircraft"`
	}
	json.NewDecoder(resp.Body).Decode(&data)
	resp.Body.Close()

	fmt.Println("Fast movers (>500 kts):")
	for _, ac := range data.Aircraft {
		if ac.Gs >= 500 {
			fmt.Printf("  %s (%s) - %d kts at %d ft\n",
				ac.Flight, ac.Type, ac.Gs, ac.Alt)
		}
	}

	fmt.Println("\nSlow movers (<100 kts):")
	for _, ac := range data.Aircraft {
		if ac.Gs > 0 && ac.Gs < 100 {
			fmt.Printf("  %s (%s) - %d kts at %d ft\n",
				ac.Flight, ac.Type, ac.Gs, ac.Alt)
		}
	}
}
```

```python Python
import requests

SKYSPY_URL = "http://localhost:5000"

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

fast_movers = [ac for ac in aircraft if ac.get("gs", 0) >= 500]
slow_movers = [ac for ac in aircraft if 0 < ac.get("gs", 0) < 100]

print("Fast movers (>500 kts):")
for ac in fast_movers:
    print(f"  {ac.get('flight', ac['hex'])} ({ac.get('type', '?')}) - "
          f"{ac.get('gs', 0)} kts at {ac.get('alt', 0):,} ft")

print("\nSlow movers (<100 kts):")
for ac in slow_movers:
    print(f"  {ac.get('flight', ac['hex'])} ({ac.get('type', '?')}) - "
          f"{ac.get('gs', 0)} kts at {ac.get('alt', 0):,} ft")
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

async function filterBySpeed() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();
    const aircraft = data.aircraft || [];

    const fastMovers = aircraft.filter(ac => ac.gs >= 500);
    const slowMovers = aircraft.filter(ac => ac.gs > 0 && ac.gs < 100);

    console.log('Fast movers (>500 kts):');
    for (const ac of fastMovers) {
        console.log(`  ${ac.flight || ac.hex} (${ac.type || '?'}) - ` +
            `${ac.gs} kts at ${ac.alt?.toLocaleString() || 0} ft`);
    }

    console.log('\nSlow movers (<100 kts):');
    for (const ac of slowMovers) {
        console.log(`  ${ac.flight || ac.hex} (${ac.type || '?'}) - ` +
            `${ac.gs} kts at ${ac.alt?.toLocaleString() || 0} ft`);
    }
}

filterBySpeed();
```

```json Response Example
{"aircraft": [{"flight": "UAL123", "gs": 520, "alt": 38000, "type": "B789"}]}
```

# Define Speed Thresholds

<!-- go@10-11 -->
<!-- python@6 -->
<!-- javascript@4-5 -->

Fast movers (>500 kts) are typically jets at cruise. Slow movers (<100 kts) are helicopters, light aircraft, or landing traffic.

# Filter Fast Aircraft

<!-- shell@1-2 -->
<!-- go@21-26 -->
<!-- python@8-13 -->
<!-- javascript@8-14 -->

Find high-speed aircraft. Military jets and supersonic aircraft can exceed 600 kts ground speed.

# Filter Slow Aircraft

<!-- go@28-33 -->
<!-- python@9,15-18 -->
<!-- javascript@9,16-20 -->

Identify slow-moving traffic like helicopters, blimps, or aircraft on approach. Useful for spotting unusual activity.
