---
title: Node-RED Flow
description: Build visual automation flows with Node-RED.
hidden: false
recipe:
  color: '#8F0000'
  icon: 🔴
---
```shell Shell
curl -X POST http://localhost:1880/flows \
  -H "Content-Type: application/json" \
  -d '[{"id":"skyspy-sse","type":"sse-client","url":"http://localhost:5000/api/v1/map/sse"}]'
```

```go Go
package main

import (
	"bytes"
	"encoding/json"
	"net/http"
)

type NodeREDFlow struct {
	ID    string `json:"id"`
	Type  string `json:"type"`
	Name  string `json:"name,omitempty"`
	URL   string `json:"url,omitempty"`
	Wires [][]string `json:"wires,omitempty"`
}

func main() {
	flow := []NodeREDFlow{
		{ID: "sse-in", Type: "sse-client", Name: "SkySpy SSE", URL: "http://localhost:5000/api/v1/map/sse", Wires: [][]string{{"filter"}}},
		{ID: "filter", Type: "function", Name: "Military Filter", Wires: [][]string{{"debug"}}},
		{ID: "debug", Type: "debug", Name: "Output"},
	}

	body, _ := json.Marshal(flow)
	http.Post("http://localhost:1880/flows", "application/json", bytes.NewReader(body))
}
```

```python Python
import requests

NODE_RED_URL = "http://localhost:1880"

flow = [
    {
        "id": "sse-in",
        "type": "sse-client",
        "name": "SkySpy SSE",
        "url": "http://localhost:5000/api/v1/map/sse",
        "wires": [["filter"]]
    },
    {
        "id": "filter",
        "type": "function",
        "name": "Military Filter",
        "func": "if (msg.payload.military) { return msg; }",
        "wires": [["notify"]]
    },
    {
        "id": "notify",
        "type": "debug",
        "name": "Alert Output"
    }
]

requests.post(f"{NODE_RED_URL}/flows", json=flow)
```

```javascript JavaScript
const NODE_RED_URL = 'http://localhost:1880';

const flow = [
    {
        id: 'sse-in',
        type: 'sse-client',
        name: 'SkySpy SSE',
        url: 'http://localhost:5000/api/v1/map/sse',
        wires: [['filter']],
    },
    {
        id: 'filter',
        type: 'function',
        name: 'Military Filter',
        func: 'if (msg.payload.military) { return msg; }',
        wires: [['notify']],
    },
    {
        id: 'notify',
        type: 'debug',
        name: 'Alert Output',
    },
];

fetch(`${NODE_RED_URL}/flows`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(flow),
});
```

```json Response Example
{"id": "flow123", "rev": "1-abc"}
```

# Define SSE Input Node

<!-- shell@1-3 -->
<!-- go@17-17 -->
<!-- python@6-12 -->
<!-- javascript@4-10 -->

Create an SSE client node that connects to SkySpy's real-time stream. This receives aircraft updates continuously.

# Add Filter Function

<!-- go@18-18 -->
<!-- python@13-19 -->
<!-- javascript@11-17 -->

Use a function node to filter aircraft. This example passes only military aircraft to the next node.

# Connect to Output

<!-- go@19-19 -->
<!-- python@20-24 -->
<!-- javascript@18-22 -->

Wire to debug, notification, or any Node-RED output node. Build complex automations visually in the Node-RED editor.
