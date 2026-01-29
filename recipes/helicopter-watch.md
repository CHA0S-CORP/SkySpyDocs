---
title: Helicopter Watch
excerpt: Dedicated monitoring for helicopter traffic
hidden: false
recipe:
  color: '#059669'
  icon: "🚁"
difficulty: beginner
tags: [specialty, helicopter, monitoring, filtering]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Push notifications configured (optional, for mobile alerts)
- Basic understanding of SkySpy's alert rules API

## What You'll Build

A comprehensive helicopter monitoring system that detects and classifies helicopters in your area. This recipe identifies helicopters through:

- **Aircraft type codes** - Common helicopter type designators (R22, R44, EC35, etc.)
- **ADS-B category codes** - A7 (rotorcraft) and B7 (rotorcraft)
- **Classification function** - Automatically categorize by mission type (Medical, Police, etc.)

### Helicopter Type Codes Reference

| Category | Type Codes |
|----------|------------|
| Robinson | R22, R44, R66 |
| Bell | B206, B407, B412, B429, B505 |
| Airbus/Eurocopter | EC35, EC45, EC55, EC30, AS50, AS55, AS32 |
| Sikorsky | S76, S92, S70, S64 |
| Military | UH60 (Black Hawk), AH64 (Apache), CH47 (Chinook), UH1 (Huey) |
| MD Helicopters | MD52, MD60, MD90 |
| Leonardo/AgustaWestland | A109, A139, AW10, AW13, AW18 |

```shell Shell
SKYSPY_URL="${SKYSPY_URL:-http://localhost:5000}"

HELI_TYPES="R22,R44,R66,B206,B407,B412,B429,B505,EC35,EC45,EC55,EC30,AS50,AS55,AS32,S76,S92,S70,S64,UH60,AH64,CH47,UH1,MD52,MD60,MD90,A109,A139,AW10,AW13,AW18"

classify_helicopter() {
  local callsign="$1"
  local type_code="$2"
  local registration="$3"

  case "$callsign" in
    *MEDEVAC*|*LIFEFLIGHT*|*AIREVAC*|*MERCY*|*REACH*|*LIFE*|*CARE*|*MED*)
      echo "Medical"
      ;;
    *POLICE*|*NPAS*|*EAGLE*|*LAPD*|*NYPD*|*SHERIFF*)
      echo "Police"
      ;;
    *FIRE*|*RESCUE*|*SAR*|*GUARD*|*USCG*)
      echo "Fire/SAR"
      ;;
    *NEWS*|*COPTER*|*SKY*|*CHOPPER*)
      echo "News"
      ;;
    *)
      case "$type_code" in
        UH60|AH64|CH47|UH1)
          echo "Military"
          ;;
        *)
          if [[ "$registration" =~ ^N[0-9]+[A-Z]{2}$ ]]; then
            echo "Tour"
          else
            echo "Civil"
          fi
          ;;
      esac
      ;;
  esac
}

curl -X POST "${SKYSPY_URL}/api/alerts/rules" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Helicopter Watch",
    "enabled": true,
    "priority": "medium",
    "conditions": {
      "operator": "OR",
      "conditions": [
        { "field": "category", "operator": "in", "value": ["A7", "B7"] },
        { "field": "type", "operator": "in", "value": ["R22","R44","R66","B206","B407","B412","B429","B505","EC35","EC45","EC55","EC30","AS50","AS55","AS32","S76","S92","S70","S64","UH60","AH64","CH47","UH1","MD52","MD60","MD90","A109","A139","AW10","AW13","AW18"] }
      ]
    },
    "notification_enabled": true
  }'
```

```go Go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"net/http"
	"os"
	"strings"
)

var helicopterTypes = []string{
	"R22", "R44", "R66",
	"B206", "B407", "B412", "B429", "B505",
	"EC35", "EC45", "EC55", "EC30", "AS50", "AS55", "AS32",
	"S76", "S92", "S70", "S64",
	"UH60", "AH64", "CH47", "UH1",
	"MD52", "MD60", "MD90",
	"A109", "A139", "AW10", "AW13", "AW18",
}

type HeliClass string

const (
	ClassMedical  HeliClass = "Medical"
	ClassPolice   HeliClass = "Police"
	ClassFireSAR  HeliClass = "Fire/SAR"
	ClassNews     HeliClass = "News"
	ClassMilitary HeliClass = "Military"
	ClassTour     HeliClass = "Tour"
	ClassCivil    HeliClass = "Civil"
)

func classifyHelicopter(callsign, typeCode, registration string) HeliClass {
	cs := strings.ToUpper(callsign)

	medicalPatterns := []string{"MEDEVAC", "LIFEFLIGHT", "AIREVAC", "MERCY", "REACH", "LIFE", "CARE", "MED"}
	for _, p := range medicalPatterns {
		if strings.Contains(cs, p) {
			return ClassMedical
		}
	}

	policePatterns := []string{"POLICE", "NPAS", "EAGLE", "LAPD", "NYPD", "SHERIFF"}
	for _, p := range policePatterns {
		if strings.Contains(cs, p) {
			return ClassPolice
		}
	}

	fireSARPatterns := []string{"FIRE", "RESCUE", "SAR", "GUARD", "USCG"}
	for _, p := range fireSARPatterns {
		if strings.Contains(cs, p) {
			return ClassFireSAR
		}
	}

	newsPatterns := []string{"NEWS", "COPTER", "SKY", "CHOPPER"}
	for _, p := range newsPatterns {
		if strings.Contains(cs, p) {
			return ClassNews
		}
	}

	militaryTypes := map[string]bool{"UH60": true, "AH64": true, "CH47": true, "UH1": true}
	if militaryTypes[typeCode] {
		return ClassMilitary
	}

	return ClassCivil
}

func getClassIcon(class HeliClass) string {
	icons := map[HeliClass]string{
		ClassMedical:  "\U0001F3E5",
		ClassPolice:   "\U0001F694",
		ClassFireSAR:  "\U0001F692",
		ClassNews:     "\U0001F4FA",
		ClassMilitary: "\U0001F396\uFE0F",
		ClassTour:     "\U0001F39F\uFE0F",
		ClassCivil:    "\U0001F681",
	}
	return icons[class]
}

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	rule := map[string]interface{}{
		"name":     "Helicopter Watch",
		"enabled":  true,
		"priority": "medium",
		"conditions": map[string]interface{}{
			"operator": "OR",
			"conditions": []map[string]interface{}{
				{"field": "category", "operator": "in", "value": []string{"A7", "B7"}},
				{"field": "type", "operator": "in", "value": helicopterTypes},
			},
		},
		"notification_enabled": true,
	}

	body, _ := json.Marshal(rule)
	resp, err := http.Post(skyspyURL+"/api/alerts/rules", "application/json", bytes.NewReader(body))
	if err != nil {
		fmt.Printf("Error creating rule: %v\n", err)
		os.Exit(1)
	}
	defer resp.Body.Close()

	if resp.StatusCode == 201 {
		fmt.Println("Helicopter Watch rule created successfully!")
	} else {
		fmt.Printf("Failed to create rule: %s\n", resp.Status)
	}
}
```

```python Python
import os
import re
import requests

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

HELICOPTER_TYPES = [
    "R22", "R44", "R66",
    "B206", "B407", "B412", "B429", "B505",
    "EC35", "EC45", "EC55", "EC30", "AS50", "AS55", "AS32",
    "S76", "S92", "S70", "S64",
    "UH60", "AH64", "CH47", "UH1",
    "MD52", "MD60", "MD90",
    "A109", "A139", "AW10", "AW13", "AW18",
]

CATEGORY_CODES = ["A7", "B7"]

class HeliClassification:
    MEDICAL = ("Medical", "\U0001F3E5")
    POLICE = ("Police", "\U0001F694")
    FIRE_SAR = ("Fire/SAR", "\U0001F692")
    NEWS = ("News", "\U0001F4FA")
    MILITARY = ("Military", "\U0001F396\uFE0F")
    TOUR = ("Tour", "\U0001F39F\uFE0F")
    CIVIL = ("Civil", "\U0001F681")


def classify_helicopter(callsign: str, type_code: str, registration: str = "") -> tuple:
    """Classify a helicopter based on callsign, type code, and registration."""
    cs = (callsign or "").upper()

    medical_patterns = ["MEDEVAC", "LIFEFLIGHT", "AIREVAC", "MERCY", "REACH", "LIFE", "CARE", "MED"]
    if any(p in cs for p in medical_patterns):
        return HeliClassification.MEDICAL

    police_patterns = ["POLICE", "NPAS", "EAGLE", "LAPD", "NYPD", "SHERIFF"]
    if any(p in cs for p in police_patterns):
        return HeliClassification.POLICE

    fire_sar_patterns = ["FIRE", "RESCUE", "SAR", "GUARD", "USCG"]
    if any(p in cs for p in fire_sar_patterns):
        return HeliClassification.FIRE_SAR

    news_patterns = ["NEWS", "COPTER", "SKY", "CHOPPER"]
    if any(p in cs for p in news_patterns):
        return HeliClassification.NEWS

    military_types = {"UH60", "AH64", "CH47", "UH1"}
    if type_code in military_types:
        return HeliClassification.MILITARY

    if registration and re.match(r"^N\d+[A-Z]{2}$", registration):
        return HeliClassification.TOUR

    return HeliClassification.CIVIL


rule = {
    "name": "Helicopter Watch",
    "enabled": True,
    "priority": "medium",
    "conditions": {
        "operator": "OR",
        "conditions": [
            {"field": "category", "operator": "in", "value": CATEGORY_CODES},
            {"field": "type", "operator": "in", "value": HELICOPTER_TYPES}
        ]
    },
    "notification_enabled": True
}

response = requests.post(f"{SKYSPY_URL}/api/alerts/rules", json=rule)

if response.status_code == 201:
    print("Helicopter Watch rule created successfully!")
    print(f"Rule ID: {response.json().get('id')}")
else:
    print(f"Failed to create rule: {response.status_code}")
    print(response.text)
```

```javascript JavaScript
const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';

const HELICOPTER_TYPES = [
  'R22', 'R44', 'R66',
  'B206', 'B407', 'B412', 'B429', 'B505',
  'EC35', 'EC45', 'EC55', 'EC30', 'AS50', 'AS55', 'AS32',
  'S76', 'S92', 'S70', 'S64',
  'UH60', 'AH64', 'CH47', 'UH1',
  'MD52', 'MD60', 'MD90',
  'A109', 'A139', 'AW10', 'AW13', 'AW18'
];

const CATEGORY_CODES = ['A7', 'B7'];

const HeliClass = {
  MEDICAL: { name: 'Medical', icon: '\u{1F3E5}' },
  POLICE: { name: 'Police', icon: '\u{1F694}' },
  FIRE_SAR: { name: 'Fire/SAR', icon: '\u{1F692}' },
  NEWS: { name: 'News', icon: '\u{1F4FA}' },
  MILITARY: { name: 'Military', icon: '\u{1F396}' },
  TOUR: { name: 'Tour', icon: '\u{1F39F}' },
  CIVIL: { name: 'Civil', icon: '\u{1F681}' }
};

function classifyHelicopter(callsign, typeCode, registration = '') {
  const cs = (callsign || '').toUpperCase();

  const medicalPatterns = ['MEDEVAC', 'LIFEFLIGHT', 'AIREVAC', 'MERCY', 'REACH', 'LIFE', 'CARE', 'MED'];
  if (medicalPatterns.some(p => cs.includes(p))) return HeliClass.MEDICAL;

  const policePatterns = ['POLICE', 'NPAS', 'EAGLE', 'LAPD', 'NYPD', 'SHERIFF'];
  if (policePatterns.some(p => cs.includes(p))) return HeliClass.POLICE;

  const fireSARPatterns = ['FIRE', 'RESCUE', 'SAR', 'GUARD', 'USCG'];
  if (fireSARPatterns.some(p => cs.includes(p))) return HeliClass.FIRE_SAR;

  const newsPatterns = ['NEWS', 'COPTER', 'SKY', 'CHOPPER'];
  if (newsPatterns.some(p => cs.includes(p))) return HeliClass.NEWS;

  const militaryTypes = ['UH60', 'AH64', 'CH47', 'UH1'];
  if (militaryTypes.includes(typeCode)) return HeliClass.MILITARY;

  if (registration && /^N\d+[A-Z]{2}$/.test(registration)) return HeliClass.TOUR;

  return HeliClass.CIVIL;
}

fetch(`${SKYSPY_URL}/api/alerts/rules`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    name: 'Helicopter Watch',
    enabled: true,
    priority: 'medium',
    conditions: {
      operator: 'OR',
      conditions: [
        { field: 'category', operator: 'in', value: CATEGORY_CODES },
        { field: 'type', operator: 'in', value: HELICOPTER_TYPES }
      ]
    },
    notification_enabled: true
  })
})
.then(res => res.json())
.then(data => {
  console.log('Helicopter Watch rule created successfully!');
  console.log('Rule ID:', data.id);
})
.catch(err => console.error('Failed to create rule:', err));
```

```json Response Example
{
  "id": 2,
  "name": "Helicopter Watch",
  "enabled": true,
  "priority": "medium",
  "conditions": {
    "operator": "OR",
    "conditions": [
      {"field": "category", "operator": "in", "value": ["A7", "B7"]},
      {"field": "type", "operator": "in", "value": ["R22", "R44", "R66", "B206", "..."]}
    ]
  },
  "notification_enabled": true,
  "created_at": "2024-01-15T12:00:00Z"
}
```

## Create Helicopter Alert Rule

<!-- shell@43-55 -->
<!-- go@83-103 -->
<!-- python@53-72 -->
<!-- javascript@46-65 -->

Create an alert rule that triggers whenever a helicopter is detected. The rule uses both ADS-B category codes (A7/B7 for rotorcraft) and specific helicopter type codes for comprehensive detection.

## Helicopter Classification System

The classification function analyzes callsigns, type codes, and registration patterns to determine the helicopter's mission type:

| Classification | Icon | Detection Method |
|----------------|------|------------------|
| Medical | 🏥 | Callsigns containing MEDEVAC, LIFEFLIGHT, AIREVAC, MERCY, REACH, LIFE, CARE, MED |
| Police | 🚔 | Callsigns containing POLICE, NPAS, EAGLE, LAPD, NYPD, SHERIFF |
| Fire/SAR | 🚒 | Callsigns containing FIRE, RESCUE, SAR, GUARD, USCG |
| News | 📺 | Callsigns containing NEWS, COPTER, SKY, CHOPPER |
| Military | 🎖️ | Type codes UH60, AH64, CH47, UH1 |
| Tour | 🎟️ | Registration pattern matching tour operators |
| Civil | 🚁 | Default classification for unmatched helicopters |

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Priority Levels

| Priority | Use Case |
|----------|----------|
| `low` | General helicopter tracking |
| `medium` | Standard monitoring (recommended) |
| `high` | Emergency services helicopters |
| `critical` | Specific aircraft of interest |

### Category Codes

| Code | Description |
|------|-------------|
| `A7` | Rotorcraft (primary helicopter category) |
| `B7` | Rotorcraft (alternate category designation) |

### Additional Conditions

Combine with other conditions for more specific alerts:

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {
        "operator": "OR",
        "conditions": [
          {"field": "category", "operator": "in", "value": ["A7", "B7"]},
          {"field": "type", "operator": "in", "value": ["R22", "R44", "EC35"]}
        ]
      },
      {"field": "alt", "operator": "lt", "value": 3000}
    ]
  }
}
```

This alerts only on low-flying helicopters (below 3000 feet).

### Filter by Classification

To alert only on medical helicopters:

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "category", "operator": "in", "value": ["A7", "B7"]},
      {"field": "callsign", "operator": "contains", "value": "LIFE"}
    ]
  }
}
```

## Testing & Verification

1. **Create the rule** using any of the code examples above
2. **Verify in SkySpy UI** - Go to Alerts -> Rules to see your new rule
3. **Test classification** - Use the classification function with sample data:

```python
# Test the classification function
print(classify_helicopter("LIFEFLIGHT1", "EC45", "N911LF"))
# Output: ('Medical', '\U0001F3E5')

print(classify_helicopter("LAPD10", "AS50", "N664PD"))
# Output: ('Police', '\U0001F694')

print(classify_helicopter("", "UH60", ""))
# Output: ('Military', '\U0001F396\uFE0F')
```

4. **Wait for helicopters** - Alerts appear in the Alerts tab when detected
5. **Check notifications** - If push notifications are configured, you'll receive mobile alerts

### List Existing Rules

```shell
curl http://localhost:5000/api/alerts/rules
```

### Delete a Rule

```shell
curl -X DELETE http://localhost:5000/api/alerts/rules/2
```

## Troubleshooting

### Rule Not Triggering

**Problem:** Helicopters appear but no alerts are generated
**Solution:**
- Verify the rule is enabled: check `enabled: true` in the API response
- Check if the helicopter's type code or category matches the rule conditions
- Some helicopters may not broadcast type codes - add more type codes to the list
- Verify the aircraft is transmitting category A7 or B7

### Missing Helicopter Types

**Problem:** Some helicopters are not being detected
**Solution:** Add additional type codes to the list. Common additions:

```json
{
  "field": "type",
  "operator": "in",
  "value": ["R22", "R44", "R66", "B206", "B407", "EC35", "EC45", "S76", "UH60", "YOUR_NEW_TYPE"]
}
```

### Classification Not Working

**Problem:** Helicopters are all classified as "Civil"
**Solution:**
- Ensure callsign data is being received (some aircraft don't broadcast callsigns)
- Add more callsign patterns to the classification function for your area
- Check if local operators use different callsign prefixes

### Too Many Alerts

**Problem:** Getting alerts for every helicopter is overwhelming
**Solution:** Add filters to reduce alerts:

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "category", "operator": "in", "value": ["A7", "B7"]},
      {"field": "distance", "operator": "lt", "value": 10}
    ]
  },
  "cooldown": 1800
}
```

This limits alerts to helicopters within 10nm and adds a 30-minute cooldown per aircraft.

### False Positives

**Problem:** Non-helicopter aircraft triggering alerts
**Solution:**
- Rely more heavily on category codes (A7/B7) which are specifically for rotorcraft
- Remove any type codes that might overlap with fixed-wing aircraft
- Add negative conditions to exclude known false positives

## Related Recipes

- [Military Aircraft Spotter](/docs/military-spotter) - Monitor military aircraft including military helicopters
- [Emergency Alert Monitor](/docs/emergency-monitor) - Track emergency squawks (1200, 7500, 7600, 7700)
- [Track Specific Aircraft](/docs/track-aircraft) - Monitor specific tail numbers
- [Discord Alert Bot](/docs/discord-alert-bot) - Send helicopter alerts to Discord
