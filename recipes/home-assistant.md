---
title: Home Assistant
description: Create aircraft sensors and automations in Home Assistant.
hidden: false
recipe:
  color: '#41BDF5'
  icon: 🏠
---
```shell Shell
curl -X POST http://homeassistant.local:8123/api/states/sensor.aircraft_count \
  -H "Authorization: Bearer $HA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "state": "5",
    "attributes": {
      "friendly_name": "Aircraft Overhead",
      "unit_of_measurement": "aircraft",
      "military_count": 1
    }
  }'
```

```go Go
package main

import (
	"bytes"
	"encoding/json"
	"net/http"
	"os"
)

type SensorState struct {
	State      string                 `json:"state"`
	Attributes map[string]interface{} `json:"attributes"`
}

func updateSensor(entityID string, state SensorState) {
	haURL := os.Getenv("HA_URL")
	token := os.Getenv("HA_TOKEN")

	body, _ := json.Marshal(state)
	req, _ := http.NewRequest("POST", haURL+"/api/states/"+entityID, bytes.NewReader(body))
	req.Header.Set("Authorization", "Bearer "+token)
	req.Header.Set("Content-Type", "application/json")

	http.DefaultClient.Do(req)
}

func main() {
	updateSensor("sensor.aircraft_count", SensorState{
		State: "5",
		Attributes: map[string]interface{}{
			"friendly_name":       "Aircraft Overhead",
			"unit_of_measurement": "aircraft",
			"military_count":      1,
		},
	})
}
```

```python Python
import os
import requests

HA_URL = os.getenv("HA_URL", "http://homeassistant.local:8123")
HA_TOKEN = os.getenv("HA_TOKEN")

def update_sensor(entity_id, state, attributes):
    url = f"{HA_URL}/api/states/{entity_id}"
    headers = {
        "Authorization": f"Bearer {HA_TOKEN}",
        "Content-Type": "application/json"
    }
    payload = {"state": str(state), "attributes": attributes}
    requests.post(url, json=payload, headers=headers)

def update_aircraft_count(total, military=0):
    update_sensor("sensor.aircraft_count", total, {
        "friendly_name": "Aircraft Overhead",
        "unit_of_measurement": "aircraft",
        "military_count": military
    })

# Example usage
update_aircraft_count(5, military=1)
```

```javascript JavaScript
const HA_URL = process.env.HA_URL || 'http://homeassistant.local:8123';
const HA_TOKEN = process.env.HA_TOKEN;

async function updateSensor(entityId, state, attributes) {
    await fetch(`${HA_URL}/api/states/${entityId}`, {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${HA_TOKEN}`,
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({ state: String(state), attributes }),
    });
}

async function updateAircraftCount(total, military = 0) {
    await updateSensor('sensor.aircraft_count', total, {
        friendly_name: 'Aircraft Overhead',
        unit_of_measurement: 'aircraft',
        military_count: military,
    });
}

// Example usage
updateAircraftCount(5, 1);
```

```json Response Example
{"entity_id": "sensor.aircraft_count", "state": "5", "attributes": {"friendly_name": "Aircraft Overhead"}}
```

# Configure Home Assistant API

<!-- shell@1-3 -->
<!-- go@1-11 -->
<!-- python@1-5 -->
<!-- javascript@1-2 -->

Get a Long-Lived Access Token from your Home Assistant profile page. Set the HA_URL and HA_TOKEN environment variables.

# Create Sensor Update Function

<!-- go@13-24 -->
<!-- python@7-14 -->
<!-- javascript@4-13 -->

Build a reusable function to update any Home Assistant sensor entity. The state is the main value, and attributes hold additional metadata.

# Update Aircraft Sensor

<!-- shell@4-11 -->
<!-- go@26-35 -->
<!-- python@16-22 -->
<!-- javascript@15-21 -->

Create or update the aircraft count sensor with the current count and any additional attributes like military aircraft count.
