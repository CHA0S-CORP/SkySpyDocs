---
title: Redis Cache
excerpt: Real-time aircraft cache with Redis for fast lookups.
hidden: false
recipe:
  color: '#DC382D'
  icon: ⚡
difficulty: intermediate
tags: [data, redis, cache, real-time]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Redis server running (local or cloud)
- Python: `pip install sseclient-py requests redis`

## What You'll Build

A real-time aircraft cache using Redis for sub-millisecond lookups, pub/sub notifications, and ephemeral storage with automatic expiration.

```shell Shell
pip install sseclient-py requests redis
export REDIS_URL="redis://localhost:6379"
python redis_cache.py
```

```python Python
import json
import os
import sys
import time
import requests
import sseclient
import redis

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379")
KEY_PREFIX = os.getenv("KEY_PREFIX", "skyspy")
TTL_SECONDS = int(os.getenv("TTL_SECONDS", "300"))  # 5 minutes

r = redis.from_url(REDIS_URL, decode_responses=True)


def update_aircraft(aircraft):
    """Update aircraft in Redis with expiring keys."""
    hex_code = aircraft.get("hex")
    if not hex_code:
        return

    # Main aircraft data
    key = f"{KEY_PREFIX}:aircraft:{hex_code}"
    r.hset(key, mapping={
        "hex": hex_code,
        "flight": aircraft.get("flight", "").strip(),
        "type": aircraft.get("type", ""),
        "altitude": str(aircraft.get("alt", 0)),
        "speed": str(aircraft.get("gs", 0)),
        "track": str(aircraft.get("track", 0)),
        "lat": str(aircraft.get("lat", 0)),
        "lon": str(aircraft.get("lon", 0)),
        "distance": str(aircraft.get("distance", 0)),
        "military": str(aircraft.get("military", False)),
        "squawk": aircraft.get("squawk", ""),
        "updated": str(int(time.time())),
    })
    r.expire(key, TTL_SECONDS)

    # Add to active set
    r.sadd(f"{KEY_PREFIX}:active", hex_code)

    # Geo index for spatial queries
    if aircraft.get("lat") and aircraft.get("lon"):
        r.geoadd(
            f"{KEY_PREFIX}:positions",
            (aircraft["lon"], aircraft["lat"], hex_code)
        )

    # Sorted set by distance
    if aircraft.get("distance"):
        r.zadd(f"{KEY_PREFIX}:by_distance", {hex_code: aircraft["distance"]})

    # Military set
    if aircraft.get("military"):
        r.sadd(f"{KEY_PREFIX}:military", hex_code)
        r.expire(f"{KEY_PREFIX}:military", TTL_SECONDS)

    # Publish update for subscribers
    r.publish(f"{KEY_PREFIX}:updates", json.dumps({
        "hex": hex_code,
        "flight": aircraft.get("flight", "").strip(),
        "lat": aircraft.get("lat"),
        "lon": aircraft.get("lon"),
        "altitude": aircraft.get("alt"),
    }))


def cleanup_expired():
    """Remove expired aircraft from sets."""
    active = r.smembers(f"{KEY_PREFIX}:active")
    for hex_code in active:
        if not r.exists(f"{KEY_PREFIX}:aircraft:{hex_code}"):
            r.srem(f"{KEY_PREFIX}:active", hex_code)
            r.srem(f"{KEY_PREFIX}:military", hex_code)
            r.zrem(f"{KEY_PREFIX}:by_distance", hex_code)
            r.zrem(f"{KEY_PREFIX}:positions", hex_code)


def main():
    print(f"Redis Cache connected to {SKYSPY_URL}")
    print(f"Caching to {REDIS_URL} with prefix '{KEY_PREFIX}'")

    last_cleanup = time.time()
    aircraft_count = 0

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)

                    for aircraft in data.get("aircraft", []):
                        update_aircraft(aircraft)
                        aircraft_count += 1

                    # Periodic cleanup
                    if time.time() - last_cleanup > 60:
                        cleanup_expired()
                        active = r.scard(f"{KEY_PREFIX}:active")
                        print(f"Active aircraft: {active}, Total updates: {aircraft_count}")
                        last_cleanup = time.time()

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```go Go
package main

import (
	"bufio"
	"context"
	"encoding/json"
	"fmt"
	"net/http"
	"os"
	"strconv"
	"strings"
	"time"

	"github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	redisURL := os.Getenv("REDIS_URL")
	if redisURL == "" {
		redisURL = "redis://localhost:6379"
	}

	prefix := os.Getenv("KEY_PREFIX")
	if prefix == "" {
		prefix = "skyspy"
	}

	ttl := 300 * time.Second

	opt, _ := redis.ParseURL(redisURL)
	rdb := redis.NewClient(opt)

	fmt.Printf("Redis Cache connected to %s\n", skyspyURL)
	fmt.Printf("Caching to %s with prefix '%s'\n", redisURL, prefix)

	for {
		resp, err := http.Get(skyspyURL + "/api/v1/map/sse")
		if err != nil {
			time.Sleep(5 * time.Second)
			continue
		}

		scanner := bufio.NewScanner(resp.Body)
		for scanner.Scan() {
			line := scanner.Text()
			if strings.HasPrefix(line, "data:") {
				var data map[string]interface{}
				json.Unmarshal([]byte(line[5:]), &data)

				if aircraft, ok := data["aircraft"].([]interface{}); ok {
					for _, a := range aircraft {
						ac := a.(map[string]interface{})
						hex := fmt.Sprint(ac["hex"])

						key := fmt.Sprintf("%s:aircraft:%s", prefix, hex)

						fields := map[string]interface{}{
							"hex":     hex,
							"updated": time.Now().Unix(),
						}

						if v, ok := ac["flight"].(string); ok {
							fields["flight"] = strings.TrimSpace(v)
						}
						if v, ok := ac["type"].(string); ok {
							fields["type"] = v
						}
						if v, ok := ac["alt"].(float64); ok {
							fields["altitude"] = strconv.FormatFloat(v, 'f', 0, 64)
						}
						if v, ok := ac["gs"].(float64); ok {
							fields["speed"] = strconv.FormatFloat(v, 'f', 0, 64)
						}
						if v, ok := ac["lat"].(float64); ok {
							fields["lat"] = strconv.FormatFloat(v, 'f', 6, 64)
						}
						if v, ok := ac["lon"].(float64); ok {
							fields["lon"] = strconv.FormatFloat(v, 'f', 6, 64)
						}

						rdb.HSet(ctx, key, fields)
						rdb.Expire(ctx, key, ttl)
						rdb.SAdd(ctx, prefix+":active", hex)

						if lat, ok := ac["lat"].(float64); ok {
							if lon, ok := ac["lon"].(float64); ok {
								rdb.GeoAdd(ctx, prefix+":positions", &redis.GeoLocation{
									Name:      hex,
									Longitude: lon,
									Latitude:  lat,
								})
							}
						}

						if ac["military"] == true {
							rdb.SAdd(ctx, prefix+":military", hex)
						}
					}
				}
			}
		}
		resp.Body.Close()
	}
}
```

```javascript JavaScript
const EventSource = require('eventsource');
const Redis = require('ioredis');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const REDIS_URL = process.env.REDIS_URL || 'redis://localhost:6379';
const KEY_PREFIX = process.env.KEY_PREFIX || 'skyspy';
const TTL_SECONDS = parseInt(process.env.TTL_SECONDS || '300');

const redis = new Redis(REDIS_URL);

console.log(`Redis Cache connected to ${SKYSPY_URL}`);
console.log(`Caching to ${REDIS_URL} with prefix '${KEY_PREFIX}'`);

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    const hex = aircraft.hex;
    const key = `${KEY_PREFIX}:aircraft:${hex}`;

    // Store aircraft data
    await redis.hset(key, {
      hex,
      flight: (aircraft.flight || '').trim(),
      type: aircraft.type || '',
      altitude: String(aircraft.alt || 0),
      speed: String(aircraft.gs || 0),
      lat: String(aircraft.lat || 0),
      lon: String(aircraft.lon || 0),
      distance: String(aircraft.distance || 0),
      military: String(aircraft.military || false),
      updated: String(Math.floor(Date.now() / 1000))
    });
    await redis.expire(key, TTL_SECONDS);

    // Active set
    await redis.sadd(`${KEY_PREFIX}:active`, hex);

    // Geo index
    if (aircraft.lat && aircraft.lon) {
      await redis.geoadd(`${KEY_PREFIX}:positions`, aircraft.lon, aircraft.lat, hex);
    }

    // Distance sorted set
    if (aircraft.distance) {
      await redis.zadd(`${KEY_PREFIX}:by_distance`, aircraft.distance, hex);
    }

    // Military set
    if (aircraft.military) {
      await redis.sadd(`${KEY_PREFIX}:military`, hex);
    }

    // Publish update
    await redis.publish(`${KEY_PREFIX}:updates`, JSON.stringify({
      hex,
      flight: (aircraft.flight || '').trim(),
      lat: aircraft.lat,
      lon: aircraft.lon
    }));
  }
});

es.onerror = (err) => console.error('SSE error:', err);
```

## Redis Data Structures

### Hash: Aircraft Details
```
skyspy:aircraft:A12345
  hex: A12345
  flight: UAL123
  type: B738
  altitude: 35000
  speed: 450
  lat: 40.7128
  lon: -74.006
  military: false
  updated: 1705312345
```

### Set: Active Aircraft
```
skyspy:active -> {A12345, A54321, AE1234, ...}
```

### Set: Military Aircraft
```
skyspy:military -> {AE1234, AE5678, ...}
```

### Sorted Set: By Distance
```
skyspy:by_distance
  A12345: 5.2
  A54321: 12.8
  AE1234: 23.1
```

### Geo Index: Positions
```
skyspy:positions -> GEO{A12345: (lat, lon), ...}
```

## Query Examples

### Get Aircraft by Hex

```shell
redis-cli HGETALL skyspy:aircraft:A12345
```

```python
aircraft = r.hgetall("skyspy:aircraft:A12345")
print(f"Flight: {aircraft['flight']}, Alt: {aircraft['altitude']}")
```

### Get All Active Aircraft

```python
active = r.smembers("skyspy:active")
for hex_code in active:
    data = r.hgetall(f"skyspy:aircraft:{hex_code}")
    print(f"{hex_code}: {data.get('flight', 'Unknown')}")
```

### Get Closest Aircraft

```python
# Top 5 closest
closest = r.zrange("skyspy:by_distance", 0, 4, withscores=True)
for hex_code, distance in closest:
    print(f"{hex_code}: {distance:.1f} nm")
```

### Find Aircraft Within Radius

```python
# Within 10 miles of a point
nearby = r.georadius(
    "skyspy:positions",
    -74.006,  # longitude
    40.7128,  # latitude
    10,       # radius
    unit="mi",
    withdist=True
)
for hex_code, dist in nearby:
    print(f"{hex_code}: {dist:.1f} miles")
```

### Subscribe to Updates

```python
pubsub = r.pubsub()
pubsub.subscribe("skyspy:updates")

for message in pubsub.listen():
    if message["type"] == "message":
        data = json.loads(message["data"])
        print(f"Update: {data['flight']} at {data['lat']}, {data['lon']}")
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `REDIS_URL` | Redis connection URL | `redis://localhost:6379` |
| `KEY_PREFIX` | Prefix for all Redis keys | `skyspy` |
| `TTL_SECONDS` | Key expiration time | `300` (5 min) |

## Redis Setup

### Docker Quick Start

```shell
docker run -d --name redis \
  -p 6379:6379 \
  redis:7-alpine
```

### With Persistence

```shell
docker run -d --name redis \
  -p 6379:6379 \
  -v redis-data:/data \
  redis:7-alpine \
  redis-server --appendonly yes
```

## API Integration

Build a fast API on top of Redis:

```python
from flask import Flask, jsonify
import redis

app = Flask(__name__)
r = redis.from_url("redis://localhost:6379", decode_responses=True)

@app.route("/api/aircraft")
def get_all_aircraft():
    active = r.smembers("skyspy:active")
    aircraft = []
    for hex_code in active:
        data = r.hgetall(f"skyspy:aircraft:{hex_code}")
        if data:
            aircraft.append(data)
    return jsonify(aircraft)

@app.route("/api/aircraft/<hex_code>")
def get_aircraft(hex_code):
    data = r.hgetall(f"skyspy:aircraft:{hex_code}")
    if not data:
        return jsonify({"error": "Not found"}), 404
    return jsonify(data)

@app.route("/api/aircraft/nearby/<float:lat>/<float:lon>/<float:radius>")
def get_nearby(lat, lon, radius):
    nearby = r.georadius("skyspy:positions", lon, lat, radius, unit="nm", withdist=True)
    return jsonify([{"hex": h, "distance": d} for h, d in nearby])

@app.route("/api/military")
def get_military():
    military = r.smembers("skyspy:military")
    return jsonify(list(military))
```

## Testing & Verification

1. **Start Redis**:
   ```shell
   docker run -d -p 6379:6379 redis:7-alpine
   ```
2. **Run the cache script**
3. **Check data**:
   ```shell
   redis-cli KEYS "skyspy:*"
   redis-cli SCARD skyspy:active
   redis-cli HGETALL skyspy:aircraft:A12345
   ```
4. **Test geo queries**:
   ```shell
   redis-cli GEORADIUS skyspy:positions -74.006 40.7128 50 mi COUNT 10
   ```

## Troubleshooting

### No Data in Redis

**Problem:** Keys not appearing
**Solution:**
- Check Redis connection
- Verify aircraft have positions
- Check for Python/client errors

### Keys Expiring Too Fast

**Problem:** Aircraft disappearing quickly
**Solution:**
- Increase `TTL_SECONDS`
- Check cleanup routine frequency
- Verify updates are refreshing TTL

### Memory Usage Growing

**Problem:** Redis using too much memory
**Solution:**
- Enable key expiration
- Run cleanup routine more frequently
- Set `maxmemory` and eviction policy

## Related Recipes

- [Live Dashboard](/docs/live-dashboard) - Use Redis for real-time updates
- [SQLite Local DB](/docs/sqlite-db) - Persistent storage
- [MQTT Publisher](/docs/mqtt-publisher) - Pub/sub alternative
- [Prometheus Metrics](/docs/prometheus-metrics) - Metrics from cache
