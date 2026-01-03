---
title: "Components"
slug: "live-dashboard/components"
excerpt: "Build the dashboard UI components."
hidden: false
---

## AircraftList

Create `src/components/AircraftList.tsx`:

```tsx
import { useState, useMemo } from 'react';
import { Aircraft } from '../hooks/useSkySpySSE';

interface AircraftListProps {
  aircraft: Aircraft[];
}

type SortField = 'flight' | 'alt' | 'gs' | 'distance';

export function AircraftList({ aircraft }: AircraftListProps) {
  const [sortField, setSortField] = useState<SortField>('distance');
  const [sortDirection, setSortDirection] = useState<'asc' | 'desc'>('asc');
  const [filter, setFilter] = useState('');

  const sortedAircraft = useMemo(() => {
    return [...aircraft]
      .filter(a => {
        if (!filter) return true;
        const search = filter.toLowerCase();
        return (
          a.hex.toLowerCase().includes(search) ||
          a.flight?.toLowerCase().includes(search) ||
          a.type?.toLowerCase().includes(search)
        );
      })
      .sort((a, b) => {
        const aVal = a[sortField] ?? 0;
        const bVal = b[sortField] ?? 0;
        const cmp = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
        return sortDirection === 'asc' ? cmp : -cmp;
      });
  }, [aircraft, sortField, sortDirection, filter]);

  return (
    <div className="aircraft-list">
      <div className="list-header">
        <h2>Aircraft ({sortedAircraft.length})</h2>
        <input
          type="text"
          placeholder="Filter..."
          value={filter}
          onChange={e => setFilter(e.target.value)}
        />
      </div>
      <table>
        <thead>
          <tr>
            <th>Callsign</th>
            <th>ICAO</th>
            <th>Type</th>
            <th>Altitude</th>
            <th>Speed</th>
            <th>Distance</th>
          </tr>
        </thead>
        <tbody>
          {sortedAircraft.map(a => (
            <tr key={a.hex} className={a.emergency ? 'emergency' : ''}>
              <td>{a.flight || '—'}</td>
              <td>{a.hex}</td>
              <td>{a.type || '—'}</td>
              <td>{a.alt ? `${a.alt.toLocaleString()} ft` : '—'}</td>
              <td>{a.gs ? `${a.gs} kts` : '—'}</td>
              <td>{a.distance ? `${a.distance.toFixed(1)} NM` : '—'}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## Statistics

Create `src/components/Statistics.tsx`:

```tsx
import { useMemo } from 'react';
import { Aircraft } from '../hooks/useSkySpySSE';

export function Statistics({ aircraft }: { aircraft: Aircraft[] }) {
  const stats = useMemo(() => {
    const withAlt = aircraft.filter(a => a.alt !== undefined);
    const withDist = aircraft.filter(a => a.distance !== undefined);

    return {
      total: aircraft.length,
      military: aircraft.filter(a => a.military).length,
      emergency: aircraft.filter(a => a.emergency).length,
      avgAltitude: withAlt.length
        ? Math.round(withAlt.reduce((sum, a) => sum + (a.alt || 0), 0) / withAlt.length)
        : 0,
      maxDistance: withDist.length
        ? Math.max(...withDist.map(a => a.distance || 0))
        : 0,
    };
  }, [aircraft]);

  return (
    <div className="statistics">
      <div className="stat-card">
        <div className="stat-value">{stats.total}</div>
        <div className="stat-label">Total</div>
      </div>
      <div className="stat-card military">
        <div className="stat-value">{stats.military}</div>
        <div className="stat-label">Military</div>
      </div>
      <div className="stat-card emergency">
        <div className="stat-value">{stats.emergency}</div>
        <div className="stat-label">Emergency</div>
      </div>
      <div className="stat-card">
        <div className="stat-value">{stats.avgAltitude.toLocaleString()}</div>
        <div className="stat-label">Avg Alt (ft)</div>
      </div>
    </div>
  );
}
```

---

## ConnectionStatus

Create `src/components/ConnectionStatus.tsx`:

```tsx
interface ConnectionStatusProps {
  connected: boolean;
  error: string | null;
  count: number;
}

export function ConnectionStatus({ connected, error, count }: ConnectionStatusProps) {
  return (
    <div className={`connection-status ${connected ? 'connected' : 'disconnected'}`}>
      <span className="status-dot" />
      <span>{connected ? `Connected • ${count} aircraft` : error || 'Disconnected'}</span>
    </div>
  );
}
```
