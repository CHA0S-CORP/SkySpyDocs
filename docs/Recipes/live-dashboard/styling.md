---
title: "Styling"
slug: "live-dashboard/styling"
excerpt: "Dark theme CSS for the dashboard."
hidden: false
---

Replace `src/App.css`:

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #0f172a;
  color: #e2e8f0;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background: #1e293b;
  border-bottom: 1px solid #334155;
}

/* Connection Status */
.connection-status {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  border-radius: 9999px;
  font-size: 0.875rem;
}

.connection-status.connected {
  background: rgba(34, 197, 94, 0.1);
  color: #22c55e;
}

.connection-status.disconnected {
  background: rgba(239, 68, 68, 0.1);
  color: #ef4444;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: currentColor;
}

/* Statistics */
.statistics {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
  padding: 1.5rem 2rem;
  background: #1e293b;
}

.stat-card {
  background: #334155;
  padding: 1rem;
  border-radius: 0.5rem;
  text-align: center;
}

.stat-card.military { border-left: 3px solid #6366f1; }
.stat-card.emergency { border-left: 3px solid #ef4444; }

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: #fff;
}

.stat-label {
  font-size: 0.75rem;
  color: #94a3b8;
  text-transform: uppercase;
}

/* Content Layout */
.content {
  display: grid;
  grid-template-columns: 1fr 300px;
  gap: 1rem;
  padding: 1rem 2rem;
  flex: 1;
}

/* Aircraft List */
.aircraft-list {
  background: #1e293b;
  border-radius: 0.5rem;
  overflow: hidden;
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  border-bottom: 1px solid #334155;
}

.list-header input {
  background: #334155;
  border: 1px solid #475569;
  color: #e2e8f0;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 0.75rem 1rem;
  text-align: left;
  border-bottom: 1px solid #334155;
}

th {
  background: #334155;
  font-weight: 600;
  font-size: 0.75rem;
  text-transform: uppercase;
  color: #94a3b8;
}

tr:hover { background: #334155; }
tr.emergency { background: rgba(239, 68, 68, 0.1); }

/* Alerts Panel */
.alerts-panel {
  background: #1e293b;
  border-radius: 0.5rem;
  padding: 1rem;
}

.alerts-panel h3 {
  margin-bottom: 1rem;
  font-size: 0.875rem;
  text-transform: uppercase;
  color: #94a3b8;
}

.no-alerts {
  color: #64748b;
  text-align: center;
  padding: 2rem;
}

.alerts-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.alert {
  padding: 0.75rem;
  border-radius: 0.25rem;
  font-size: 0.875rem;
}

.alert.warning {
  background: rgba(250, 204, 21, 0.1);
  border-left: 3px solid #facc15;
}

.alert.critical {
  background: rgba(239, 68, 68, 0.1);
  border-left: 3px solid #ef4444;
}
```
