---
title: "SSE Hook"
slug: "live-dashboard/sse-hook"
excerpt: "Create the useSkySpySSE React hook for real-time data."
hidden: false
---

Create `src/hooks/useSkySpySSE.ts`:

```typescript
import { useEffect, useState, useCallback } from 'react';

export interface Aircraft {
  hex: string;
  flight?: string;
  lat?: number;
  lon?: number;
  alt?: number;
  gs?: number;
  track?: number;
  vr?: number;
  squawk?: string;
  category?: string;
  type?: string;
  military?: boolean;
  emergency?: boolean;
  distance?: number;
}

export interface SafetyEvent {
  event_type: string;
  severity: string;
  icao: string;
  message: string;
  timestamp: string;
}

interface UseSkySpySSEOptions {
  url?: string;
  onSafetyEvent?: (event: SafetyEvent) => void;
}

export function useSkySpySSE(options: UseSkySpySSEOptions = {}) {
  const {
    url = 'http://localhost:5000/api/v1/map/sse?replay_history=true',
    onSafetyEvent
  } = options;

  const [aircraft, setAircraft] = useState<Map<string, Aircraft>>(new Map());
  const [connected, setConnected] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const eventSource = new EventSource(url);

    eventSource.onopen = () => {
      setConnected(true);
      setError(null);
    };

    eventSource.onerror = () => {
      setConnected(false);
      setError('Connection lost. Reconnecting...');
    };

    // Handle aircraft updates
    eventSource.addEventListener('aircraft_update', (e) => {
      const data = JSON.parse(e.data);
      setAircraft(prev => {
        const next = new Map(prev);
        data.aircraft.forEach((a: Aircraft) => {
          const existing = next.get(a.hex);
          next.set(a.hex, { ...existing, ...a });
        });
        return next;
      });
    });

    // Handle new aircraft
    eventSource.addEventListener('aircraft_new', (e) => {
      const data = JSON.parse(e.data);
      setAircraft(prev => {
        const next = new Map(prev);
        data.aircraft.forEach((a: Aircraft) => next.set(a.hex, a));
        return next;
      });
    });

    // Handle aircraft removal
    eventSource.addEventListener('aircraft_remove', (e) => {
      const data = JSON.parse(e.data);
      setAircraft(prev => {
        const next = new Map(prev);
        data.icaos.forEach((hex: string) => next.delete(hex));
        return next;
      });
    });

    // Handle safety events
    eventSource.addEventListener('safety_event', (e) => {
      const data = JSON.parse(e.data);
      onSafetyEvent?.(data);
    });

    return () => {
      eventSource.close();
    };
  }, [url, onSafetyEvent]);

  return {
    aircraft: Array.from(aircraft.values()),
    connected,
    error,
    count: aircraft.size
  };
}
```

## Usage

```tsx
const { aircraft, connected, count } = useSkySpySSE({
  url: 'http://localhost:5000/api/v1/map/sse',
  onSafetyEvent: (event) => console.log('Safety:', event)
});
```
