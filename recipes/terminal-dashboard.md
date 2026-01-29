---
title: Terminal Dashboard
description: Display live aircraft data in the terminal.
hidden: false
recipe:
  color: '#22C55E'
  icon: 🖥️
---
```shell Shell
watch -n 1 'curl -s http://localhost:5000/api/v1/aircraft | jq -r ".aircraft[] | [.flight, .type, .alt] | @tsv"'
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"os"
	"os/exec"
	"time"
)

func clearScreen() {
	cmd := exec.Command("clear")
	cmd.Stdout = os.Stdout
	cmd.Run()
}

func main() {
	for {
		clearScreen()
		fmt.Println("╔══════════════════════════════════════════════════╗")
		fmt.Println("║           SKYSPY TERMINAL DASHBOARD              ║")
		fmt.Println("╠══════════════════════════════════════════════════╣")
		fmt.Printf("║ %-10s %-8s %-10s %-8s %-8s ║\n", "FLIGHT", "TYPE", "ALT", "SPD", "HDG")
		fmt.Println("╠══════════════════════════════════════════════════╣")

		resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
		var data struct {
			Aircraft []struct {
				Flight string `json:"flight"`
				Type   string `json:"type"`
				Alt    int    `json:"alt"`
				Gs     int    `json:"gs"`
				Track  int    `json:"track"`
			} `json:"aircraft"`
		}
		json.NewDecoder(resp.Body).Decode(&data)
		resp.Body.Close()

		for _, ac := range data.Aircraft[:min(10, len(data.Aircraft))] {
			fmt.Printf("║ %-10s %-8s %6d ft %4d kts %4d° ║\n",
				ac.Flight, ac.Type, ac.Alt, ac.Gs, ac.Track)
		}
		fmt.Println("╚══════════════════════════════════════════════════╝")

		time.Sleep(time.Second)
	}
}
```

```python Python
import json
import os
import time
import requests

SKYSPY_URL = "http://localhost:5000"

def clear_screen():
    os.system('clear' if os.name == 'posix' else 'cls')

def display_dashboard():
    while True:
        clear_screen()
        print("╔══════════════════════════════════════════════════╗")
        print("║           SKYSPY TERMINAL DASHBOARD              ║")
        print("╠══════════════════════════════════════════════════╣")
        print(f"║ {'FLIGHT':<10} {'TYPE':<8} {'ALT':<10} {'SPD':<8} {'HDG':<8} ║")
        print("╠══════════════════════════════════════════════════╣")

        response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
        aircraft = response.json().get("aircraft", [])[:10]

        for ac in aircraft:
            flight = ac.get("flight", "")[:10]
            atype = ac.get("type", "")[:8]
            alt = ac.get("alt", 0)
            gs = ac.get("gs", 0)
            track = ac.get("track", 0)
            print(f"║ {flight:<10} {atype:<8} {alt:>6} ft {gs:>4} kts {track:>4}° ║")

        print("╚══════════════════════════════════════════════════╝")
        time.sleep(1)

display_dashboard()
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

function clearScreen() {
    process.stdout.write('\x1B[2J\x1B[0f');
}

async function displayDashboard() {
    while (true) {
        clearScreen();
        console.log('╔══════════════════════════════════════════════════╗');
        console.log('║           SKYSPY TERMINAL DASHBOARD              ║');
        console.log('╠══════════════════════════════════════════════════╣');
        console.log(`║ ${'FLIGHT'.padEnd(10)} ${'TYPE'.padEnd(8)} ${'ALT'.padEnd(10)} ${'SPD'.padEnd(8)} ${'HDG'.padEnd(8)} ║`);
        console.log('╠══════════════════════════════════════════════════╣');

        const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
        const data = await response.json();

        for (const ac of (data.aircraft || []).slice(0, 10)) {
            const flight = (ac.flight || '').padEnd(10).slice(0, 10);
            const type = (ac.type || '').padEnd(8).slice(0, 8);
            const alt = String(ac.alt || 0).padStart(6);
            const gs = String(ac.gs || 0).padStart(4);
            const track = String(ac.track || 0).padStart(4);
            console.log(`║ ${flight} ${type} ${alt} ft ${gs} kts ${track}° ║`);
        }
        console.log('╚══════════════════════════════════════════════════╝');

        await new Promise(r => setTimeout(r, 1000));
    }
}

displayDashboard();
```

```json Response Example
{"aircraft": [{"flight": "UAL123", "type": "B738", "alt": 35000, "gs": 450, "track": 270}]}
```

# Set Up Display Loop

<!-- shell@1 -->
<!-- go@12-16 -->
<!-- python@9-10 -->
<!-- javascript@3-5 -->

Clear the screen before each update to create a refreshing display effect.

# Draw Dashboard Frame

<!-- go@18-24 -->
<!-- python@12-17 -->
<!-- javascript@9-14 -->

Draw a bordered table with column headers for flight info. Unicode box-drawing characters create clean borders.

# Display Aircraft Data

<!-- go@26-40 -->
<!-- python@19-29 -->
<!-- javascript@16-27 -->

Fetch current aircraft and display the top 10 in formatted columns with altitude, speed, and heading.
