---
title: Node-RED Flows
excerpt: Visual automation flows for aircraft events.
hidden: false
recipe:
  color: '#8F0000'
  icon: 🔴
difficulty: intermediate
tags: [automation, node-red, iot, visual]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Node-RED installed (standalone or via Home Assistant)
- Basic understanding of Node-RED flows

## What You'll Build

Visual automation flows that process aircraft data in real-time, enabling complex filtering, transformations, and actions without writing code.

## Node-RED Setup

### Docker Quick Start

```shell
docker run -d --name nodered \
  -p 1880:1880 \
  -v nodered-data:/data \
  nodered/node-red
```

Access Node-RED at `http://localhost:1880`

### Install Required Nodes

Install via Palette Manager or npm:

```shell
cd ~/.node-red
npm install node-red-contrib-sse-client
npm install node-red-node-email
npm install node-red-contrib-discord
```

## Flow 1: Basic SSE Connection

This flow connects to SkySpy's SSE stream and processes aircraft updates.

```json SSE Connection Flow
[
  {
    "id": "skyspy-sse",
    "type": "sse-client",
    "name": "SkySpy SSE",
    "url": "http://localhost:5000/api/v1/map/sse",
    "events": "aircraft_update",
    "headers": {},
    "x": 130,
    "y": 100,
    "wires": [["parse-json"]]
  },
  {
    "id": "parse-json",
    "type": "json",
    "name": "Parse JSON",
    "property": "payload",
    "x": 310,
    "y": 100,
    "wires": [["split-aircraft"]]
  },
  {
    "id": "split-aircraft",
    "type": "split",
    "name": "Split Aircraft",
    "splt": "aircraft",
    "x": 490,
    "y": 100,
    "wires": [["debug"]]
  },
  {
    "id": "debug",
    "type": "debug",
    "name": "Aircraft Debug",
    "active": true,
    "tosidebar": true,
    "x": 670,
    "y": 100,
    "wires": []
  }
]
```

## Flow 2: Military Aircraft Alert

Filters for military aircraft and sends Discord notifications.

```json Military Alert Flow
[
  {
    "id": "skyspy-sse-2",
    "type": "sse-client",
    "name": "SkySpy SSE",
    "url": "http://localhost:5000/api/v1/map/sse",
    "events": "aircraft_update",
    "x": 130,
    "y": 200,
    "wires": [["parse-json-2"]]
  },
  {
    "id": "parse-json-2",
    "type": "json",
    "name": "Parse",
    "property": "payload",
    "x": 290,
    "y": 200,
    "wires": [["extract-aircraft"]]
  },
  {
    "id": "extract-aircraft",
    "type": "change",
    "name": "Extract Aircraft",
    "rules": [
      {
        "t": "set",
        "p": "payload",
        "pt": "msg",
        "to": "payload.aircraft",
        "tot": "msg"
      }
    ],
    "x": 450,
    "y": 200,
    "wires": [["split-2"]]
  },
  {
    "id": "split-2",
    "type": "split",
    "name": "Split",
    "splt": "\\n",
    "x": 590,
    "y": 200,
    "wires": [["filter-military"]]
  },
  {
    "id": "filter-military",
    "type": "switch",
    "name": "Military?",
    "property": "payload.military",
    "propertyType": "msg",
    "rules": [
      {"t": "true"}
    ],
    "x": 730,
    "y": 200,
    "wires": [["dedup-military"]]
  },
  {
    "id": "dedup-military",
    "type": "rbe",
    "name": "Deduplicate",
    "func": "rbe",
    "gap": "",
    "start": "",
    "inout": "out",
    "property": "payload.hex",
    "x": 880,
    "y": 200,
    "wires": [["format-discord"]]
  },
  {
    "id": "format-discord",
    "type": "template",
    "name": "Format Message",
    "field": "payload",
    "template": "🎖️ **Military Aircraft Spotted**\nCallsign: {{payload.flight}}\nType: {{payload.type}}\nAltitude: {{payload.alt}} ft\nDistance: {{payload.distance}} nm",
    "x": 1050,
    "y": 200,
    "wires": [["discord-webhook"]]
  },
  {
    "id": "discord-webhook",
    "type": "http request",
    "name": "Discord",
    "method": "POST",
    "url": "",
    "x": 1210,
    "y": 200,
    "wires": [[]]
  }
]
```

## Flow 3: Emergency Squawk Monitor

Monitors for emergency squawk codes with high-priority alerts.

```json Emergency Monitor Flow
[
  {
    "id": "emergency-filter",
    "type": "switch",
    "name": "Emergency Squawk?",
    "property": "payload.squawk",
    "propertyType": "msg",
    "rules": [
      {"t": "eq", "v": "7700", "vt": "str"},
      {"t": "eq", "v": "7600", "vt": "str"},
      {"t": "eq", "v": "7500", "vt": "str"}
    ],
    "checkall": "false",
    "repair": false,
    "x": 400,
    "y": 300,
    "wires": [
      ["emergency-7700"],
      ["emergency-7600"],
      ["emergency-7500"]
    ]
  },
  {
    "id": "emergency-7700",
    "type": "change",
    "name": "7700 Emergency",
    "rules": [
      {
        "t": "set",
        "p": "emergency_type",
        "pt": "msg",
        "to": "General Emergency",
        "tot": "str"
      }
    ],
    "x": 620,
    "y": 260,
    "wires": [["format-emergency"]]
  },
  {
    "id": "emergency-7600",
    "type": "change",
    "name": "7600 Radio Failure",
    "rules": [
      {
        "t": "set",
        "p": "emergency_type",
        "pt": "msg",
        "to": "Radio Failure",
        "tot": "str"
      }
    ],
    "x": 620,
    "y": 300,
    "wires": [["format-emergency"]]
  },
  {
    "id": "emergency-7500",
    "type": "change",
    "name": "7500 Hijack",
    "rules": [
      {
        "t": "set",
        "p": "emergency_type",
        "pt": "msg",
        "to": "Hijack",
        "tot": "str"
      }
    ],
    "x": 620,
    "y": 340,
    "wires": [["format-emergency"]]
  },
  {
    "id": "format-emergency",
    "type": "template",
    "name": "Format Alert",
    "template": "🚨 EMERGENCY AIRCRAFT\n\nSquawk: {{payload.squawk}} ({{emergency_type}})\nCallsign: {{payload.flight}}\nType: {{payload.type}}\nAltitude: {{payload.alt}} ft\nPosition: {{payload.lat}}, {{payload.lon}}",
    "x": 840,
    "y": 300,
    "wires": [["multi-notify"]]
  }
]
```

## Flow 4: Distance-Based Alerts

Alert when aircraft come within a specified distance.

```json Distance Alert Flow
[
  {
    "id": "distance-check",
    "type": "switch",
    "name": "Within 5nm?",
    "property": "payload.distance",
    "propertyType": "msg",
    "rules": [
      {"t": "lt", "v": "5", "vt": "num"}
    ],
    "x": 400,
    "y": 400,
    "wires": [["close-aircraft"]]
  },
  {
    "id": "close-aircraft",
    "type": "rbe",
    "name": "New Only",
    "func": "rbe",
    "property": "payload.hex",
    "x": 570,
    "y": 400,
    "wires": [["format-close"]]
  },
  {
    "id": "format-close",
    "type": "template",
    "name": "Format",
    "template": "✈️ Close aircraft: {{payload.flight}} ({{payload.type}}) at {{payload.distance}}nm, {{payload.alt}}ft",
    "x": 730,
    "y": 400,
    "wires": [["notify"]]
  }
]
```

## Flow 5: Aircraft Statistics

Aggregate statistics and store in context.

```json Statistics Flow
[
  {
    "id": "count-aircraft",
    "type": "function",
    "name": "Update Stats",
    "func": "// Get or initialize stats\nvar stats = flow.get('stats') || {\n    total_seen: 0,\n    military_count: 0,\n    types: {},\n    last_update: null\n};\n\nvar aircraft = msg.payload;\n\n// Update counts\nstats.total_seen++;\nif (aircraft.military) stats.military_count++;\n\n// Track types\nvar type = aircraft.type || 'unknown';\nstats.types[type] = (stats.types[type] || 0) + 1;\n\nstats.last_update = new Date().toISOString();\n\nflow.set('stats', stats);\n\nreturn msg;",
    "x": 400,
    "y": 500,
    "wires": [[]]
  },
  {
    "id": "get-stats",
    "type": "inject",
    "name": "Get Stats",
    "repeat": "60",
    "x": 130,
    "y": 560,
    "wires": [["read-stats"]]
  },
  {
    "id": "read-stats",
    "type": "function",
    "name": "Read Stats",
    "func": "msg.payload = flow.get('stats') || {};\nreturn msg;",
    "x": 300,
    "y": 560,
    "wires": [["stats-debug"]]
  },
  {
    "id": "stats-debug",
    "type": "debug",
    "name": "Stats",
    "active": true,
    "x": 470,
    "y": 560,
    "wires": []
  }
]
```

## Complete Flow: Import JSON

Import this complete flow into Node-RED:

```json Complete SkySpy Flow
[
  {
    "id": "flow-tab",
    "type": "tab",
    "label": "SkySpy Alerts",
    "disabled": false
  },
  {
    "id": "skyspy-sse-main",
    "type": "sse-client",
    "z": "flow-tab",
    "name": "SkySpy SSE",
    "url": "http://localhost:5000/api/v1/map/sse",
    "events": "aircraft_update",
    "x": 130,
    "y": 100,
    "wires": [["parse-main"]]
  },
  {
    "id": "parse-main",
    "type": "json",
    "z": "flow-tab",
    "name": "",
    "property": "payload",
    "x": 290,
    "y": 100,
    "wires": [["extract-main"]]
  },
  {
    "id": "extract-main",
    "type": "change",
    "z": "flow-tab",
    "name": "Extract",
    "rules": [{"t": "set", "p": "payload", "pt": "msg", "to": "payload.aircraft", "tot": "msg"}],
    "x": 430,
    "y": 100,
    "wires": [["split-main"]]
  },
  {
    "id": "split-main",
    "type": "split",
    "z": "flow-tab",
    "name": "",
    "x": 550,
    "y": 100,
    "wires": [["military-switch", "emergency-switch", "distance-switch"]]
  },
  {
    "id": "military-switch",
    "type": "switch",
    "z": "flow-tab",
    "name": "Military?",
    "property": "payload.military",
    "rules": [{"t": "true"}],
    "x": 720,
    "y": 60,
    "wires": [["military-dedup"]]
  },
  {
    "id": "military-dedup",
    "type": "rbe",
    "z": "flow-tab",
    "name": "Dedup",
    "func": "rbe",
    "property": "payload.hex",
    "x": 870,
    "y": 60,
    "wires": [["military-msg"]]
  },
  {
    "id": "military-msg",
    "type": "template",
    "z": "flow-tab",
    "name": "Format",
    "template": "🎖️ Military: {{payload.flight}} ({{payload.type}}) at {{payload.alt}}ft",
    "x": 1010,
    "y": 60,
    "wires": [["notify-all"]]
  },
  {
    "id": "emergency-switch",
    "type": "switch",
    "z": "flow-tab",
    "name": "Emergency?",
    "property": "payload.squawk",
    "rules": [
      {"t": "eq", "v": "7700", "vt": "str"},
      {"t": "eq", "v": "7600", "vt": "str"},
      {"t": "eq", "v": "7500", "vt": "str"}
    ],
    "x": 720,
    "y": 120,
    "wires": [["emergency-dedup"], ["emergency-dedup"], ["emergency-dedup"]]
  },
  {
    "id": "emergency-dedup",
    "type": "rbe",
    "z": "flow-tab",
    "name": "Dedup",
    "property": "payload.hex",
    "x": 870,
    "y": 120,
    "wires": [["emergency-msg"]]
  },
  {
    "id": "emergency-msg",
    "type": "template",
    "z": "flow-tab",
    "name": "Format",
    "template": "🚨 EMERGENCY: {{payload.flight}} squawking {{payload.squawk}} at {{payload.alt}}ft",
    "x": 1010,
    "y": 120,
    "wires": [["notify-all"]]
  },
  {
    "id": "distance-switch",
    "type": "switch",
    "z": "flow-tab",
    "name": "Close?",
    "property": "payload.distance",
    "rules": [{"t": "lt", "v": "3", "vt": "num"}],
    "x": 710,
    "y": 180,
    "wires": [["distance-dedup"]]
  },
  {
    "id": "distance-dedup",
    "type": "rbe",
    "z": "flow-tab",
    "name": "Dedup",
    "property": "payload.hex",
    "x": 870,
    "y": 180,
    "wires": [["distance-msg"]]
  },
  {
    "id": "distance-msg",
    "type": "template",
    "z": "flow-tab",
    "name": "Format",
    "template": "✈️ Close: {{payload.flight}} at {{payload.distance}}nm",
    "x": 1010,
    "y": 180,
    "wires": [["notify-all"]]
  },
  {
    "id": "notify-all",
    "type": "debug",
    "z": "flow-tab",
    "name": "Notifications",
    "active": true,
    "tosidebar": true,
    "x": 1190,
    "y": 120,
    "wires": []
  }
]
```

## Configuration

Configure the SSE URL in the `sse-client` node:

| Setting | Value |
|---------|-------|
| URL | `http://localhost:5000/api/v1/map/sse` |
| Events | `aircraft_update` |

## Node Types Used

| Node | Purpose |
|------|---------|
| `sse-client` | Connect to SkySpy SSE stream |
| `json` | Parse JSON payloads |
| `split` | Process each aircraft individually |
| `switch` | Filter by conditions |
| `rbe` | Deduplicate (report by exception) |
| `template` | Format messages |
| `function` | Custom JavaScript logic |
| `change` | Transform message properties |

## Adding Notification Outputs

Replace the debug node with actual notification nodes:

### Discord Webhook
```json
{
  "type": "http request",
  "method": "POST",
  "url": "https://discord.com/api/webhooks/...",
  "paytoqs": "ignore"
}
```

### Email
```json
{
  "type": "e-mail",
  "server": "smtp.gmail.com",
  "port": "465",
  "secure": true,
  "to": "you@example.com"
}
```

### MQTT
```json
{
  "type": "mqtt out",
  "topic": "skyspy/alerts",
  "broker": "mqtt-broker-config"
}
```

## Testing & Verification

1. **Import the flow** into Node-RED
2. **Configure the SSE URL** to point to SkySpy
3. **Deploy the flow**
4. **Watch the debug sidebar** for aircraft events
5. **Modify filters** as needed

## Troubleshooting

### SSE Connection Failing

**Problem:** Red triangle on SSE node
**Solution:**
- Verify SkySpy URL is correct
- Check network connectivity
- Ensure SkySpy is running

### No Messages Flowing

**Problem:** Flow deployed but no output
**Solution:**
- Check SSE node status (should be "connected")
- Verify aircraft are being received
- Check switch node conditions

### Duplicate Alerts

**Problem:** Same aircraft alerting repeatedly
**Solution:**
- Add RBE (report by exception) node
- Use `payload.hex` as property
- Set appropriate mode

## Related Recipes

- [Home Assistant](/docs/home-assistant) - HA integration
- [MQTT Publisher](/docs/mqtt-publisher) - MQTT output
- [Webhook Notifications](/docs/webhook-notifications) - Generic webhooks
- [IFTTT Applets](/docs/ifttt-integration) - IFTTT triggers
