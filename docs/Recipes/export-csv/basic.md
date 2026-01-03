---
title: "Basic Logger"
slug: "export-csv/basic"
excerpt: "Simple CSV logging script."
hidden: false
---

Create `csv_logger.py`:

```python
#!/usr/bin/env python3
"""SkySpy CSV Logger - Logs all aircraft sightings to CSV."""

import csv
import json
import os
from datetime import datetime
from pathlib import Path

import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
OUTPUT_DIR = Path("logs")
OUTPUT_FILE = OUTPUT_DIR / "aircraft.csv"

FIELDS = [
    "timestamp", "hex", "flight", "type", "lat", "lon",
    "alt", "gs", "track", "vr", "squawk", "distance",
    "military", "emergency",
]


def setup_csv():
    OUTPUT_DIR.mkdir(exist_ok=True)
    if not OUTPUT_FILE.exists():
        with open(OUTPUT_FILE, "w", newline="") as f:
            writer = csv.DictWriter(f, fieldnames=FIELDS)
            writer.writeheader()
        print(f"Created {OUTPUT_FILE}")


def log_aircraft(aircraft: dict):
    row = {
        "timestamp": datetime.now().isoformat(),
        "hex": aircraft.get("hex", ""),
        "flight": aircraft.get("flight", "").strip(),
        "type": aircraft.get("type", ""),
        "lat": aircraft.get("lat", ""),
        "lon": aircraft.get("lon", ""),
        "alt": aircraft.get("alt", ""),
        "gs": aircraft.get("gs", ""),
        "track": aircraft.get("track", ""),
        "vr": aircraft.get("vr", ""),
        "squawk": aircraft.get("squawk", ""),
        "distance": round(aircraft.get("distance", 0), 2),
        "military": aircraft.get("military", False),
        "emergency": aircraft.get("emergency", False),
    }

    with open(OUTPUT_FILE, "a", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=FIELDS)
        writer.writerow(row)


def main():
    print(f"📝 SkySpy CSV Logger")
    print(f"📡 Connecting to {SKYSPY_URL}")
    print(f"📁 Output: {OUTPUT_FILE}")

    setup_csv()
    logged_this_minute = set()
    last_minute = None
    count = 0

    while True:
        try:
            url = f"{SKYSPY_URL}/api/v1/map/sse"
            response = requests.get(url, stream=True)
            client = sseclient.SSEClient(response)

            print("✅ Connected, logging aircraft...")

            for event in client.events():
                try:
                    if event.event not in ["aircraft_update", "aircraft_new"]:
                        continue

                    data = json.loads(event.data)
                    current_minute = datetime.now().replace(second=0, microsecond=0)

                    if current_minute != last_minute:
                        logged_this_minute.clear()
                        last_minute = current_minute

                    for aircraft in data.get("aircraft", []):
                        icao = aircraft.get("hex")
                        if icao in logged_this_minute:
                            continue

                        log_aircraft(aircraft)
                        logged_this_minute.add(icao)
                        count += 1

                        if count % 100 == 0:
                            print(f"📊 Logged {count} records")

                except json.JSONDecodeError:
                    continue

        except requests.exceptions.ConnectionError:
            print("❌ Connection lost, reconnecting...")
            import time
            time.sleep(5)
        except KeyboardInterrupt:
            print(f"\n✅ Logged {count} total records to {OUTPUT_FILE}")
            break


if __name__ == "__main__":
    main()
```

## Run

```bash
pip install sseclient-py requests
python csv_logger.py
```
