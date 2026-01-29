---
title: Rare Aircraft Spotter
excerpt: Alert on unusual and rare aircraft types.
hidden: false
recipe:
  color: '#EC4899'
  icon: "\U0001F48E"
difficulty: intermediate
tags: [specialty, rare, tracking, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- `sseclient-py` and `requests` for Python
- `eventsource` package for Node.js
- Basic understanding of aircraft type codes

## What You'll Build

A rare aircraft detector that monitors the SSE stream and alerts when unusual, historic, or uncommon aircraft types appear in your coverage area. Aircraft are categorized and displayed with relevant emojis for quick identification.

```shell Shell
pip install sseclient-py requests
export SKYSPY_URL="http://localhost:5000"
python rare_spotter.py
```

```go Go
package main

import (
	"bufio"
	"encoding/json"
	"fmt"
	"net/http"
	"os"
	"strings"
	"time"
)

// Aircraft categories with emojis
var categoryEmoji = map[string]string{
	"warbird":    "\U0001F985",  // Eagle
	"jumbo":      "\U00002708",  // Airplane
	"cargo":      "\U0001F4E6",  // Package
	"military":   "\U0001F396",  // Military medal
	"government": "\U0001F3DB",  // Classical building
	"historic":   "\U0001F3F0",  // Castle
}

// RareAircraft holds information about a rare aircraft type
type RareAircraft struct {
	Name     string
	Category string
	Notes    string
}

// Rare aircraft types database
var rareAircraft = map[string]RareAircraft{
	// Warbirds - WWII era aircraft
	"B17":  {Name: "Boeing B-17 Flying Fortress", Category: "warbird", Notes: "WWII heavy bomber"},
	"B29":  {Name: "Boeing B-29 Superfortress", Category: "warbird", Notes: "WWII heavy bomber"},
	"P51":  {Name: "North American P-51 Mustang", Category: "warbird", Notes: "WWII fighter"},
	"F4U":  {Name: "Vought F4U Corsair", Category: "warbird", Notes: "WWII fighter"},
	"TBM":  {Name: "Grumman TBM Avenger", Category: "warbird", Notes: "WWII torpedo bomber"},
	"B24":  {Name: "Consolidated B-24 Liberator", Category: "warbird", Notes: "WWII heavy bomber"},
	"B25":  {Name: "North American B-25 Mitchell", Category: "warbird", Notes: "WWII medium bomber"},
	"C47":  {Name: "Douglas C-47 Skytrain", Category: "warbird", Notes: "Military DC-3"},
	"DC3":  {Name: "Douglas DC-3", Category: "historic", Notes: "1930s airliner"},
	"SPIT": {Name: "Supermarine Spitfire", Category: "warbird", Notes: "WWII British fighter"},
	"ME09": {Name: "Messerschmitt Bf 109", Category: "warbird", Notes: "WWII German fighter"},
	"P38":  {Name: "Lockheed P-38 Lightning", Category: "warbird", Notes: "WWII fighter"},
	"P40":  {Name: "Curtiss P-40 Warhawk", Category: "warbird", Notes: "WWII fighter"},

	// Large/Jumbo aircraft
	"A388": {Name: "Airbus A380-800", Category: "jumbo", Notes: "World's largest passenger aircraft"},
	"A380": {Name: "Airbus A380", Category: "jumbo", Notes: "Double-deck widebody"},
	"B744": {Name: "Boeing 747-400", Category: "jumbo", Notes: "Classic jumbo jet"},
	"B748": {Name: "Boeing 747-8", Category: "jumbo", Notes: "Latest 747 variant"},
	"B74S": {Name: "Boeing 747SP", Category: "jumbo", Notes: "Short-body long-range 747"},
	"CONC": {Name: "Aerospatiale Concorde", Category: "historic", Notes: "Supersonic airliner"},

	// Heavy cargo aircraft
	"A124": {Name: "Antonov An-124 Ruslan", Category: "cargo", Notes: "Heavy cargo transport"},
	"A225": {Name: "Antonov An-225 Mriya", Category: "cargo", Notes: "World's largest aircraft"},
	"C5":   {Name: "Lockheed C-5 Galaxy", Category: "cargo", Notes: "US strategic airlifter"},
	"C5M":  {Name: "Lockheed C-5M Super Galaxy", Category: "cargo", Notes: "Modernized C-5"},
	"C17":  {Name: "Boeing C-17 Globemaster III", Category: "cargo", Notes: "Strategic/tactical airlifter"},
	"B52":  {Name: "Boeing B-52 Stratofortress", Category: "military", Notes: "Strategic bomber"},
	"B1":   {Name: "Rockwell B-1 Lancer", Category: "military", Notes: "Supersonic bomber"},
	"B2":   {Name: "Northrop B-2 Spirit", Category: "military", Notes: "Stealth bomber"},

	// Government/Special mission aircraft
	"E4B":  {Name: "Boeing E-4B Nightwatch", Category: "government", Notes: "Airborne command post"},
	"VC25": {Name: "Boeing VC-25A", Category: "government", Notes: "Air Force One"},
	"C32":  {Name: "Boeing C-32A", Category: "government", Notes: "Air Force Two"},
	"E6B":  {Name: "Boeing E-6B Mercury", Category: "government", Notes: "TACAMO aircraft"},
	"E3TF": {Name: "Boeing E-3 Sentry AWACS", Category: "government", Notes: "Airborne warning"},
	"E8":   {Name: "Northrop E-8 JSTARS", Category: "government", Notes: "Battle management"},

	// Other rare/historic types
	"DC6":  {Name: "Douglas DC-6", Category: "historic", Notes: "1940s propliner"},
	"DC4":  {Name: "Douglas DC-4", Category: "historic", Notes: "1940s propliner"},
	"L049": {Name: "Lockheed Constellation", Category: "historic", Notes: "Classic propliner"},
	"AN2":  {Name: "Antonov An-2", Category: "historic", Notes: "Soviet biplane"},
	"C130": {Name: "Lockheed C-130 Hercules", Category: "military", Notes: "Tactical airlifter"},
	"SR71": {Name: "Lockheed SR-71 Blackbird", Category: "military", Notes: "Reconnaissance aircraft"},
	"U2":   {Name: "Lockheed U-2", Category: "military", Notes: "High-altitude reconnaissance"},
}

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	cooldownMinutes := 30
	alerted := make(map[string]time.Time)

	fmt.Printf("%s Rare Aircraft Spotter\n", categoryEmoji["historic"])
	fmt.Printf("Monitoring for %d rare aircraft types...\n\n", len(rareAircraft))

	for {
		resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
		if err != nil {
			fmt.Printf("Connection error: %v, retrying...\n", err)
			time.Sleep(5 * time.Second)
			continue
		}

		scanner := bufio.NewScanner(resp.Body)
		for scanner.Scan() {
			line := scanner.Text()
			if !strings.HasPrefix(line, "data:") {
				continue
			}

			var data map[string]interface{}
			if err := json.Unmarshal([]byte(line[5:]), &data); err != nil {
				continue
			}

			aircraft, ok := data["aircraft"].([]interface{})
			if !ok {
				continue
			}

			for _, a := range aircraft {
				ac := a.(map[string]interface{})
				typeCode := strings.ToUpper(fmt.Sprint(ac["t"]))
				hex := fmt.Sprint(ac["hex"])

				if rare, isRare := rareAircraft[typeCode]; isRare {
					// Check cooldown
					if lastAlert, seen := alerted[hex]; seen {
						if time.Since(lastAlert) < time.Duration(cooldownMinutes)*time.Minute {
							continue
						}
					}

					emoji := categoryEmoji[rare.Category]
					fmt.Printf("\n%s RARE AIRCRAFT DETECTED!\n", emoji)
					fmt.Printf("   Type: %s (%s)\n", rare.Name, typeCode)
					fmt.Printf("   Category: %s\n", strings.Title(rare.Category))
					fmt.Printf("   Callsign: %s\n", ac["flight"])
					fmt.Printf("   Altitude: %v ft\n", ac["alt"])
					fmt.Printf("   Speed: %v kts\n", ac["gs"])
					fmt.Printf("   Position: %v, %v\n", ac["lat"], ac["lon"])
					fmt.Printf("   Notes: %s\n", rare.Notes)

					alerted[hex] = time.Now()
				}
			}
		}
		resp.Body.Close()
	}
}
```

```python Python
import json
import os
import time
from datetime import datetime, timedelta
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
COOLDOWN_MINUTES = int(os.getenv("COOLDOWN_MINUTES", "30"))

# Category emojis for display
CATEGORY_EMOJI = {
    "warbird": "\U0001F985",     # Eagle
    "jumbo": "\U00002708",       # Airplane
    "cargo": "\U0001F4E6",       # Package
    "military": "\U0001F396",    # Military medal
    "government": "\U0001F3DB",  # Classical building
    "historic": "\U0001F3F0",    # Castle
}

# Rare aircraft types database
RARE_AIRCRAFT = {
    # Warbirds - WWII era aircraft
    "B17": {"name": "Boeing B-17 Flying Fortress", "category": "warbird", "notes": "WWII heavy bomber"},
    "B29": {"name": "Boeing B-29 Superfortress", "category": "warbird", "notes": "WWII heavy bomber"},
    "P51": {"name": "North American P-51 Mustang", "category": "warbird", "notes": "WWII fighter"},
    "F4U": {"name": "Vought F4U Corsair", "category": "warbird", "notes": "WWII fighter"},
    "TBM": {"name": "Grumman TBM Avenger", "category": "warbird", "notes": "WWII torpedo bomber"},
    "B24": {"name": "Consolidated B-24 Liberator", "category": "warbird", "notes": "WWII heavy bomber"},
    "B25": {"name": "North American B-25 Mitchell", "category": "warbird", "notes": "WWII medium bomber"},
    "C47": {"name": "Douglas C-47 Skytrain", "category": "warbird", "notes": "Military DC-3"},
    "DC3": {"name": "Douglas DC-3", "category": "historic", "notes": "1930s airliner"},
    "SPIT": {"name": "Supermarine Spitfire", "category": "warbird", "notes": "WWII British fighter"},
    "ME09": {"name": "Messerschmitt Bf 109", "category": "warbird", "notes": "WWII German fighter"},
    "P38": {"name": "Lockheed P-38 Lightning", "category": "warbird", "notes": "WWII fighter"},
    "P40": {"name": "Curtiss P-40 Warhawk", "category": "warbird", "notes": "WWII fighter"},

    # Large/Jumbo aircraft
    "A388": {"name": "Airbus A380-800", "category": "jumbo", "notes": "World's largest passenger aircraft"},
    "A380": {"name": "Airbus A380", "category": "jumbo", "notes": "Double-deck widebody"},
    "B744": {"name": "Boeing 747-400", "category": "jumbo", "notes": "Classic jumbo jet"},
    "B748": {"name": "Boeing 747-8", "category": "jumbo", "notes": "Latest 747 variant"},
    "B74S": {"name": "Boeing 747SP", "category": "jumbo", "notes": "Short-body long-range 747"},
    "CONC": {"name": "Aerospatiale Concorde", "category": "historic", "notes": "Supersonic airliner"},

    # Heavy cargo aircraft
    "A124": {"name": "Antonov An-124 Ruslan", "category": "cargo", "notes": "Heavy cargo transport"},
    "A225": {"name": "Antonov An-225 Mriya", "category": "cargo", "notes": "World's largest aircraft"},
    "C5": {"name": "Lockheed C-5 Galaxy", "category": "cargo", "notes": "US strategic airlifter"},
    "C5M": {"name": "Lockheed C-5M Super Galaxy", "category": "cargo", "notes": "Modernized C-5"},
    "C17": {"name": "Boeing C-17 Globemaster III", "category": "cargo", "notes": "Strategic/tactical airlifter"},
    "B52": {"name": "Boeing B-52 Stratofortress", "category": "military", "notes": "Strategic bomber"},
    "B1": {"name": "Rockwell B-1 Lancer", "category": "military", "notes": "Supersonic bomber"},
    "B2": {"name": "Northrop B-2 Spirit", "category": "military", "notes": "Stealth bomber"},

    # Government/Special mission aircraft
    "E4B": {"name": "Boeing E-4B Nightwatch", "category": "government", "notes": "Airborne command post"},
    "VC25": {"name": "Boeing VC-25A", "category": "government", "notes": "Air Force One"},
    "C32": {"name": "Boeing C-32A", "category": "government", "notes": "Air Force Two"},
    "E6B": {"name": "Boeing E-6B Mercury", "category": "government", "notes": "TACAMO aircraft"},
    "E3TF": {"name": "Boeing E-3 Sentry AWACS", "category": "government", "notes": "Airborne warning"},
    "E8": {"name": "Northrop E-8 JSTARS", "category": "government", "notes": "Battle management"},

    # Other rare/historic types
    "DC6": {"name": "Douglas DC-6", "category": "historic", "notes": "1940s propliner"},
    "DC4": {"name": "Douglas DC-4", "category": "historic", "notes": "1940s propliner"},
    "L049": {"name": "Lockheed Constellation", "category": "historic", "notes": "Classic propliner"},
    "AN2": {"name": "Antonov An-2", "category": "historic", "notes": "Soviet biplane"},
    "C130": {"name": "Lockheed C-130 Hercules", "category": "military", "notes": "Tactical airlifter"},
    "SR71": {"name": "Lockheed SR-71 Blackbird", "category": "military", "notes": "Reconnaissance aircraft"},
    "U2": {"name": "Lockheed U-2", "category": "military", "notes": "High-altitude reconnaissance"},
}

# Track alerted aircraft with timestamps for cooldown
alerted = {}


def on_rare_spotted(hex_code, aircraft, rare_info):
    """Handle rare aircraft detection."""
    category = rare_info["category"]
    emoji = CATEGORY_EMOJI.get(category, "\U0001F48E")

    print(f"\n{emoji} RARE AIRCRAFT DETECTED!")
    print(f"   Type: {rare_info['name']} ({aircraft.get('t', 'Unknown')})")
    print(f"   Category: {category.title()}")
    print(f"   Callsign: {aircraft.get('flight', 'Unknown')}")
    print(f"   Altitude: {aircraft.get('alt', 0):,} ft")
    print(f"   Speed: {aircraft.get('gs', 0)} kts")
    print(f"   Position: {aircraft.get('lat', 0)}, {aircraft.get('lon', 0)}")
    print(f"   Distance: {aircraft.get('distance', 0):.1f} nm")
    print(f"   Notes: {rare_info['notes']}")
    print()


def is_cooldown_expired(hex_code):
    """Check if cooldown period has expired for an aircraft."""
    if hex_code not in alerted:
        return True
    elapsed = datetime.now() - alerted[hex_code]
    return elapsed > timedelta(minutes=COOLDOWN_MINUTES)


def main():
    print(f"\U0001F48E Rare Aircraft Spotter")
    print(f"Connected to {SKYSPY_URL}")
    print(f"Monitoring for {len(RARE_AIRCRAFT)} rare aircraft types...")
    print(f"Cooldown: {COOLDOWN_MINUTES} minutes\n")

    # Print categories being monitored
    categories = set(a["category"] for a in RARE_AIRCRAFT.values())
    for cat in sorted(categories):
        emoji = CATEGORY_EMOJI.get(cat, "\U0001F48E")
        count = sum(1 for a in RARE_AIRCRAFT.values() if a["category"] == cat)
        print(f"   {emoji} {cat.title()}: {count} types")
    print()

    while True:
        try:
            response = requests.get(
                f"{SKYSPY_URL}/api/v1/map/sse",
                stream=True,
                timeout=30
            )
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)

                    for aircraft in data.get("aircraft", []):
                        type_code = aircraft.get("t", "").upper()
                        hex_code = aircraft.get("hex", "")

                        if type_code in RARE_AIRCRAFT and is_cooldown_expired(hex_code):
                            on_rare_spotted(hex_code, aircraft, RARE_AIRCRAFT[type_code])
                            alerted[hex_code] = datetime.now()

        except KeyboardInterrupt:
            print("\nShutting down...")
            break
        except Exception as e:
            print(f"Error: {e}, reconnecting in 5 seconds...")
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const COOLDOWN_MINUTES = parseInt(process.env.COOLDOWN_MINUTES || '30', 10);

// Category emojis for display
const CATEGORY_EMOJI = {
  warbird: '\u{1F985}',     // Eagle
  jumbo: '\u2708',          // Airplane
  cargo: '\u{1F4E6}',       // Package
  military: '\u{1F396}',    // Military medal
  government: '\u{1F3DB}',  // Classical building
  historic: '\u{1F3F0}',    // Castle
};

// Rare aircraft types database
const RARE_AIRCRAFT = {
  // Warbirds - WWII era aircraft
  'B17': { name: 'Boeing B-17 Flying Fortress', category: 'warbird', notes: 'WWII heavy bomber' },
  'B29': { name: 'Boeing B-29 Superfortress', category: 'warbird', notes: 'WWII heavy bomber' },
  'P51': { name: 'North American P-51 Mustang', category: 'warbird', notes: 'WWII fighter' },
  'F4U': { name: 'Vought F4U Corsair', category: 'warbird', notes: 'WWII fighter' },
  'TBM': { name: 'Grumman TBM Avenger', category: 'warbird', notes: 'WWII torpedo bomber' },
  'B24': { name: 'Consolidated B-24 Liberator', category: 'warbird', notes: 'WWII heavy bomber' },
  'B25': { name: 'North American B-25 Mitchell', category: 'warbird', notes: 'WWII medium bomber' },
  'C47': { name: 'Douglas C-47 Skytrain', category: 'warbird', notes: 'Military DC-3' },
  'DC3': { name: 'Douglas DC-3', category: 'historic', notes: '1930s airliner' },
  'SPIT': { name: 'Supermarine Spitfire', category: 'warbird', notes: 'WWII British fighter' },
  'ME09': { name: 'Messerschmitt Bf 109', category: 'warbird', notes: 'WWII German fighter' },
  'P38': { name: 'Lockheed P-38 Lightning', category: 'warbird', notes: 'WWII fighter' },
  'P40': { name: 'Curtiss P-40 Warhawk', category: 'warbird', notes: 'WWII fighter' },

  // Large/Jumbo aircraft
  'A388': { name: 'Airbus A380-800', category: 'jumbo', notes: "World's largest passenger aircraft" },
  'A380': { name: 'Airbus A380', category: 'jumbo', notes: 'Double-deck widebody' },
  'B744': { name: 'Boeing 747-400', category: 'jumbo', notes: 'Classic jumbo jet' },
  'B748': { name: 'Boeing 747-8', category: 'jumbo', notes: 'Latest 747 variant' },
  'B74S': { name: 'Boeing 747SP', category: 'jumbo', notes: 'Short-body long-range 747' },
  'CONC': { name: 'Aerospatiale Concorde', category: 'historic', notes: 'Supersonic airliner' },

  // Heavy cargo aircraft
  'A124': { name: 'Antonov An-124 Ruslan', category: 'cargo', notes: 'Heavy cargo transport' },
  'A225': { name: 'Antonov An-225 Mriya', category: 'cargo', notes: "World's largest aircraft" },
  'C5': { name: 'Lockheed C-5 Galaxy', category: 'cargo', notes: 'US strategic airlifter' },
  'C5M': { name: 'Lockheed C-5M Super Galaxy', category: 'cargo', notes: 'Modernized C-5' },
  'C17': { name: 'Boeing C-17 Globemaster III', category: 'cargo', notes: 'Strategic/tactical airlifter' },
  'B52': { name: 'Boeing B-52 Stratofortress', category: 'military', notes: 'Strategic bomber' },
  'B1': { name: 'Rockwell B-1 Lancer', category: 'military', notes: 'Supersonic bomber' },
  'B2': { name: 'Northrop B-2 Spirit', category: 'military', notes: 'Stealth bomber' },

  // Government/Special mission aircraft
  'E4B': { name: 'Boeing E-4B Nightwatch', category: 'government', notes: 'Airborne command post' },
  'VC25': { name: 'Boeing VC-25A', category: 'government', notes: 'Air Force One' },
  'C32': { name: 'Boeing C-32A', category: 'government', notes: 'Air Force Two' },
  'E6B': { name: 'Boeing E-6B Mercury', category: 'government', notes: 'TACAMO aircraft' },
  'E3TF': { name: 'Boeing E-3 Sentry AWACS', category: 'government', notes: 'Airborne warning' },
  'E8': { name: 'Northrop E-8 JSTARS', category: 'government', notes: 'Battle management' },

  // Other rare/historic types
  'DC6': { name: 'Douglas DC-6', category: 'historic', notes: '1940s propliner' },
  'DC4': { name: 'Douglas DC-4', category: 'historic', notes: '1940s propliner' },
  'L049': { name: 'Lockheed Constellation', category: 'historic', notes: 'Classic propliner' },
  'AN2': { name: 'Antonov An-2', category: 'historic', notes: 'Soviet biplane' },
  'C130': { name: 'Lockheed C-130 Hercules', category: 'military', notes: 'Tactical airlifter' },
  'SR71': { name: 'Lockheed SR-71 Blackbird', category: 'military', notes: 'Reconnaissance aircraft' },
  'U2': { name: 'Lockheed U-2', category: 'military', notes: 'High-altitude reconnaissance' },
};

// Track alerted aircraft with timestamps
const alerted = new Map();

function isCooldownExpired(hexCode) {
  if (!alerted.has(hexCode)) return true;
  const elapsed = Date.now() - alerted.get(hexCode);
  return elapsed > COOLDOWN_MINUTES * 60 * 1000;
}

function onRareSpotted(hexCode, aircraft, rareInfo) {
  const emoji = CATEGORY_EMOJI[rareInfo.category] || '\u{1F48E}';

  console.log(`\n${emoji} RARE AIRCRAFT DETECTED!`);
  console.log(`   Type: ${rareInfo.name} (${aircraft.t || 'Unknown'})`);
  console.log(`   Category: ${rareInfo.category.charAt(0).toUpperCase() + rareInfo.category.slice(1)}`);
  console.log(`   Callsign: ${aircraft.flight || 'Unknown'}`);
  console.log(`   Altitude: ${(aircraft.alt || 0).toLocaleString()} ft`);
  console.log(`   Speed: ${aircraft.gs || 0} kts`);
  console.log(`   Position: ${aircraft.lat || 0}, ${aircraft.lon || 0}`);
  console.log(`   Distance: ${(aircraft.distance || 0).toFixed(1)} nm`);
  console.log(`   Notes: ${rareInfo.notes}`);
  console.log();
}

console.log('\u{1F48E} Rare Aircraft Spotter');
console.log(`Connected to ${SKYSPY_URL}`);
console.log(`Monitoring for ${Object.keys(RARE_AIRCRAFT).length} rare aircraft types...`);
console.log(`Cooldown: ${COOLDOWN_MINUTES} minutes\n`);

// Print categories being monitored
const categories = [...new Set(Object.values(RARE_AIRCRAFT).map(a => a.category))].sort();
for (const cat of categories) {
  const emoji = CATEGORY_EMOJI[cat] || '\u{1F48E}';
  const count = Object.values(RARE_AIRCRAFT).filter(a => a.category === cat).length;
  console.log(`   ${emoji} ${cat.charAt(0).toUpperCase() + cat.slice(1)}: ${count} types`);
}
console.log();

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const typeCode = (aircraft.t || '').toUpperCase();
    const hexCode = aircraft.hex || '';

    if (RARE_AIRCRAFT[typeCode] && isCooldownExpired(hexCode)) {
      onRareSpotted(hexCode, aircraft, RARE_AIRCRAFT[typeCode]);
      alerted.set(hexCode, Date.now());
    }
  }
});

es.addEventListener('aircraft_new', (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const typeCode = (aircraft.t || '').toUpperCase();
    const hexCode = aircraft.hex || '';

    if (RARE_AIRCRAFT[typeCode] && isCooldownExpired(hexCode)) {
      onRareSpotted(hexCode, aircraft, RARE_AIRCRAFT[typeCode]);
      alerted.set(hexCode, Date.now());
    }
  }
});

es.onerror = (err) => {
  console.error('SSE connection error:', err);
};
```

```json Database Format
{
  "B17": {
    "name": "Boeing B-17 Flying Fortress",
    "category": "warbird",
    "notes": "WWII heavy bomber"
  },
  "A388": {
    "name": "Airbus A380-800",
    "category": "jumbo",
    "notes": "World's largest passenger aircraft"
  },
  "E4B": {
    "name": "Boeing E-4B Nightwatch",
    "category": "government",
    "notes": "Airborne command post"
  }
}
```

## Aircraft Categories

The spotter classifies rare aircraft into six categories:

| Category | Emoji | Examples |
|----------|-------|----------|
| Warbird | Eagle | B-17, B-29, P-51, Spitfire, Corsair |
| Jumbo | Airplane | A380, 747-400, 747-8, 747SP |
| Cargo | Package | An-124, An-225, C-5 Galaxy, C-17 |
| Military | Medal | B-52, B-1, B-2, C-130 |
| Government | Building | E-4B, VC-25 (Air Force One), E-6B |
| Historic | Castle | DC-3, DC-6, Constellation, An-2 |

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `COOLDOWN_MINUTES` | Minutes before re-alerting same aircraft | `30` |

### Category Filtering

Monitor only specific categories:

```python
# Only monitor warbirds and historic aircraft
MONITOR_CATEGORIES = {"warbird", "historic"}

# In the main loop:
if type_code in RARE_AIRCRAFT:
    if RARE_AIRCRAFT[type_code]["category"] in MONITOR_CATEGORIES:
        on_rare_spotted(...)
```

### Custom Aircraft Database

Load from an external JSON file:

```python
import json

# Load custom database
with open("rare_aircraft.json") as f:
    RARE_AIRCRAFT = json.load(f)

# Or merge with built-in database
with open("custom_rare.json") as f:
    custom = json.load(f)
    RARE_AIRCRAFT.update(custom)
```

### Adding Notifications

Integrate with Discord or other notification services:

```python
from discord_webhook import DiscordWebhook, DiscordEmbed

def on_rare_spotted(hex_code, aircraft, rare_info):
    # Console output
    emoji = CATEGORY_EMOJI.get(rare_info["category"], "\U0001F48E")
    print(f"{emoji} {rare_info['name']} spotted!")

    # Discord notification
    webhook = DiscordWebhook(url=os.getenv("DISCORD_WEBHOOK"))
    embed = DiscordEmbed(
        title=f"{emoji} Rare Aircraft Spotted!",
        description=rare_info["name"],
        color=0xEC4899
    )
    embed.add_embed_field(name="Type", value=aircraft.get("t", "Unknown"))
    embed.add_embed_field(name="Category", value=rare_info["category"].title())
    embed.add_embed_field(name="Callsign", value=aircraft.get("flight", "Unknown"))
    embed.add_embed_field(name="Altitude", value=f"{aircraft.get('alt', 0):,} ft")
    webhook.add_embed(embed)
    webhook.execute()
```

## Testing & Verification

1. **Start the spotter** and verify connection to SkySpy
2. **Check category counts** - The script displays monitored types at startup
3. **Test with common types** - Temporarily add a frequent type to the database:

```python
# Add a common type for testing
RARE_AIRCRAFT["A320"] = {
    "name": "Test Aircraft",
    "category": "historic",
    "notes": "Testing only - remove after"
}
```

4. **Verify cooldown** - Same aircraft should not alert within cooldown period
5. **Check SSE events** - Monitor browser dev tools or curl to verify events:

```shell
curl -N http://localhost:5000/api/v1/map/sse
```

### Simulating Rare Aircraft

If no rare aircraft are in your area, you can:
1. Check FlightRadar24 or ADS-B Exchange for active rare types
2. Temporarily add common aircraft types to test alerting
3. Use SkySpy's test mode if available

## Troubleshooting

### No Alerts Firing

**Problem:** Rare aircraft visible in SkySpy but no alerts
**Solution:**
- Verify the aircraft type code matches your database (check exact ICAO type designator)
- Type codes are case-insensitive but must match exactly (e.g., "B744" not "747-400")
- Check if cooldown is active for that aircraft hex code

### Wrong Type Codes

**Problem:** Aircraft type doesn't match expected code
**Solution:**
- Different databases may use different type codes
- Look up the official ICAO type designator
- Check SkySpy's aircraft details for the actual type code being broadcast

### SSE Connection Drops

**Problem:** Connection keeps dropping
**Solution:**
- The scripts have automatic reconnection built in
- Check network connectivity to SkySpy server
- Verify SkySpy is running and SSE endpoint is accessible
- Try increasing the timeout value in requests

### High CPU Usage

**Problem:** Script using too much CPU
**Solution:**
- The SSE stream is event-driven and should be efficient
- If processing many aircraft, consider batching alerts
- Add a small delay between processing aircraft in high-traffic areas

### Missing Aircraft Categories

**Problem:** Want to add new category of rare aircraft
**Solution:**
- Add new category to `CATEGORY_EMOJI` dictionary
- Add aircraft types with the new category to `RARE_AIRCRAFT`
- Example for adding "experimental" category:

```python
CATEGORY_EMOJI["experimental"] = "\U0001F52C"  # Microscope

RARE_AIRCRAFT["X15"] = {
    "name": "North American X-15",
    "category": "experimental",
    "notes": "Hypersonic research aircraft"
}
```

## Related Recipes

- [VIP Aircraft Tracker](/docs/vip-tracker) - Track celebrity and VIP aircraft by hex code
- [Military Aircraft Spotter](/docs/military-spotter) - Built-in military detection
- [Discord Alert Bot](/docs/discord-alert-bot) - Send notifications to Discord
- [Telegram Bot](/docs/telegram-bot) - Send alerts to Telegram
