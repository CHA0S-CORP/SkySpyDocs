---
title: MQTT Publisher
description: Publish aircraft data to MQTT brokers.
hidden: false
recipe:
  color: '#660066'
  icon: 📡
---
```shell Shell
mosquitto_pub -h localhost -t "skyspy/aircraft/military" \
  -m '{"hex":"A12345","flight":"RCH419","alt":35000}'
```

```go Go
package main

import (
	"encoding/json"
	"os"

	mqtt "github.com/eclipse/paho.mqtt.golang"
)

func main() {
	broker := os.Getenv("MQTT_BROKER")
	if broker == "" {
		broker = "tcp://localhost:1883"
	}

	opts := mqtt.NewClientOptions().AddBroker(broker)
	client := mqtt.NewClient(opts)
	client.Connect().Wait()
	defer client.Disconnect(250)

	aircraft := map[string]interface{}{
		"hex":    "A12345",
		"flight": "RCH419",
		"alt":    35000,
		"type":   "C-17",
	}

	payload, _ := json.Marshal(aircraft)
	client.Publish("skyspy/aircraft/military", 0, false, payload)
}
```

```python Python
import json
import os
import paho.mqtt.client as mqtt

MQTT_BROKER = os.getenv("MQTT_BROKER", "localhost")
MQTT_PORT = int(os.getenv("MQTT_PORT", "1883"))

client = mqtt.Client()
client.connect(MQTT_BROKER, MQTT_PORT)

def publish_aircraft(topic, aircraft):
    payload = json.dumps(aircraft)
    client.publish(topic, payload)

# Example usage
publish_aircraft("skyspy/aircraft/military", {
    "hex": "A12345",
    "flight": "RCH419",
    "alt": 35000,
    "type": "C-17"
})

client.disconnect()
```

```javascript JavaScript
const mqtt = require('mqtt');

const MQTT_BROKER = process.env.MQTT_BROKER || 'mqtt://localhost:1883';
const client = mqtt.connect(MQTT_BROKER);

client.on('connect', () => {
    const aircraft = {
        hex: 'A12345',
        flight: 'RCH419',
        alt: 35000,
        type: 'C-17',
    };

    client.publish('skyspy/aircraft/military', JSON.stringify(aircraft));
    client.end();
});
```

```json Response Example
{"topic": "skyspy/aircraft/military", "qos": 0, "retained": false}
```

# Connect to MQTT Broker

<!-- shell@1 -->
<!-- go@1-18 -->
<!-- python@1-9 -->
<!-- javascript@1-5 -->

Connect to your MQTT broker. Use Mosquitto, HiveMQ, or any MQTT 3.1.1+ compatible broker.

# Build Aircraft Payload

<!-- go@20-26 -->
<!-- python@11-13 -->
<!-- javascript@7-12 -->

Structure the aircraft data as JSON with hex code, callsign, altitude, and type.

# Publish to Topic

<!-- shell@2 -->
<!-- go@28-29 -->
<!-- python@15-20 -->
<!-- javascript@14-15 -->

Publish to topic paths like `skyspy/aircraft/military` or `skyspy/aircraft/{hex}` for per-aircraft topics.
