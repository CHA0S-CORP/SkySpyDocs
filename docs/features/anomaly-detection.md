---
title: Anomaly Detection
hidden: false
---

# 🛰️ Flight-Pattern Anomaly Detection

> Classify flown tracks by shape — orbits, holds, survey grids, multi-orbit surveillance patterns — and rank them by how unusual they are.

---

## 📖 Overview

Beyond the per-track orbit flag, SkySpy v3 adds a **geometry classifier** that reconstructs an aircraft's flown path and labels its shape. It catches patterns a simple displacement check misses — most notably the **multi-orbit survey** shape (several tight orbits joined by long transit legs) typical of aerial-survey and surveillance work.

Detection is pure geometry (no ML, no external calls) and is surfaced through the [AI Assistant](./ai-assistant) via the `detect_unusual_patterns` and `aircraft_track` tools.

![Flown-track view](https://raw.githubusercontent.com/CHA0S-CORP/SkySpy/main/docs/screenshots/desktop/airframe-track-tab.png)

---

## 🧭 Patterns

`analyze_track()` returns one label with a **tone** (urgency) and an **unusualness score** (higher = stranger):

| Pattern | Tone | Shape |
|:--------|:-----|:------|
| `multi_orbit_survey` | 🔴 danger | 3+ orbit clusters joined by transit legs — survey/surveillance |
| `repositioned_orbit` | 🟠 warn | Two distinct orbit locations with transit between |
| `orbit_loiter` | 🟠 warn | Single holding pattern / loiter |
| `orbit_then_transit` | 🟠 warn | Orbit followed by departure |
| `grid_or_zigzag` | 🟠 warn | Survey grid or zig-zag (4+ heading reversals) |
| `circling` | 🟠 warn | Circular holding pattern |
| `meandering` | 🔵 info | Winding, non-direct path |
| `transit` | 🟢 ok | Direct point-to-point cruise |
| `stationary` | 🟢 ok | No meaningful movement (path < 2nm) |
| `insufficient_data` | 🟢 ok | Fewer than 4 samples |

---

## 📐 Metrics

Each analysis reports the geometry it measured, so a result is explainable and map-able:

| Metric | Meaning |
|:-------|:--------|
| `score` | Unusualness: `3.0·clusters + 1.0·revolutions + 0.5·reversals + tortuosity` (capped per factor) |
| `is_unusual` | True unless `transit` / `stationary` / `insufficient_data` |
| `loiter_count` / `loiter_clusters` | Orbit clusters (each: lat, lon, points, path_nm, revolutions) |
| `revolutions` | Total turning as multiples of 360° |
| `reversals` | Heading-direction flips |
| `tortuosity` | Path length ÷ net displacement (higher = more winding) |
| `path_nm` / `net_displacement_nm` | Distance flown vs straight-line start→end |
| `point_count` | Position samples analyzed |

<details>
<summary><strong>Detection parameters</strong></summary>

| Parameter | Value | Purpose |
|:----------|:------|:--------|
| `LOITER_RADIUS_NM` | 4.0 | Max radius to cluster positions as the same "place" |
| `LOITER_PATH_FACTOR` | 2.0 | Path inside a cluster must be ≥2× cluster radius to count as an orbit |
| `LOITER_MIN_POINTS` | 4 | Minimum positions inside a cluster |
| `MIN_PATH_NM` | 2.0 | Minimum track length to classify |
| `MIN_ORBIT_SPREAD_NM` | 0.4 | Orbit must swing ≥0.4nm from center (kills parked-GPS jitter) |
| `MIN_ORBIT_REVOLUTIONS` | 1.0 | Orbit must loop ≥360° |
| `TIME_GAP_SPLIT_S` | 900 | Split separate flights on 15-min gaps (landing + refly) |

</details>

---

## 🔎 Scanning

`scan_unusual_patterns(hours=6, limit=15)` loads the most position-rich aircraft in the window (capped at 150), splits each on time gaps, analyzes every leg, keeps the unusual ones, and returns the top *N* ranked by score:

```json
{
  "hours": 6, "scanned": 145, "count": 8,
  "results": [{
    "icao_hex": "a1b2c3", "callsign": "N12345", "aircraft_type": "Cessna 172",
    "center": {"lat": 34.0521, "lon": -118.2437},
    "pattern": "multi_orbit_survey", "tone": "danger", "score": 12.4,
    "loiter_count": 3, "revolutions": 6.2, "reversals": 18,
    "tortuosity": 2.8, "path_nm": 45.3, "net_displacement_nm": 12.1, "point_count": 102
  }]
}
```

---

## 💬 Using It

Anomaly detection is **assistant-only** — there is no dedicated REST endpoint or WebSocket event. Ask the [AI Assistant](./ai-assistant):

> *"Is anyone flying a strange or suspicious pattern right now?"*
> *"Is anything surveying or orbiting the area?"*
> *"Explain the odd shape near the harbor on the map."*

The agent calls `detect_unusual_patterns` to find hits, then `aircraft_track` to drill into a specific airframe's polyline and behavior flags (`orbit_or_loiter`, `rapid_climb`, `rapid_descent`). Each hit carries a map-able center so the answer can render on a map.

---

## 📚 Related

- [AI Assistant](./ai-assistant) — the interface for anomaly queries
- [Cannonball Mode](./cannonball-mode) — mobile-threat detection near the receiver
- [Safety Monitoring & Alerts](./safety-alerts)
