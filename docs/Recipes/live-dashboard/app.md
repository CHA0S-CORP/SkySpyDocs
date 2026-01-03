---
title: "Main App"
slug: "live-dashboard/app"
excerpt: "Wire together the dashboard components."
hidden: false
---

Replace `src/App.tsx`:

```tsx
import { useCallback, useState } from 'react';
import { useSkySpySSE, SafetyEvent } from './hooks/useSkySpySSE';
import { AircraftList } from './components/AircraftList';
import { Statistics } from './components/Statistics';
import { ConnectionStatus } from './components/ConnectionStatus';
import './App.css';

function App() {
  const [alerts, setAlerts] = useState<SafetyEvent[]>([]);

  const handleSafetyEvent = useCallback((event: SafetyEvent) => {
    setAlerts(prev => [event, ...prev].slice(0, 10));
  }, []);

  const { aircraft, connected, error, count } = useSkySpySSE({
    url: 'http://localhost:5000/api/v1/map/sse?replay_history=true',
    onSafetyEvent: handleSafetyEvent
  });

  return (
    <div className="app">
      <header>
        <h1>✈️ SkySpy Dashboard</h1>
        <ConnectionStatus connected={connected} error={error} count={count} />
      </header>

      <main>
        <Statistics aircraft={aircraft} />

        <div className="content">
          <AircraftList aircraft={aircraft} />

          <div className="alerts-panel">
            <h3>Safety Alerts</h3>
            {alerts.length === 0 ? (
              <p className="no-alerts">No recent alerts</p>
            ) : (
              <ul className="alerts-list">
                {alerts.map((alert, i) => (
                  <li key={i} className={`alert ${alert.severity}`}>
                    <span className="alert-type">{alert.event_type}</span>
                    <span className="alert-message">{alert.message}</span>
                  </li>
                ))}
              </ul>
            )}
          </div>
        </div>
      </main>
    </div>
  );
}

export default App;
```
