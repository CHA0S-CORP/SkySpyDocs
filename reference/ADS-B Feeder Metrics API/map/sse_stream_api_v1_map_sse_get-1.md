---
title: SSE Aircraft Stream
excerpt: |-
  Server-Sent Events stream for real-time aircraft updates.

  **Event Types:**
  - `aircraft_update`: Periodic aircraft position updates
  - `alert_triggered`: When an alert rule matches
  - `safety_event`: TCAS/safety event detected
  - `acars_message`: ACARS/VDL2 message received

  **Connection:**
  ```javascript
  const sse = new EventSource('/api/v1/map/sse');
  sse.addEventListener('aircraft_update', (e) => {
      const data = JSON.parse(e.data);
      console.log(data.aircraft);
  });
  ```

  **Query Parameters:**
  - `replay_history=true`: Receive recent events on connect

  The connection will send a heartbeat every 30 seconds to keep alive.
api:
  file: openapi.json
  operationId: sse_stream_api_v1_map_sse_get
hidden: false
---