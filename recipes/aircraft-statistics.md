---
title: Aircraft Statistics
description: Generate daily and weekly flight statistics.
hidden: false
recipe:
  color: '#3B82F6'
  icon: 📊
---
```shell Shell
sqlite3 aircraft.db "SELECT type, COUNT(*) as count FROM sightings WHERE seen_at > datetime('now', '-1 day') GROUP BY type ORDER BY count DESC LIMIT 10"
```

```go Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"sort"
)

type Stats struct {
	Total      int
	ByType     map[string]int
	Military   int
	MaxAlt     int
	AvgAlt     float64
}

func main() {
	resp, _ := http.Get("http://localhost:5000/api/v1/aircraft")
	var data struct {
		Aircraft []struct {
			Type     string `json:"type"`
			Alt      int    `json:"alt"`
			Military bool   `json:"military"`
		} `json:"aircraft"`
	}
	json.NewDecoder(resp.Body).Decode(&data)
	resp.Body.Close()

	stats := Stats{ByType: make(map[string]int)}
	totalAlt := 0

	for _, ac := range data.Aircraft {
		stats.Total++
		stats.ByType[ac.Type]++
		if ac.Military { stats.Military++ }
		if ac.Alt > stats.MaxAlt { stats.MaxAlt = ac.Alt }
		totalAlt += ac.Alt
	}
	if stats.Total > 0 { stats.AvgAlt = float64(totalAlt) / float64(stats.Total) }

	fmt.Printf("Total: %d | Military: %d | Max Alt: %d ft | Avg Alt: %.0f ft\n",
		stats.Total, stats.Military, stats.MaxAlt, stats.AvgAlt)
}
```

```python Python
from collections import Counter
import requests

SKYSPY_URL = "http://localhost:5000"

response = requests.get(f"{SKYSPY_URL}/api/v1/aircraft")
aircraft = response.json().get("aircraft", [])

stats = {
    "total": len(aircraft),
    "military": sum(1 for ac in aircraft if ac.get("military")),
    "by_type": Counter(ac.get("type", "Unknown") for ac in aircraft),
    "max_alt": max((ac.get("alt", 0) for ac in aircraft), default=0),
    "avg_alt": sum(ac.get("alt", 0) for ac in aircraft) / len(aircraft) if aircraft else 0,
}

print(f"Total: {stats['total']} | Military: {stats['military']}")
print(f"Max Alt: {stats['max_alt']:,} ft | Avg Alt: {stats['avg_alt']:,.0f} ft")
print("\nTop aircraft types:")
for ac_type, count in stats["by_type"].most_common(5):
    print(f"  {ac_type}: {count}")
```

```javascript JavaScript
const SKYSPY_URL = 'http://localhost:5000';

async function getStatistics() {
    const response = await fetch(`${SKYSPY_URL}/api/v1/aircraft`);
    const data = await response.json();
    const aircraft = data.aircraft || [];

    const stats = {
        total: aircraft.length,
        military: aircraft.filter(ac => ac.military).length,
        byType: {},
        maxAlt: Math.max(...aircraft.map(ac => ac.alt || 0)),
        avgAlt: aircraft.reduce((sum, ac) => sum + (ac.alt || 0), 0) / aircraft.length || 0,
    };

    for (const ac of aircraft) {
        const type = ac.type || 'Unknown';
        stats.byType[type] = (stats.byType[type] || 0) + 1;
    }

    console.log(`Total: ${stats.total} | Military: ${stats.military}`);
    console.log(`Max Alt: ${stats.maxAlt.toLocaleString()} ft | Avg Alt: ${stats.avgAlt.toFixed(0)} ft`);

    const sorted = Object.entries(stats.byType).sort((a, b) => b[1] - a[1]).slice(0, 5);
    console.log('\nTop aircraft types:');
    for (const [type, count] of sorted) {
        console.log(`  ${type}: ${count}`);
    }
}

getStatistics();
```

```json Response Example
{"total": 150, "military": 5, "max_alt": 43000, "avg_alt": 28500, "top_type": "B738"}
```

# Fetch Current Data

<!-- shell@1 -->
<!-- go@17-26 -->
<!-- python@5-6 -->
<!-- javascript@4-6 -->

Get all current aircraft data. For historical stats, query your database instead of the live API.

# Calculate Statistics

<!-- go@28-38 -->
<!-- python@8-14 -->
<!-- javascript@8-18 -->

Aggregate counts by type, count military aircraft, and calculate altitude metrics.

# Display Results

<!-- go@40-41 -->
<!-- python@16-21 -->
<!-- javascript@20-26 -->

Output summary statistics. Track over time for trend analysis and daily reports.
