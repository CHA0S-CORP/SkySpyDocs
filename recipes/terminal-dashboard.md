---
title: Terminal Dashboard
excerpt: Display aircraft in a CLI terminal interface.
hidden: false
recipe:
  color: '#22C55E'
  icon: 💻
difficulty: beginner
tags: [visualization, terminal, cli, display]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Terminal with ANSI color support
- Python: `pip install sseclient-py requests rich`

## What You'll Build

A real-time terminal dashboard showing aircraft in a table format with colors and live updates. Perfect for headless servers or terminal enthusiasts.

```shell Shell
pip install sseclient-py requests rich
python terminal_dashboard.py
```

```go Go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "os/exec"
    "sort"
    "strings"
    "time"
)

type Aircraft struct {
    Hex      string  `json:"hex"`
    Flight   string  `json:"flight"`
    Type     string  `json:"type"`
    Alt      int     `json:"alt"`
    Speed    int     `json:"gs"`
    Distance float64 `json:"distance"`
    Military bool    `json:"military"`
}

var aircraft = make(map[string]*Aircraft)

func main() {
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    go streamAircraft(skyspyURL)

    // Refresh display every second
    for {
        clearScreen()
        printDashboard()
        time.Sleep(time.Second)
    }
}

func streamAircraft(skyspyURL string) {
    for {
        resp, _ := http.Get(skyspyURL + "/api/v1/map/sse")
        scanner := bufio.NewScanner(resp.Body)

        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data struct {
                    Aircraft []Aircraft `json:"aircraft"`
                }
                json.Unmarshal([]byte(line[5:]), &data)
                for i := range data.Aircraft {
                    ac := &data.Aircraft[i]
                    aircraft[ac.Hex] = ac
                }
            }
        }
        resp.Body.Close()
        time.Sleep(5 * time.Second)
    }
}

func clearScreen() {
    cmd := exec.Command("clear")
    cmd.Stdout = os.Stdout
    cmd.Run()
}

func printDashboard() {
    fmt.Println("\033[1;36m╔════════════════════════════════════════════════════════════════╗\033[0m")
    fmt.Println("\033[1;36m║\033[0m              \033[1;37mSkySpy Terminal Dashboard\033[0m                        \033[1;36m║\033[0m")
    fmt.Println("\033[1;36m╠════════════════════════════════════════════════════════════════╣\033[0m")
    fmt.Printf("\033[1;36m║\033[0m  Aircraft: \033[1;32m%-5d\033[0m   Military: \033[1;33m%-5d\033[0m                            \033[1;36m║\033[0m\n",
        len(aircraft), countMilitary())
    fmt.Println("\033[1;36m╠════════════════════════════════════════════════════════════════╣\033[0m")
    fmt.Println("\033[1;36m║\033[0m  Callsign    Type    Alt      Speed   Distance  Military   \033[1;36m║\033[0m")
    fmt.Println("\033[1;36m╠════════════════════════════════════════════════════════════════╣\033[0m")

    sorted := sortByDistance()
    for i, ac := range sorted {
        if i >= 15 {
            break
        }
        mil := ""
        color := "\033[0m"
        if ac.Military {
            mil = "🎖️"
            color = "\033[1;33m"
        }
        fmt.Printf("\033[1;36m║\033[0m  %s%-10s %-7s %-8d %-7d %-9.1f %-6s\033[0m   \033[1;36m║\033[0m\n",
            color, truncate(ac.Flight, 10), truncate(ac.Type, 7), ac.Alt, ac.Speed, ac.Distance, mil)
    }
    fmt.Println("\033[1;36m╚════════════════════════════════════════════════════════════════╝\033[0m")
}

func countMilitary() int {
    count := 0
    for _, ac := range aircraft {
        if ac.Military {
            count++
        }
    }
    return count
}

func sortByDistance() []*Aircraft {
    list := make([]*Aircraft, 0, len(aircraft))
    for _, ac := range aircraft {
        list = append(list, ac)
    }
    sort.Slice(list, func(i, j int) bool {
        return list[i].Distance < list[j].Distance
    })
    return list
}

func truncate(s string, n int) string {
    if len(s) > n {
        return s[:n]
    }
    return s
}
```

```python Python
import json
import os
import sys
import time
from datetime import datetime
import requests
import sseclient
from rich.console import Console
from rich.table import Table
from rich.live import Live
from rich.panel import Panel
from rich.layout import Layout

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

console = Console()
aircraft = {}


def create_dashboard():
    """Create the dashboard layout."""
    layout = Layout()

    # Stats
    total = len(aircraft)
    military = sum(1 for a in aircraft.values() if a.get("military"))
    closest = min((a.get("distance", 999) for a in aircraft.values()), default=0)

    stats = f"[bold green]Aircraft:[/] {total}  [bold yellow]Military:[/] {military}  [bold blue]Closest:[/] {closest:.1f}nm"

    # Aircraft table
    table = Table(show_header=True, header_style="bold cyan", box=None)
    table.add_column("Callsign", width=12)
    table.add_column("Type", width=8)
    table.add_column("Altitude", width=10, justify="right")
    table.add_column("Speed", width=8, justify="right")
    table.add_column("Distance", width=10, justify="right")
    table.add_column("Track", width=6, justify="right")
    table.add_column("Mil", width=4)

    # Sort by distance
    sorted_aircraft = sorted(
        aircraft.values(),
        key=lambda a: a.get("distance", 999)
    )[:20]

    for ac in sorted_aircraft:
        flight = ac.get("flight", ac.get("hex", "???"))
        military = "🎖️" if ac.get("military") else ""
        style = "yellow" if ac.get("military") else ""

        table.add_row(
            f"[{style}]{flight}[/]",
            ac.get("type", "???"),
            f"{ac.get('alt', 0):,}",
            f"{ac.get('gs', 0)}",
            f"{ac.get('distance', 0):.1f}nm",
            f"{ac.get('track', 0):.0f}°",
            military
        )

    panel = Panel(
        table,
        title=f"[bold]SkySpy Terminal Dashboard[/] - {datetime.now().strftime('%H:%M:%S')}",
        subtitle=stats,
        border_style="cyan"
    )

    return panel


def stream_aircraft():
    """Stream aircraft updates in background."""
    global aircraft

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    for ac in data.get("aircraft", []):
                        ac["_last_seen"] = time.time()
                        aircraft[ac["hex"]] = ac

                    # Remove stale aircraft
                    now = time.time()
                    aircraft = {k: v for k, v in aircraft.items()
                               if now - v.get("_last_seen", 0) < 60}
        except Exception as e:
            console.print(f"[red]Error: {e}[/]")
            time.sleep(5)


def main():
    import threading
    thread = threading.Thread(target=stream_aircraft, daemon=True)
    thread.start()

    with Live(create_dashboard(), refresh_per_second=1, console=console) as live:
        while True:
            live.update(create_dashboard())
            time.sleep(1)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const aircraft = new Map();

function clearScreen() {
  process.stdout.write('\x1B[2J\x1B[0f');
}

function printDashboard() {
  clearScreen();

  const total = aircraft.size;
  const military = [...aircraft.values()].filter(a => a.military).length;

  console.log('\x1b[36m╔════════════════════════════════════════════════════════════════╗\x1b[0m');
  console.log('\x1b[36m║\x1b[0m              \x1b[1;37mSkySpy Terminal Dashboard\x1b[0m                        \x1b[36m║\x1b[0m');
  console.log('\x1b[36m╠════════════════════════════════════════════════════════════════╣\x1b[0m');
  console.log(`\x1b[36m║\x1b[0m  Aircraft: \x1b[32m${total.toString().padEnd(5)}\x1b[0m   Military: \x1b[33m${military.toString().padEnd(5)}\x1b[0m                            \x1b[36m║\x1b[0m`);
  console.log('\x1b[36m╠════════════════════════════════════════════════════════════════╣\x1b[0m');
  console.log('\x1b[36m║\x1b[0m  Callsign    Type    Alt      Speed   Distance  Military   \x1b[36m║\x1b[0m');
  console.log('\x1b[36m╠════════════════════════════════════════════════════════════════╣\x1b[0m');

  const sorted = [...aircraft.values()]
    .sort((a, b) => (a.distance || 999) - (b.distance || 999))
    .slice(0, 15);

  for (const ac of sorted) {
    const flight = (ac.flight || ac.hex || '???').slice(0, 10).padEnd(10);
    const type = (ac.type || '???').slice(0, 7).padEnd(7);
    const alt = (ac.alt || 0).toString().padEnd(8);
    const speed = (ac.gs || 0).toString().padEnd(7);
    const dist = (ac.distance || 0).toFixed(1).padEnd(9);
    const mil = ac.military ? '🎖️' : '  ';
    const color = ac.military ? '\x1b[33m' : '\x1b[0m';

    console.log(`\x1b[36m║\x1b[0m  ${color}${flight} ${type} ${alt} ${speed} ${dist} ${mil}\x1b[0m   \x1b[36m║\x1b[0m`);
  }

  console.log('\x1b[36m╚════════════════════════════════════════════════════════════════╝\x1b[0m');
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);
  for (const ac of data.aircraft || []) {
    ac._lastSeen = Date.now();
    aircraft.set(ac.hex, ac);
  }
});

// Cleanup stale
setInterval(() => {
  const cutoff = Date.now() - 60000;
  for (const [hex, ac] of aircraft) {
    if (ac._lastSeen < cutoff) aircraft.delete(hex);
  }
}, 10000);

// Update display
setInterval(printDashboard, 1000);
```

```text Output Example
╔════════════════════════════════════════════════════════════════╗
║              SkySpy Terminal Dashboard                        ║
╠════════════════════════════════════════════════════════════════╣
║  Aircraft: 23      Military: 2                                 ║
╠════════════════════════════════════════════════════════════════╣
║  Callsign    Type    Alt      Speed   Distance  Military       ║
╠════════════════════════════════════════════════════════════════╣
║  UAL123      B738    35000    450     12.4nm               ║
║  RCH419      C17     28000    380     15.2nm    🎖️           ║
║  DAL456      A321    38000    485     18.7nm               ║
╚════════════════════════════════════════════════════════════════╝
```

## Rich Library Dashboard

<!-- python@18-70 -->

The Python version uses the `rich` library for beautiful terminal output with colors, tables, and live updates.

### Install Rich

```shell
pip install rich
```

## ANSI Color Codes

For languages without rich library support, use ANSI escape codes:

| Code | Color |
|------|-------|
| `\033[0m` | Reset |
| `\033[1;31m` | Red |
| `\033[1;32m` | Green |
| `\033[1;33m` | Yellow |
| `\033[1;36m` | Cyan |

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Customizing Display

Sort by different fields:

```python
# Sort by altitude (highest first)
sorted_aircraft = sorted(aircraft.values(), key=lambda a: -a.get("alt", 0))

# Sort by callsign
sorted_aircraft = sorted(aircraft.values(), key=lambda a: a.get("flight", ""))

# Military first
sorted_aircraft = sorted(aircraft.values(), key=lambda a: (not a.get("military"), a.get("distance", 999)))
```

### Compact Mode

Show more aircraft with less detail:

```python
table.add_column("Call", width=8)
table.add_column("Type", width=4)
table.add_column("Alt", width=6)
table.add_column("Dist", width=5)
```

### Add Sparklines

Show altitude history:

```python
from rich.sparkline import Sparkline

# Store altitude history per aircraft
history = {}

def update_history(hex_code, alt):
    if hex_code not in history:
        history[hex_code] = []
    history[hex_code].append(alt)
    history[hex_code] = history[hex_code][-20:]  # Keep last 20

# In table
table.add_column("Trend")
table.add_row(..., Sparkline(history.get(ac['hex'], [0])))
```

## Testing & Verification

1. **Start the script** in a terminal
2. **Verify updates** - aircraft should appear and update
3. **Check colors** - military should be highlighted
4. **Test scrolling** - handle many aircraft gracefully

### SSH Sessions

Works great over SSH for remote monitoring:

```shell
ssh user@server "python terminal_dashboard.py"
```

## Troubleshooting

### Colors Not Showing

**Problem:** See escape codes instead of colors
**Solution:**
- Use a terminal that supports ANSI
- Check `TERM` environment variable
- Try `export TERM=xterm-256color`

### Flickering Display

**Problem:** Screen flickers during updates
**Solution:**
- Use `rich.live.Live` for smooth updates
- Reduce refresh rate
- Use double-buffering

### Terminal Too Small

**Problem:** Layout broken on small terminals
**Solution:**
- Add terminal size detection
- Create compact layouts for small screens

```python
import shutil
cols, rows = shutil.get_terminal_size()
if cols < 80:
    # Use compact layout
```

## Related Recipes

- [Build a Live Dashboard](/docs/live-dashboard) - Web dashboard
- [Leaflet.js Map](/docs/leaflet-map) - Map display
- [OBS Browser Source](/docs/obs-overlay) - Stream overlay
