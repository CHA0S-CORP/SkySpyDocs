---
title: "React Hook"
slug: "real-time-api/react-hook"
excerpt: "useSkySpySocket hook for React applications."
hidden: false
---

## useSkySpySocket Hook

```typescript
import { useEffect, useState, useCallback, useRef } from 'react';
import { io, Socket } from 'socket.io-client';

interface Aircraft {
  hex: string;
  flight?: string;
  lat: number;
  lon: number;
  alt: number;
  gs: number;
  track: number;
  vr: number;
}

interface UseSkySpySocketOptions {
  url?: string;
  topics?: string;
}

export function useSkySpySocket(options: UseSkySpySocketOptions = {}) {
  const { url = 'http://localhost:5000', topics = 'aircraft' } = options;
  const [connected, setConnected] = useState(false);
  const [aircraft, setAircraft] = useState<Map<string, Aircraft>>(new Map());
  const socketRef = useRef<Socket | null>(null);

  useEffect(() => {
    const socket = io(url, {
      path: '/socket.io/socket.io',
      query: { topics },
      transports: ['websocket', 'polling']
    });

    socketRef.current = socket;

    socket.on('connect', () => setConnected(true));
    socket.on('disconnect', () => setConnected(false));

    socket.on('aircraft:snapshot', (data) => {
      setAircraft(new Map(data.aircraft.map((a: Aircraft) => [a.hex, a])));
    });

    socket.on('aircraft:update', (data) => {
      setAircraft((prev) => {
        const next = new Map(prev);
        for (const a of data.aircraft) {
          const existing = next.get(a.hex);
          next.set(a.hex, { ...existing, ...a });
        }
        return next;
      });
    });

    socket.on('aircraft:remove', (data) => {
      setAircraft((prev) => {
        const next = new Map(prev);
        for (const icao of data.icaos) {
          next.delete(icao);
        }
        return next;
      });
    });

    return () => {
      socket.disconnect();
    };
  }, [url, topics]);

  return {
    connected,
    aircraft: Array.from(aircraft.values()),
    socket: socketRef.current
  };
}
```

---

## Usage Example

```tsx
function AircraftList() {
  const { connected, aircraft } = useSkySpySocket({
    topics: 'aircraft,safety'
  });

  if (!connected) {
    return <div>🔄 Connecting...</div>;
  }

  return (
    <div>
      <h2>✈️ {aircraft.length} Aircraft</h2>
      <ul>
        {aircraft.map((a) => (
          <li key={a.hex}>
            {a.flight || a.hex} - {a.alt}ft @ {a.gs}kts
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## With Safety Events

```tsx
function SafetyMonitor() {
  const { socket, connected } = useSkySpySocket({
    topics: 'safety'
  });
  const [alerts, setAlerts] = useState<any[]>([]);

  useEffect(() => {
    if (!socket) return;

    socket.on('safety:event', (event) => {
      setAlerts((prev) => [event, ...prev].slice(0, 50));
    });
  }, [socket]);

  return (
    <div>
      <h2>🛡️ Safety Alerts</h2>
      {alerts.map((alert, i) => (
        <div key={i} className={alert.severity}>
          {alert.message}
        </div>
      ))}
    </div>
  );
}
```
