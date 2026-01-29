---
title: Aircraft Statistics
excerpt: Daily and weekly flight statistics and reports.
hidden: false
recipe:
  color: '#6366F1'
  icon: "📊"
difficulty: intermediate
tags: [data, statistics, analytics, reports]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Python: `pip install sseclient-py requests`
- Node.js: `npm install eventsource`
- Go: No external dependencies required

## What You'll Build

A statistics tracking system that monitors aircraft activity and generates daily and weekly reports. This includes:

- **Unique aircraft counts** - Track distinct aircraft seen
- **Type distribution** - Count aircraft by type (B738, A320, etc.)
- **Military tracking** - Monitor military aircraft activity
- **Busiest hours** - Identify peak traffic periods
- **Report generation** - Export stats as JSON or HTML

```shell Shell
pip install sseclient-py requests
python aircraft_stats.py
# Output: stats_2024-01-15.json, stats_weekly.html
```

```go Go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "html/template"
    "net/http"
    "os"
    "sort"
    "strings"
    "sync"
    "time"
)

type Stats struct {
    Date           string         `json:"date"`
    UniqueAircraft int            `json:"unique_aircraft"`
    TotalSightings int            `json:"total_sightings"`
    MilitaryCount  int            `json:"military_count"`
    TypeCounts     map[string]int `json:"type_counts"`
    HourlyCounts   map[int]int    `json:"hourly_counts"`
}

var (
    stats    Stats
    seen     = make(map[string]bool)
    mu       sync.Mutex
    skyspyURL string
)

func main() {
    skyspyURL = os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    stats = Stats{
        Date:         time.Now().Format("2006-01-02"),
        TypeCounts:   make(map[string]int),
        HourlyCounts: make(map[int]int),
    }

    // Start report generation goroutine
    go generateReports()

    fmt.Printf("Tracking statistics from %s\n", skyspyURL)

    for {
        resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
        if err != nil {
            fmt.Printf("Connection failed: %v, retrying in 5s\n", err)
            time.Sleep(5 * time.Second)
            continue
        }

        scanner := bufio.NewScanner(resp.Body)
        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                processEvent(line[5:])
            }
        }
        resp.Body.Close()
    }
}

func processEvent(data string) {
    var event map[string]interface{}
    if err := json.Unmarshal([]byte(data), &event); err != nil {
        return
    }

    aircraft, ok := event["aircraft"].([]interface{})
    if !ok {
        return
    }

    mu.Lock()
    defer mu.Unlock()

    hour := time.Now().Hour()

    for _, a := range aircraft {
        ac := a.(map[string]interface{})
        hex := fmt.Sprint(ac["hex"])

        stats.TotalSightings++
        stats.HourlyCounts[hour]++

        if !seen[hex] {
            seen[hex] = true
            stats.UniqueAircraft++

            if acType, ok := ac["type"].(string); ok && acType != "" {
                stats.TypeCounts[acType]++
            }

            if military, ok := ac["military"].(bool); ok && military {
                stats.MilitaryCount++
            }
        }
    }
}

func generateReports() {
    ticker := time.NewTicker(5 * time.Minute)
    defer ticker.Stop()

    for range ticker.C {
        mu.Lock()
        saveJSONReport()
        saveHTMLReport()
        mu.Unlock()
    }
}

func saveJSONReport() {
    filename := fmt.Sprintf("stats_%s.json", stats.Date)
    data, _ := json.MarshalIndent(stats, "", "  ")
    os.WriteFile(filename, data, 0644)
    fmt.Printf("Updated %s\n", filename)
}

func saveHTMLReport() {
    tmpl := `<!DOCTYPE html>
<html>
<head><title>Aircraft Statistics - {{.Date}}</title></head>
<body>
<h1>Aircraft Statistics - {{.Date}}</h1>
<ul>
<li>Unique Aircraft: {{.UniqueAircraft}}</li>
<li>Total Sightings: {{.TotalSightings}}</li>
<li>Military Aircraft: {{.MilitaryCount}}</li>
</ul>
</body>
</html>`
    t, _ := template.New("report").Parse(tmpl)
    f, _ := os.Create(fmt.Sprintf("stats_%s.html", stats.Date))
    t.Execute(f, stats)
    f.Close()
}
```

```python Python
import json
import os
import sys
from datetime import datetime, date, timedelta
from collections import defaultdict
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
OUTPUT_DIR = os.getenv("OUTPUT_DIR", ".")
REPORT_INTERVAL = int(os.getenv("REPORT_INTERVAL", "300"))  # 5 minutes

class AircraftStats:
    def __init__(self):
        self.reset_daily()
        self.weekly_stats = []

    def reset_daily(self):
        self.date = date.today()
        self.unique_aircraft = set()
        self.total_sightings = 0
        self.military_count = 0
        self.type_counts = defaultdict(int)
        self.hourly_counts = defaultdict(int)

    def record_aircraft(self, aircraft):
        hex_code = aircraft.get("hex")
        hour = datetime.now().hour

        self.total_sightings += 1
        self.hourly_counts[hour] += 1

        if hex_code not in self.unique_aircraft:
            self.unique_aircraft.add(hex_code)

            ac_type = aircraft.get("type", "Unknown")
            if ac_type:
                self.type_counts[ac_type] += 1

            if aircraft.get("military", False):
                self.military_count += 1

    def get_busiest_hours(self, top_n=5):
        sorted_hours = sorted(self.hourly_counts.items(), key=lambda x: x[1], reverse=True)
        return sorted_hours[:top_n]

    def get_top_types(self, top_n=10):
        sorted_types = sorted(self.type_counts.items(), key=lambda x: x[1], reverse=True)
        return sorted_types[:top_n]

    def to_dict(self):
        return {
            "date": self.date.isoformat(),
            "unique_aircraft": len(self.unique_aircraft),
            "total_sightings": self.total_sightings,
            "military_count": self.military_count,
            "type_counts": dict(self.type_counts),
            "hourly_counts": dict(self.hourly_counts),
            "busiest_hours": self.get_busiest_hours(),
            "top_types": self.get_top_types(),
        }

    def save_json_report(self):
        filename = os.path.join(OUTPUT_DIR, f"stats_{self.date.isoformat()}.json")
        with open(filename, "w") as f:
            json.dump(self.to_dict(), f, indent=2)
        print(f"Updated {filename}")

    def save_html_report(self):
        filename = os.path.join(OUTPUT_DIR, f"stats_{self.date.isoformat()}.html")
        html = f"""<!DOCTYPE html>
<html>
<head>
    <title>Aircraft Statistics - {self.date}</title>
    <style>
        body {{ font-family: Arial, sans-serif; margin: 40px; }}
        .stat {{ background: #f0f0f0; padding: 20px; margin: 10px 0; border-radius: 8px; }}
        .number {{ font-size: 2em; font-weight: bold; color: #6366F1; }}
        table {{ border-collapse: collapse; width: 100%; margin: 20px 0; }}
        th, td {{ border: 1px solid #ddd; padding: 12px; text-align: left; }}
        th {{ background: #6366F1; color: white; }}
    </style>
</head>
<body>
    <h1>Aircraft Statistics - {self.date}</h1>

    <div class="stat">
        <div class="number">{len(self.unique_aircraft)}</div>
        <div>Unique Aircraft</div>
    </div>

    <div class="stat">
        <div class="number">{self.total_sightings}</div>
        <div>Total Sightings</div>
    </div>

    <div class="stat">
        <div class="number">{self.military_count}</div>
        <div>Military Aircraft</div>
    </div>

    <h2>Top Aircraft Types</h2>
    <table>
        <tr><th>Type</th><th>Count</th></tr>
        {"".join(f"<tr><td>{t}</td><td>{c}</td></tr>" for t, c in self.get_top_types())}
    </table>

    <h2>Busiest Hours</h2>
    <table>
        <tr><th>Hour</th><th>Sightings</th></tr>
        {"".join(f"<tr><td>{h}:00</td><td>{c}</td></tr>" for h, c in self.get_busiest_hours())}
    </table>
</body>
</html>"""
        with open(filename, "w") as f:
            f.write(html)
        print(f"Updated {filename}")


def main():
    stats = AircraftStats()
    last_report = datetime.now()

    try:
        while True:
            try:
                response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
                response.raise_for_status()
                client = sseclient.SSEClient(response)

                print(f"Connected to {SKYSPY_URL}, tracking statistics...")

                for event in client.events():
                    # Check for date rollover
                    if date.today() != stats.date:
                        stats.save_json_report()
                        stats.save_html_report()
                        stats.weekly_stats.append(stats.to_dict())
                        stats.reset_daily()
                        print(f"New day - reset statistics for {stats.date}")

                    # Generate reports periodically
                    if (datetime.now() - last_report).total_seconds() >= REPORT_INTERVAL:
                        stats.save_json_report()
                        stats.save_html_report()
                        last_report = datetime.now()

                    if event.event in ["aircraft_update", "aircraft_new"]:
                        try:
                            data = json.loads(event.data)
                        except json.JSONDecodeError:
                            continue

                        for aircraft in data.get("aircraft", []):
                            stats.record_aircraft(aircraft)

            except requests.RequestException as e:
                print(f"Connection error: {e}, reconnecting in 5s...")
                import time
                time.sleep(5)

    except KeyboardInterrupt:
        print("\nSaving final reports...")
        stats.save_json_report()
        stats.save_html_report()


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const fs = require('fs');
const path = require('path');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const OUTPUT_DIR = process.env.OUTPUT_DIR || '.';
const REPORT_INTERVAL = parseInt(process.env.REPORT_INTERVAL || '300') * 1000;

class AircraftStats {
  constructor() {
    this.resetDaily();
  }

  resetDaily() {
    this.date = new Date().toISOString().split('T')[0];
    this.uniqueAircraft = new Set();
    this.totalSightings = 0;
    this.militaryCount = 0;
    this.typeCounts = {};
    this.hourlyCounts = {};
  }

  recordAircraft(aircraft) {
    const hex = aircraft.hex;
    const hour = new Date().getHours();

    this.totalSightings++;
    this.hourlyCounts[hour] = (this.hourlyCounts[hour] || 0) + 1;

    if (!this.uniqueAircraft.has(hex)) {
      this.uniqueAircraft.add(hex);

      const type = aircraft.type || 'Unknown';
      if (type) {
        this.typeCounts[type] = (this.typeCounts[type] || 0) + 1;
      }

      if (aircraft.military) {
        this.militaryCount++;
      }
    }
  }

  getBusiestHours(topN = 5) {
    return Object.entries(this.hourlyCounts)
      .sort((a, b) => b[1] - a[1])
      .slice(0, topN);
  }

  getTopTypes(topN = 10) {
    return Object.entries(this.typeCounts)
      .sort((a, b) => b[1] - a[1])
      .slice(0, topN);
  }

  toJSON() {
    return {
      date: this.date,
      unique_aircraft: this.uniqueAircraft.size,
      total_sightings: this.totalSightings,
      military_count: this.militaryCount,
      type_counts: this.typeCounts,
      hourly_counts: this.hourlyCounts,
      busiest_hours: this.getBusiestHours(),
      top_types: this.getTopTypes()
    };
  }

  saveJSONReport() {
    const filename = path.join(OUTPUT_DIR, `stats_${this.date}.json`);
    fs.writeFileSync(filename, JSON.stringify(this.toJSON(), null, 2));
    console.log(`Updated ${filename}`);
  }

  saveHTMLReport() {
    const filename = path.join(OUTPUT_DIR, `stats_${this.date}.html`);
    const topTypes = this.getTopTypes()
      .map(([t, c]) => `<tr><td>${t}</td><td>${c}</td></tr>`)
      .join('');
    const busiestHours = this.getBusiestHours()
      .map(([h, c]) => `<tr><td>${h}:00</td><td>${c}</td></tr>`)
      .join('');

    const html = `<!DOCTYPE html>
<html>
<head>
    <title>Aircraft Statistics - ${this.date}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        .stat { background: #f0f0f0; padding: 20px; margin: 10px 0; border-radius: 8px; }
        .number { font-size: 2em; font-weight: bold; color: #6366F1; }
        table { border-collapse: collapse; width: 100%; margin: 20px 0; }
        th, td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        th { background: #6366F1; color: white; }
    </style>
</head>
<body>
    <h1>Aircraft Statistics - ${this.date}</h1>

    <div class="stat">
        <div class="number">${this.uniqueAircraft.size}</div>
        <div>Unique Aircraft</div>
    </div>

    <div class="stat">
        <div class="number">${this.totalSightings}</div>
        <div>Total Sightings</div>
    </div>

    <div class="stat">
        <div class="number">${this.militaryCount}</div>
        <div>Military Aircraft</div>
    </div>

    <h2>Top Aircraft Types</h2>
    <table>
        <tr><th>Type</th><th>Count</th></tr>
        ${topTypes}
    </table>

    <h2>Busiest Hours</h2>
    <table>
        <tr><th>Hour</th><th>Sightings</th></tr>
        ${busiestHours}
    </table>
</body>
</html>`;
    fs.writeFileSync(filename, html);
    console.log(`Updated ${filename}`);
  }
}

const stats = new AircraftStats();
let currentDate = stats.date;

// Generate reports periodically
setInterval(() => {
  stats.saveJSONReport();
  stats.saveHTMLReport();
}, REPORT_INTERVAL);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.onerror = () => {
  console.error('SSE connection error, will retry...');
};

es.onopen = () => {
  console.log(`Connected to ${SKYSPY_URL}, tracking statistics...`);
};

es.addEventListener('aircraft_update', (e) => {
  // Check for date rollover
  const today = new Date().toISOString().split('T')[0];
  if (today !== currentDate) {
    stats.saveJSONReport();
    stats.saveHTMLReport();
    stats.resetDaily();
    currentDate = today;
    console.log(`New day - reset statistics for ${currentDate}`);
  }

  let data;
  try {
    data = JSON.parse(e.data);
  } catch (err) {
    return;
  }

  for (const aircraft of data.aircraft || []) {
    stats.recordAircraft(aircraft);
  }
});

process.on('SIGINT', () => {
  console.log('\nSaving final reports...');
  stats.saveJSONReport();
  stats.saveHTMLReport();
  process.exit(0);
});
```

```json Response Example
{
  "date": "2024-01-15",
  "unique_aircraft": 147,
  "total_sightings": 2834,
  "military_count": 12,
  "type_counts": {
    "B738": 32,
    "A320": 28,
    "B77W": 15,
    "A321": 14,
    "E75L": 11
  },
  "hourly_counts": {
    "8": 245,
    "9": 312,
    "10": 289,
    "17": 356,
    "18": 298
  },
  "busiest_hours": [[17, 356], [9, 312], [18, 298]],
  "top_types": [["B738", 32], ["A320", 28], ["B77W", 15]]
}
```

## Initialize Statistics Tracking

<!-- shell@1-3 -->
<!-- go@15-32 -->
<!-- python@14-28 -->
<!-- javascript@11-23 -->

Set up data structures to track unique aircraft, type distribution, military count, and hourly activity. The stats object maintains running totals throughout the day.

## Record Aircraft Data

<!-- go@60-82 -->
<!-- python@30-45 -->
<!-- javascript@25-44 -->

Process each aircraft sighting and update statistics. Track unique aircraft using hex codes, count aircraft types, identify military activity, and log hourly traffic patterns.

## Generate Reports

<!-- go@88-115 -->
<!-- python@60-95 -->
<!-- javascript@56-98 -->

Periodically generate JSON and HTML reports with current statistics. Reports include summary stats, top aircraft types, and busiest hours analysis.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `OUTPUT_DIR` | Directory for report files | Current directory |
| `REPORT_INTERVAL` | Seconds between report generation | `300` (5 minutes) |

### Customizing Statistics

Track additional metrics:

```python
# Track altitude distribution
self.altitude_ranges = {
    "low": 0,      # < 10,000 ft
    "medium": 0,   # 10,000 - 30,000 ft
    "high": 0      # > 30,000 ft
}

alt = aircraft.get("alt", 0)
if alt < 10000:
    self.altitude_ranges["low"] += 1
elif alt < 30000:
    self.altitude_ranges["medium"] += 1
else:
    self.altitude_ranges["high"] += 1
```

### Weekly Aggregation

Combine daily stats into weekly reports:

```python
def generate_weekly_report(self):
    week_start = date.today() - timedelta(days=7)
    weekly = {
        "period": f"{week_start} to {date.today()}",
        "total_unique": sum(d["unique_aircraft"] for d in self.weekly_stats),
        "total_military": sum(d["military_count"] for d in self.weekly_stats),
        "daily_average": sum(d["unique_aircraft"] for d in self.weekly_stats) / 7,
    }
    return weekly
```

## Testing & Verification

1. **Start the tracker** using any of the code examples
2. **Check console output** - You should see connection confirmation
3. **Wait for first report** - After REPORT_INTERVAL, files should appear
4. **Verify statistics** - Open JSON/HTML files to confirm data collection

### Sample JSON Output

```json
{
  "date": "2024-01-15",
  "unique_aircraft": 147,
  "total_sightings": 2834,
  "military_count": 12,
  "busiest_hours": [[17, 356], [9, 312]]
}
```

### Sample HTML Report

The HTML report provides a styled dashboard with:
- Large stat cards for key metrics
- Sortable tables for type distribution
- Hourly traffic visualization

## Troubleshooting

### No Reports Generated

**Problem:** Report files are not being created
**Solution:**
- Check write permissions in OUTPUT_DIR
- Verify REPORT_INTERVAL hasn't elapsed yet
- Look for error messages in console output

### Inaccurate Counts

**Problem:** Statistics don't match expected values
**Solution:**
- Unique aircraft uses hex codes - same aircraft counted once
- Total sightings counts every update received
- Military flag depends on SkySpy's aircraft database

### High Memory Usage

**Problem:** Script uses increasing memory over time
**Solution:**
- The unique aircraft set grows throughout the day
- Memory resets at midnight with daily rollover
- Consider periodic cleanup for very busy locations

### Missing Aircraft Types

**Problem:** Many aircraft show as "Unknown" type
**Solution:**
- Type data depends on SkySpy's aircraft database
- Some aircraft may not transmit type information
- Check if aircraft database is properly configured

## Related Recipes

- [Export to CSV](/docs/export-csv) - Raw data logging
- [SQLite Local DB](/docs/sqlite-db) - Persistent statistics storage
- [Grafana Dashboard](/docs/grafana-dashboard) - Real-time visualization
- [Discord Alert Bot](/docs/discord-alert-bot) - Alert on milestones
