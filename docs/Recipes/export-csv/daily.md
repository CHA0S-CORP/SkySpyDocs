---
title: "Daily Rotation"
slug: "export-csv/daily"
excerpt: "Rotate log files daily."
hidden: false
---

Create `daily_logger.py`:

```python
#!/usr/bin/env python3
"""SkySpy Daily CSV Logger - Rotates logs by date."""

import csv
import json
import os
from datetime import datetime, date
from pathlib import Path

import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
OUTPUT_DIR = Path("logs")

FIELDS = [
    "timestamp", "hex", "flight", "type", "lat", "lon",
    "alt", "gs", "track", "vr", "squawk", "distance",
    "military", "emergency", "category"
]


class DailyCSVLogger:
    def __init__(self, base_dir: Path):
        self.base_dir = base_dir
        self.base_dir.mkdir(exist_ok=True)
        self.current_date = None
        self.writer = None
        self.file_handle = None

    def get_file_path(self, d: date) -> Path:
        return self.base_dir / f"aircraft_{d.isoformat()}.csv"

    def rotate_if_needed(self):
        today = date.today()
        if today != self.current_date:
            if self.file_handle:
                self.file_handle.close()

            file_path = self.get_file_path(today)
            file_exists = file_path.exists()

            self.file_handle = open(file_path, "a", newline="")
            self.writer = csv.DictWriter(self.file_handle, fieldnames=FIELDS)

            if not file_exists:
                self.writer.writeheader()
                print(f"📁 Created new log file: {file_path}")

            self.current_date = today

    def log(self, aircraft: dict):
        self.rotate_if_needed()
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
            "category": aircraft.get("category", ""),
        }
        self.writer.writerow(row)
        self.file_handle.flush()

    def close(self):
        if self.file_handle:
            self.file_handle.close()


def main():
    print(f"📝 SkySpy Daily CSV Logger")
    print(f"📁 Output directory: {OUTPUT_DIR}")

    logger = DailyCSVLogger(OUTPUT_DIR)
    logged_this_minute = set()
    last_minute = None
    count = 0

    try:
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

                            logger.log(aircraft)
                            logged_this_minute.add(icao)
                            count += 1

                    except json.JSONDecodeError:
                        continue

            except requests.exceptions.ConnectionError:
                print("❌ Connection lost, reconnecting...")
                import time
                time.sleep(5)

    except KeyboardInterrupt:
        print(f"\n✅ Logged {count} total records")
    finally:
        logger.close()


if __name__ == "__main__":
    main()
```
