---
title: MQTT Publisher
excerpt: Publish aircraft data to MQTT brokers.
hidden: false
recipe:
  color: '#660066'
  icon: 📡
difficulty: intermediate
tags: [smart-home, mqtt, iot, integration]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- MQTT broker (Mosquitto, HiveMQ, etc.)
- Python: `pip install sseclient-py requests paho-mqtt`

## What You'll Build

An MQTT publisher that streams aircraft data to your broker, enabling integration with Home Assistant, Node-RED, and other MQTT-compatible systems.

```shell Shell
pip install sseclient-py requests paho-mqtt
export MQTT_BROKER="localhost"
export MQTT_PORT="1883"
python mqtt_publisher.py
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

    mqtt "github.com/eclipse/paho.mqtt.golang"
)

func main() {
    broker := os.Getenv("MQTT_BROKER")
    if broker == "" {
        broker = "tcp://localhost:1883"
    }
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    opts := mqtt.NewClientOptions().AddBroker(broker)
    opts.SetClientID("skyspy-publisher")
    opts.SetAutoReconnect(true)

    client := mqtt.NewClient(opts)
    if token := client.Connect(); token.Wait() && token.Error() != nil {
        fmt.Printf("MQTT connection failed: %v\n", token.Error())
        os.Exit(1)
    }
    defer client.Disconnect(250)

    fmt.Println("MQTT connected, publishing aircraft data...")

    seen := make(map[string]time.Time)

    for {
        resp, _ := http.Get(skyspyURL + "/api/v1/map/sse")
        scanner := bufio.NewScanner(resp.Body)

        for scanner.Scan() {
            line := scanner.Text()
            if strings.HasPrefix(line, "data:") {
                var data map[string]interface{}
                json.Unmarshal([]byte(line[5:]), &data)

                if aircraft, ok := data["aircraft"].([]interface{}); ok {
                    // Publish summary
                    summary := map[string]interface{}{
                        "count":    len(aircraft),
                        "military": countMilitary(aircraft),
                    }
                    summaryJSON, _ := json.Marshal(summary)
                    client.Publish("skyspy/summary", 0, false, summaryJSON)

                    // Publish individual aircraft
                    for _, a := range aircraft {
                        ac := a.(map[string]interface{})
                        hex := fmt.Sprint(ac["hex"])

                        if last, exists := seen[hex]; exists && time.Since(last) < 10*time.Second {
                            continue
                        }
                        seen[hex] = time.Now()

                        acJSON, _ := json.Marshal(ac)
                        client.Publish(fmt.Sprintf("skyspy/aircraft/%s", hex), 0, false, acJSON)

                        if ac["military"] == true {
                            client.Publish("skyspy/military", 0, false, acJSON)
                        }
                    }
                }
            }
        }
        resp.Body.Close()
    }
}

func countMilitary(aircraft []interface{}) int {
    count := 0
    for _, a := range aircraft {
        if ac, ok := a.(map[string]interface{}); ok && ac["military"] == true {
            count++
        }
    }
    return count
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
import paho.mqtt.client as mqtt

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
MQTT_BROKER = os.getenv("MQTT_BROKER", "localhost")
MQTT_PORT = int(os.getenv("MQTT_PORT", "1883"))
MQTT_USER = os.getenv("MQTT_USER", "")
MQTT_PASS = os.getenv("MQTT_PASS", "")
MQTT_PREFIX = os.getenv("MQTT_PREFIX", "skyspy")

# Connect to MQTT
client = mqtt.Client(client_id="skyspy-publisher")
if MQTT_USER:
    client.username_pw_set(MQTT_USER, MQTT_PASS)

try:
    client.connect(MQTT_BROKER, MQTT_PORT, 60)
    client.loop_start()
    print(f"MQTT connected to {MQTT_BROKER}:{MQTT_PORT}")
except Exception as e:
    print(f"MQTT connection failed: {e}")
    sys.exit(1)

seen = {}


def publish(topic, payload, retain=False):
    """Publish JSON payload to MQTT topic."""
    client.publish(f"{MQTT_PREFIX}/{topic}", json.dumps(payload), retain=retain)


def process_aircraft(aircraft_list):
    """Process and publish aircraft data."""
    now = time.time()

    # Summary
    total = len(aircraft_list)
    military = sum(1 for a in aircraft_list if a.get("military"))
    closest = min((a.get("distance", 999) for a in aircraft_list), default=0)

    publish("summary", {
        "count": total,
        "military": military,
        "closest": round(closest, 1),
        "timestamp": datetime.utcnow().isoformat() + "Z"
    }, retain=True)

    # Individual aircraft
    for aircraft in aircraft_list:
        hex_code = aircraft.get("hex")

        # Rate limit per aircraft
        if hex_code in seen and now - seen[hex_code] < 10:
            continue
        seen[hex_code] = now

        # Clean up payload
        payload = {
            "hex": hex_code,
            "flight": aircraft.get("flight"),
            "type": aircraft.get("type"),
            "altitude": aircraft.get("alt"),
            "speed": aircraft.get("gs"),
            "track": aircraft.get("track"),
            "lat": aircraft.get("lat"),
            "lon": aircraft.get("lon"),
            "distance": aircraft.get("distance"),
            "military": aircraft.get("military", False),
            "squawk": aircraft.get("squawk"),
            "timestamp": datetime.utcnow().isoformat() + "Z"
        }

        # Publish to aircraft-specific topic
        publish(f"aircraft/{hex_code}", payload)

        # Publish to category topics
        if aircraft.get("military"):
            publish("military", payload)

        squawk = aircraft.get("squawk", "")
        if squawk in ["7700", "7600", "7500"]:
            publish(f"emergency/{squawk}", payload)

    # Cleanup old entries
    cutoff = now - 120
    for hex_code in list(seen.keys()):
        if seen[hex_code] < cutoff:
            del seen[hex_code]
            # Publish empty to remove retained message
            publish(f"aircraft/{hex_code}", {}, retain=True)


def main():
    print(f"Publishing to {MQTT_PREFIX}/* ...")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client_sse = sseclient.SSEClient(response)

            for event in client_sse.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    aircraft_list = data.get("aircraft", [])
                    if aircraft_list:
                        process_aircraft(aircraft_list)

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const mqtt = require('mqtt');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const MQTT_BROKER = process.env.MQTT_BROKER || 'mqtt://localhost:1883';
const MQTT_PREFIX = process.env.MQTT_PREFIX || 'skyspy';

const client = mqtt.connect(MQTT_BROKER);
const seen = new Map();

client.on('connect', () => {
  console.log('MQTT connected');
});

function publish(topic, payload, retain = false) {
  client.publish(`${MQTT_PREFIX}/${topic}`, JSON.stringify(payload), { retain });
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);
  const aircraft = data.aircraft || [];
  const now = Date.now();

  // Summary
  publish('summary', {
    count: aircraft.length,
    military: aircraft.filter(a => a.military).length,
    timestamp: new Date().toISOString()
  }, true);

  // Individual aircraft
  for (const ac of aircraft) {
    if (seen.has(ac.hex) && now - seen.get(ac.hex) < 10000) continue;
    seen.set(ac.hex, now);

    publish(`aircraft/${ac.hex}`, ac);

    if (ac.military) {
      publish('military', ac);
    }
  }
});
```

```yaml Home Assistant MQTT Sensors
# configuration.yaml

mqtt:
  sensor:
    - name: "SkySpy Aircraft Count"
      state_topic: "skyspy/summary"
      value_template: "{{ value_json.count }}"
      icon: mdi:airplane

    - name: "SkySpy Military Count"
      state_topic: "skyspy/summary"
      value_template: "{{ value_json.military }}"
      icon: mdi:shield-airplane

    - name: "SkySpy Closest Aircraft"
      state_topic: "skyspy/summary"
      value_template: "{{ value_json.closest }}"
      unit_of_measurement: "nm"
      icon: mdi:map-marker-distance

  binary_sensor:
    - name: "Military Aircraft Nearby"
      state_topic: "skyspy/military"
      payload_on: "True"
      device_class: presence
```

## Topic Structure

| Topic | Description | Retained |
|-------|-------------|----------|
| `skyspy/summary` | Count and stats | Yes |
| `skyspy/aircraft/{hex}` | Individual aircraft | No |
| `skyspy/military` | Military aircraft only | No |
| `skyspy/emergency/{squawk}` | Emergency squawks | No |

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `MQTT_BROKER` | MQTT broker hostname | `localhost` |
| `MQTT_PORT` | MQTT broker port | `1883` |
| `MQTT_USER` | MQTT username | None |
| `MQTT_PASS` | MQTT password | None |
| `MQTT_PREFIX` | Topic prefix | `skyspy` |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### TLS/SSL

For secure connections:

```python
import ssl

client.tls_set(
    ca_certs="/path/to/ca.crt",
    certfile="/path/to/client.crt",
    keyfile="/path/to/client.key",
    tls_version=ssl.PROTOCOL_TLSv1_2
)
```

### QoS Levels

| QoS | Description | Use Case |
|-----|-------------|----------|
| 0 | At most once | Real-time updates |
| 1 | At least once | Important alerts |
| 2 | Exactly once | Critical data |

```python
client.publish(topic, payload, qos=1)
```

## Testing & Verification

1. **Start MQTT broker** (or use cloud broker)
2. **Subscribe to topics**:
   ```shell
   mosquitto_sub -h localhost -t "skyspy/#" -v
   ```
3. **Start the publisher** and verify messages appear
4. **Check Home Assistant** if using MQTT sensors

### Test Broker

Use HiveMQ public broker for testing:

```shell
export MQTT_BROKER="broker.hivemq.com"
export MQTT_PORT="1883"
```

## Troubleshooting

### Connection Refused

**Problem:** Cannot connect to broker
**Solution:**
- Verify broker is running
- Check firewall rules
- Confirm port is correct

### Messages Not Appearing

**Problem:** Publishing but nothing received
**Solution:**
- Check topic names match subscriptions
- Verify no ACL blocking
- Check for connection errors

### Retained Messages Not Clearing

**Problem:** Old aircraft still show in HA
**Solution:**
- Publish empty payload with retain flag to clear
- Implement cleanup in publisher

## Related Recipes

- [Home Assistant Integration](/docs/home-assistant) - Direct HA API
- [Node-RED Flows](/docs/node-red-flows) - Visual automation
- [Grafana Dashboard](/docs/grafana-dashboard) - Metrics visualization
- [ntfy.sh Integration](/docs/ntfy-integration) - Push notifications
