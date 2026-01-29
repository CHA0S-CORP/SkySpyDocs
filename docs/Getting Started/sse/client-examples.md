---
title: "Client Examples"
slug: "sse/client-examples"
excerpt: "SSE client implementations in JavaScript, React, Python, and curl."
hidden: false
---

## JavaScript

```javascript
const stream = new EventSource('/api/v1/map/sse?replay_history=true');
const aircraft = new Map();

stream.addEventListener('aircraft_update', (e) => {
  const data = JSON.parse(e.data);
  data.aircraft.forEach(a => aircraft.set(a.hex, { ...aircraft.get(a.hex), ...a }));
});

stream.addEventListener('aircraft_new', (e) => {
  const data = JSON.parse(e.data);
  data.aircraft.forEach(a => aircraft.set(a.hex, a));
});

stream.addEventListener('aircraft_remove', (e) => {
  const data = JSON.parse(e.data);
  data.icaos.forEach(hex => aircraft.delete(hex));
});

stream.onerror = () => {
  console.error('Connection lost, reconnecting...');
  // EventSource auto-reconnects
};
```

---

## React Hook

```typescript
import { useEffect, useState } from 'react';

interface Aircraft {
  hex: string;
  flight?: string;
  lat: number;
  lon: number;
  alt: number;
}

export function useSkySpySSE(url = '/api/v1/map/sse?replay_history=true') {
  const [aircraft, setAircraft] = useState<Map<string, Aircraft>>(new Map());
  const [connected, setConnected] = useState(false);

  useEffect(() => {
    const stream = new EventSource(url);

    stream.onopen = () => setConnected(true);
    stream.onerror = () => setConnected(false);

    stream.addEventListener('aircraft_update', (e) => {
      const data = JSON.parse(e.data);
      setAircraft(prev => {
        const next = new Map(prev);
        data.aircraft.forEach((a: Aircraft) => {
          next.set(a.hex, { ...next.get(a.hex), ...a });
        });
        return next;
      });
    });

    stream.addEventListener('aircraft_new', (e) => {
      const data = JSON.parse(e.data);
      setAircraft(prev => {
        const next = new Map(prev);
        data.aircraft.forEach((a: Aircraft) => next.set(a.hex, a));
        return next;
      });
    });

    stream.addEventListener('aircraft_remove', (e) => {
      const data = JSON.parse(e.data);
      setAircraft(prev => {
        const next = new Map(prev);
        data.icaos.forEach((hex: string) => next.delete(hex));
        return next;
      });
    });

    return () => stream.close();
  }, [url]);

  return { aircraft: Array.from(aircraft.values()), connected };
}
```

### Usage

```tsx
function AircraftList() {
  const { aircraft, connected } = useSkySpySSE();

  if (!connected) return <div>🔄 Connecting...</div>;

  return (
    <ul>
      {aircraft.map(a => (
        <li key={a.hex}>
          ✈️ {a.flight || a.hex} - {a.alt}ft
        </li>
      ))}
    </ul>
  );
}
```

---

## Python

```python
import json
import sseclient
import requests

def stream_aircraft():
    url = 'http://localhost:5000/api/v1/map/sse?replay_history=true'
    response = requests.get(url, stream=True)
    client = sseclient.SSEClient(response)

    for event in client.events():
        data = json.loads(event.data)

        if event.event == 'aircraft_update':
            for aircraft in data['aircraft']:
                print(f"✈️ {aircraft.get('flight', aircraft['hex'])}: {aircraft.get('alt')}ft")

        elif event.event == 'safety_event':
            print(f"⚠️ {data['message']}")

if __name__ == '__main__':
    stream_aircraft()
```

Install dependencies:

```bash
pip install sseclient-py requests
```

---

## curl

```bash
curl -N http://localhost:5000/api/v1/map/sse
```

> 💡 Tip
>
> Use `-N` to disable buffering and see events in real-time.
