---
title: Drone Detection
excerpt: Low and slow target alerts for potential drones
hidden: false
recipe:
  color: '#DC2626'
  icon: "🎯"
difficulty: intermediate
tags: [specialty, drone, security, monitoring]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Push notifications configured (optional, for mobile alerts)
- Basic understanding of SkySpy's alert rules API
- Familiarity with ADS-B data characteristics

## What You'll Build

A comprehensive drone detection system that identifies potential drone activity in your area. This recipe detects drones through:

- **Low altitude filtering** - Drones typically operate below 400 feet AGL
- **Slow speed detection** - Most drones cruise between 0-60 knots
- **Small category identification** - ADS-B category codes for light/small aircraft
- **Remote ID detection** - FAA Remote ID broadcasts when available
- **False positive filtering** - Exclude known non-drone aircraft patterns

### Drone Detection Criteria Reference

| Criteria | Typical Value | Notes |
|----------|---------------|-------|
| Altitude | < 500 feet | FAA limit is 400ft AGL for recreational |
| Speed | < 60 knots | Most consumer drones max ~40 knots |
| Category | A1, B1, B2 | Light/small aircraft categories |
| Vertical Rate | Variable | Drones can hover (0) or climb/descend rapidly |

### Remote ID Message Types

| Field | Description |
|-------|-------------|
| `remote_id` | FAA Remote ID broadcast identifier |
| `operator_id` | Registered operator identification |
| `ua_type` | Unmanned aircraft type (1=Helicopter, 2=Multirotor, etc.) |

```shell Shell
SKYSPY_URL="${SKYSPY_URL:-http://localhost:5000}"

MAX_ALTITUDE=500
MAX_SPEED=60
SMALL_CATEGORIES="A1,B1,B2"

is_likely_drone() {
  local altitude="$1"
  local speed="$2"
  local category="$3"
  local callsign="$4"
  local remote_id="$5"

  if [[ -n "$remote_id" ]]; then
    echo "confirmed"
    return 0
  fi

  if [[ "$callsign" =~ ^(N[0-9]+|UAL|AAL|DAL|SWA|JBU) ]]; then
    echo "excluded"
    return 1
  fi

  if (( altitude < MAX_ALTITUDE && speed < MAX_SPEED )); then
    case "$category" in
      A1|B1|B2)
        echo "likely"
        return 0
        ;;
    esac
  fi

  echo "unlikely"
  return 1
}

assess_threat_level() {
  local altitude="$1"
  local speed="$2"
  local distance="$3"
  local is_hovering="$4"

  if (( distance < 1 && altitude < 200 )); then
    echo "high"
  elif (( distance < 3 && altitude < 400 )); then
    echo "medium"
  else
    echo "low"
  fi
}

curl -X POST "${SKYSPY_URL}/api/alerts/rules" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Drone Detection - Low & Slow",
    "enabled": true,
    "priority": "high",
    "conditions": {
      "operator": "AND",
      "conditions": [
        { "field": "alt", "operator": "lt", "value": 500 },
        { "field": "gs", "operator": "lt", "value": 60 },
        {
          "operator": "OR",
          "conditions": [
            { "field": "category", "operator": "in", "value": ["A1", "B1", "B2"] },
            { "field": "remote_id", "operator": "exists", "value": true }
          ]
        }
      ]
    },
    "notification_enabled": true,
    "cooldown": 300
  }'

curl -X POST "${SKYSPY_URL}/api/alerts/rules" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Drone Detection - Remote ID",
    "enabled": true,
    "priority": "medium",
    "conditions": {
      "operator": "OR",
      "conditions": [
        { "field": "remote_id", "operator": "exists", "value": true },
        { "field": "ua_type", "operator": "in", "value": [1, 2, 3, 4, 5] }
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

const (
	MaxAltitude = 500
	MaxSpeed    = 60
)

var smallCategories = []string{"A1", "B1", "B2"}

type DroneClassification string

const (
	DroneConfirmed DroneClassification = "confirmed"
	DroneLikely    DroneClassification = "likely"
	DronePossible  DroneClassification = "possible"
	DroneUnlikely  DroneClassification = "unlikely"
	DroneExcluded  DroneClassification = "excluded"
)

type ThreatLevel string

const (
	ThreatHigh   ThreatLevel = "high"
	ThreatMedium ThreatLevel = "medium"
	ThreatLow    ThreatLevel = "low"
)

type Aircraft struct {
	Hex        string  `json:"hex"`
	Callsign   string  `json:"callsign"`
	Altitude   int     `json:"alt"`
	Speed      float64 `json:"gs"`
	Category   string  `json:"category"`
	RemoteID   string  `json:"remote_id"`
	OperatorID string  `json:"operator_id"`
	UAType     int     `json:"ua_type"`
	Distance   float64 `json:"distance"`
	VertRate   int     `json:"vert_rate"`
}

func classifyDrone(ac Aircraft) DroneClassification {
	if ac.RemoteID != "" || ac.UAType > 0 {
		return DroneConfirmed
	}

	excludedPrefixes := []string{"UAL", "AAL", "DAL", "SWA", "JBU", "FFT", "SKW"}
	cs := strings.ToUpper(ac.Callsign)
	for _, prefix := range excludedPrefixes {
		if strings.HasPrefix(cs, prefix) {
			return DroneExcluded
		}
	}

	if len(ac.Callsign) > 0 && ac.Callsign[0] == 'N' {
		if len(ac.Callsign) >= 4 && ac.Callsign[1] >= '0' && ac.Callsign[1] <= '9' {
			return DroneExcluded
		}
	}

	if ac.Altitude < MaxAltitude && ac.Speed < MaxSpeed {
		for _, cat := range smallCategories {
			if ac.Category == cat {
				return DroneLikely
			}
		}
		if ac.VertRate == 0 && ac.Speed < 5 {
			return DronePossible
		}
	}

	return DroneUnlikely
}

func assessThreat(ac Aircraft) ThreatLevel {
	if ac.Distance < 1 && ac.Altitude < 200 {
		return ThreatHigh
	}
	if ac.Distance < 3 && ac.Altitude < 400 {
		return ThreatMedium
	}
	return ThreatLow
}

func getClassificationIcon(class DroneClassification) string {
	icons := map[DroneClassification]string{
		DroneConfirmed: "\U0001F6A8",
		DroneLikely:    "\U0001F3AF",
		DronePossible:  "\U00002753",
		DroneUnlikely:  "\U00002714\uFE0F",
		DroneExcluded:  "\U0000274C",
	}
	return icons[class]
}

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	lowSlowRule := map[string]interface{}{
		"name":     "Drone Detection - Low & Slow",
		"enabled":  true,
		"priority": "high",
		"conditions": map[string]interface{}{
			"operator": "AND",
			"conditions": []map[string]interface{}{
				{"field": "alt", "operator": "lt", "value": MaxAltitude},
				{"field": "gs", "operator": "lt", "value": MaxSpeed},
				{
					"operator": "OR",
					"conditions": []map[string]interface{}{
						{"field": "category", "operator": "in", "value": smallCategories},
						{"field": "remote_id", "operator": "exists", "value": true},
					},
				},
			},
		},
		"notification_enabled": true,
		"cooldown":             300,
	}

	body, _ := json.Marshal(lowSlowRule)
	resp, err := http.Post(skyspyURL+"/api/alerts/rules", "application/json", bytes.NewReader(body))
	if err != nil {
		fmt.Printf("Error creating rule: %v\n", err)
		os.Exit(1)
	}
	defer resp.Body.Close()

	if resp.StatusCode == 201 {
		fmt.Println("Drone Detection (Low & Slow) rule created successfully!")
	} else {
		fmt.Printf("Failed to create rule: %s\n", resp.Status)
	}

	remoteIDRule := map[string]interface{}{
		"name":     "Drone Detection - Remote ID",
		"enabled":  true,
		"priority": "medium",
		"conditions": map[string]interface{}{
			"operator": "OR",
			"conditions": []map[string]interface{}{
				{"field": "remote_id", "operator": "exists", "value": true},
				{"field": "ua_type", "operator": "in", "value": []int{1, 2, 3, 4, 5}},
			},
		},
		"notification_enabled": true,
	}

	body, _ = json.Marshal(remoteIDRule)
	resp, err = http.Post(skyspyURL+"/api/alerts/rules", "application/json", bytes.NewReader(body))
	if err != nil {
		fmt.Printf("Error creating Remote ID rule: %v\n", err)
		os.Exit(1)
	}
	defer resp.Body.Close()

	if resp.StatusCode == 201 {
		fmt.Println("Drone Detection (Remote ID) rule created successfully!")
	} else {
		fmt.Printf("Failed to create Remote ID rule: %s\n", resp.Status)
	}
}
```

```python Python
import os
import re
import requests
from enum import Enum
from dataclasses import dataclass
from typing import Optional

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")

MAX_ALTITUDE = 500
MAX_SPEED = 60
SMALL_CATEGORIES = ["A1", "B1", "B2"]


class DroneClassification(Enum):
    CONFIRMED = ("confirmed", "\U0001F6A8")
    LIKELY = ("likely", "\U0001F3AF")
    POSSIBLE = ("possible", "\U00002753")
    UNLIKELY = ("unlikely", "\U00002714\uFE0F")
    EXCLUDED = ("excluded", "\U0000274C")


class ThreatLevel(Enum):
    HIGH = "high"
    MEDIUM = "medium"
    LOW = "low"


@dataclass
class Aircraft:
    hex: str
    callsign: str = ""
    altitude: int = 0
    speed: float = 0.0
    category: str = ""
    remote_id: str = ""
    operator_id: str = ""
    ua_type: int = 0
    distance: float = 0.0
    vert_rate: int = 0


def classify_drone(ac: Aircraft) -> DroneClassification:
    """Classify whether an aircraft is likely a drone."""
    if ac.remote_id or ac.ua_type > 0:
        return DroneClassification.CONFIRMED

    excluded_prefixes = ["UAL", "AAL", "DAL", "SWA", "JBU", "FFT", "SKW"]
    cs = (ac.callsign or "").upper()
    if any(cs.startswith(prefix) for prefix in excluded_prefixes):
        return DroneClassification.EXCLUDED

    if re.match(r"^N\d", ac.callsign):
        return DroneClassification.EXCLUDED

    if ac.altitude < MAX_ALTITUDE and ac.speed < MAX_SPEED:
        if ac.category in SMALL_CATEGORIES:
            return DroneClassification.LIKELY
        if ac.vert_rate == 0 and ac.speed < 5:
            return DroneClassification.POSSIBLE

    return DroneClassification.UNLIKELY


def assess_threat(ac: Aircraft) -> ThreatLevel:
    """Assess threat level based on proximity and altitude."""
    if ac.distance < 1 and ac.altitude < 200:
        return ThreatLevel.HIGH
    if ac.distance < 3 and ac.altitude < 400:
        return ThreatLevel.MEDIUM
    return ThreatLevel.LOW


low_slow_rule = {
    "name": "Drone Detection - Low & Slow",
    "enabled": True,
    "priority": "high",
    "conditions": {
        "operator": "AND",
        "conditions": [
            {"field": "alt", "operator": "lt", "value": MAX_ALTITUDE},
            {"field": "gs", "operator": "lt", "value": MAX_SPEED},
            {
                "operator": "OR",
                "conditions": [
                    {"field": "category", "operator": "in", "value": SMALL_CATEGORIES},
                    {"field": "remote_id", "operator": "exists", "value": True}
                ]
            }
        ]
    },
    "notification_enabled": True,
    "cooldown": 300
}

response = requests.post(f"{SKYSPY_URL}/api/alerts/rules", json=low_slow_rule)

if response.status_code == 201:
    print("Drone Detection (Low & Slow) rule created successfully!")
    print(f"Rule ID: {response.json().get('id')}")
else:
    print(f"Failed to create rule: {response.status_code}")
    print(response.text)

remote_id_rule = {
    "name": "Drone Detection - Remote ID",
    "enabled": True,
    "priority": "medium",
    "conditions": {
        "operator": "OR",
        "conditions": [
            {"field": "remote_id", "operator": "exists", "value": True},
            {"field": "ua_type", "operator": "in", "value": [1, 2, 3, 4, 5]}
        ]
    },
    "notification_enabled": True
}

response = requests.post(f"{SKYSPY_URL}/api/alerts/rules", json=remote_id_rule)

if response.status_code == 201:
    print("Drone Detection (Remote ID) rule created successfully!")
    print(f"Rule ID: {response.json().get('id')}")
else:
    print(f"Failed to create rule: {response.status_code}")
    print(response.text)
```

```javascript JavaScript
const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';

const MAX_ALTITUDE = 500;
const MAX_SPEED = 60;
const SMALL_CATEGORIES = ['A1', 'B1', 'B2'];

const DroneClassification = {
  CONFIRMED: { name: 'confirmed', icon: '\u{1F6A8}' },
  LIKELY: { name: 'likely', icon: '\u{1F3AF}' },
  POSSIBLE: { name: 'possible', icon: '\u{2753}' },
  UNLIKELY: { name: 'unlikely', icon: '\u{2714}' },
  EXCLUDED: { name: 'excluded', icon: '\u{274C}' }
};

const ThreatLevel = {
  HIGH: 'high',
  MEDIUM: 'medium',
  LOW: 'low'
};

function classifyDrone(aircraft) {
  const { callsign, altitude, speed, category, remote_id, ua_type, vert_rate } = aircraft;

  if (remote_id || ua_type > 0) {
    return DroneClassification.CONFIRMED;
  }

  const excludedPrefixes = ['UAL', 'AAL', 'DAL', 'SWA', 'JBU', 'FFT', 'SKW'];
  const cs = (callsign || '').toUpperCase();
  if (excludedPrefixes.some(prefix => cs.startsWith(prefix))) {
    return DroneClassification.EXCLUDED;
  }

  if (/^N\d/.test(callsign)) {
    return DroneClassification.EXCLUDED;
  }

  if (altitude < MAX_ALTITUDE && speed < MAX_SPEED) {
    if (SMALL_CATEGORIES.includes(category)) {
      return DroneClassification.LIKELY;
    }
    if (vert_rate === 0 && speed < 5) {
      return DroneClassification.POSSIBLE;
    }
  }

  return DroneClassification.UNLIKELY;
}

function assessThreat(aircraft) {
  const { altitude, distance } = aircraft;

  if (distance < 1 && altitude < 200) {
    return ThreatLevel.HIGH;
  }
  if (distance < 3 && altitude < 400) {
    return ThreatLevel.MEDIUM;
  }
  return ThreatLevel.LOW;
}

async function createRules() {
  const lowSlowRule = {
    name: 'Drone Detection - Low & Slow',
    enabled: true,
    priority: 'high',
    conditions: {
      operator: 'AND',
      conditions: [
        { field: 'alt', operator: 'lt', value: MAX_ALTITUDE },
        { field: 'gs', operator: 'lt', value: MAX_SPEED },
        {
          operator: 'OR',
          conditions: [
            { field: 'category', operator: 'in', value: SMALL_CATEGORIES },
            { field: 'remote_id', operator: 'exists', value: true }
          ]
        }
      ]
    },
    notification_enabled: true,
    cooldown: 300
  };

  try {
    let res = await fetch(`${SKYSPY_URL}/api/alerts/rules`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(lowSlowRule)
    });
    let data = await res.json();
    console.log('Drone Detection (Low & Slow) rule created successfully!');
    console.log('Rule ID:', data.id);
  } catch (err) {
    console.error('Failed to create Low & Slow rule:', err);
  }

  const remoteIDRule = {
    name: 'Drone Detection - Remote ID',
    enabled: true,
    priority: 'medium',
    conditions: {
      operator: 'OR',
      conditions: [
        { field: 'remote_id', operator: 'exists', value: true },
        { field: 'ua_type', operator: 'in', value: [1, 2, 3, 4, 5] }
      ]
    },
    notification_enabled: true
  };

  try {
    let res = await fetch(`${SKYSPY_URL}/api/alerts/rules`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(remoteIDRule)
    });
    let data = await res.json();
    console.log('Drone Detection (Remote ID) rule created successfully!');
    console.log('Rule ID:', data.id);
  } catch (err) {
    console.error('Failed to create Remote ID rule:', err);
  }
}

createRules();
```

```json Response Example
{
  "id": 5,
  "name": "Drone Detection - Low & Slow",
  "enabled": true,
  "priority": "high",
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "alt", "operator": "lt", "value": 500},
      {"field": "gs", "operator": "lt", "value": 60},
      {
        "operator": "OR",
        "conditions": [
          {"field": "category", "operator": "in", "value": ["A1", "B1", "B2"]},
          {"field": "remote_id", "operator": "exists", "value": true}
        ]
      }
    ]
  },
  "notification_enabled": true,
  "cooldown": 300,
  "created_at": "2024-01-15T12:00:00Z"
}
```

## Create Drone Detection Rules

<!-- shell@54-94 -->
<!-- go@88-142 -->
<!-- python@57-95 -->
<!-- javascript@54-98 -->

Create alert rules that trigger when potential drone activity is detected. Two complementary rules are provided:

1. **Low & Slow Rule** - Combines altitude, speed, and category filters to identify typical drone flight profiles
2. **Remote ID Rule** - Detects aircraft broadcasting FAA Remote ID signals (required for most drones since 2023)

## Drone Classification System

The classification function analyzes multiple data points to determine the likelihood of an aircraft being a drone:

| Classification | Icon | Detection Method |
|----------------|------|------------------|
| Confirmed | 🚨 | Remote ID broadcast present or UA type field populated |
| Likely | 🎯 | Low altitude + slow speed + small aircraft category |
| Possible | ❓ | Low altitude + very slow/hovering + no category match |
| Unlikely | ✔️ | Does not match drone profile criteria |
| Excluded | ❌ | Matches known airline/GA aircraft patterns |

## Threat Assessment

The threat assessment function evaluates proximity and altitude to prioritize alerts:

| Threat Level | Criteria | Action |
|--------------|----------|--------|
| High | Distance < 1nm AND Altitude < 200ft | Immediate notification |
| Medium | Distance < 3nm AND Altitude < 400ft | Standard notification |
| Low | All other detections | Log only (optional notification) |

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `MAX_ALTITUDE` | Maximum altitude threshold (feet) | `500` |
| `MAX_SPEED` | Maximum speed threshold (knots) | `60` |

### Adjusting Detection Sensitivity

For **higher sensitivity** (more detections, more false positives):

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "alt", "operator": "lt", "value": 1000},
      {"field": "gs", "operator": "lt", "value": 100}
    ]
  }
}
```

For **lower sensitivity** (fewer detections, fewer false positives):

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "alt", "operator": "lt", "value": 300},
      {"field": "gs", "operator": "lt", "value": 40},
      {"field": "category", "operator": "in", "value": ["B1", "B2"]}
    ]
  }
}
```

### Remote ID UA Types

| UA Type | Description |
|---------|-------------|
| 1 | Helicopter (rotorcraft) |
| 2 | Multirotor (quadcopter, etc.) |
| 3 | Fixed-wing |
| 4 | Hybrid lift |
| 5 | Other |

### Geofencing for Sensitive Areas

Add location-based filtering to focus on specific areas:

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "alt", "operator": "lt", "value": 500},
      {"field": "gs", "operator": "lt", "value": 60},
      {"field": "distance", "operator": "lt", "value": 5},
      {"field": "lat", "operator": "between", "value": [40.7, 40.8]},
      {"field": "lon", "operator": "between", "value": [-74.1, -74.0]}
    ]
  }
}
```

## Testing & Verification

1. **Create the rules** using any of the code examples above
2. **Verify in SkySpy UI** - Go to Alerts -> Rules to see your new rules
3. **Test classification** - Use the classification function with sample data:

```python
# Test the classification function
ac = Aircraft(hex="abc123", altitude=200, speed=25, category="B2")
print(classify_drone(ac))
# Output: DroneClassification.LIKELY

ac = Aircraft(hex="def456", remote_id="FA-1234567890")
print(classify_drone(ac))
# Output: DroneClassification.CONFIRMED

ac = Aircraft(hex="ghi789", callsign="UAL123", altitude=300, speed=50)
print(classify_drone(ac))
# Output: DroneClassification.EXCLUDED
```

4. **Test threat assessment**:

```python
ac = Aircraft(hex="abc123", altitude=150, distance=0.5)
print(assess_threat(ac))
# Output: ThreatLevel.HIGH
```

5. **Monitor alerts** - Watch the Alerts tab for detections
6. **Check notifications** - If push notifications are configured, you'll receive mobile alerts

### List Existing Rules

```shell
curl http://localhost:5000/api/alerts/rules
```

### Delete a Rule

```shell
curl -X DELETE http://localhost:5000/api/alerts/rules/5
```

## Troubleshooting

### Too Many False Positives

**Problem:** Alerts triggered by light aircraft, ultralights, or paragliders
**Solution:**
- Add callsign exclusion patterns for known local aircraft
- Tighten altitude and speed thresholds
- Require category match (don't rely solely on altitude/speed)

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "alt", "operator": "lt", "value": 400},
      {"field": "gs", "operator": "lt", "value": 45},
      {"field": "category", "operator": "in", "value": ["B1", "B2"]},
      {"field": "callsign", "operator": "not_starts_with", "value": "N"}
    ]
  }
}
```

### Missing Drone Detections

**Problem:** Known drones in the area are not being detected
**Solution:**
- Many consumer drones don't broadcast ADS-B (only Remote ID on 1090 MHz or Bluetooth)
- Ensure your receiver supports Remote ID if available
- Lower detection thresholds temporarily to validate coverage
- Not all drones are equipped with transponders

### Remote ID Not Working

**Problem:** Drones with Remote ID are not triggering alerts
**Solution:**
- Remote ID broadcasts on 1090 MHz may require specific receiver support
- Verify your receiver firmware supports Remote ID message parsing
- Check if `remote_id` field is populated in aircraft data:

```shell
curl http://localhost:5000/api/aircraft | jq '.[] | select(.remote_id != null)'
```

### Distinguishing Drones from Helicopters

**Problem:** Low-flying helicopters trigger drone alerts
**Solution:** Add helicopter exclusion to your rule:

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "alt", "operator": "lt", "value": 500},
      {"field": "gs", "operator": "lt", "value": 60},
      {"field": "category", "operator": "not_in", "value": ["A7", "B7"]}
    ]
  }
}
```

### High Alert Volume

**Problem:** Getting too many alerts
**Solution:** Add cooldown and distance filtering:

```json
{
  "conditions": {
    "operator": "AND",
    "conditions": [
      {"field": "alt", "operator": "lt", "value": 500},
      {"field": "gs", "operator": "lt", "value": 60},
      {"field": "distance", "operator": "lt", "value": 3}
    ]
  },
  "cooldown": 600
}
```

This limits alerts to within 3nm and adds a 10-minute cooldown per aircraft.

## Related Recipes

- [Helicopter Watch](/docs/helicopter-watch) - Monitor helicopter traffic (exclude from drone alerts)
- [Altitude Filter](/docs/altitude-filter) - General low-flying aircraft detection
- [Geofence Alerts](/docs/geofence-alerts) - Monitor specific geographic areas
- [Emergency Alert Monitor](/docs/emergency-monitor) - Track emergency squawks
