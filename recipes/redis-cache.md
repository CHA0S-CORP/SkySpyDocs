---
title: Redis Cache
description: Cache aircraft data in Redis for fast lookups.
hidden: false
recipe:
  color: '#DC382D'
  icon: ⚡
---
```shell Shell
redis-cli SET aircraft:A12345 '{"flight":"RCH419","alt":35000}' EX 300
redis-cli GET aircraft:A12345
```

```go Go
package main

import (
	"context"
	"encoding/json"
	"time"

	"github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
	rdb := redis.NewClient(&redis.Options{Addr: "localhost:6379"})

	aircraft := map[string]interface{}{
		"hex": "A12345", "flight": "RCH419", "alt": 35000, "type": "C-17",
	}
	data, _ := json.Marshal(aircraft)

	rdb.Set(ctx, "aircraft:A12345", data, 5*time.Minute)

	val, _ := rdb.Get(ctx, "aircraft:A12345").Result()
	var cached map[string]interface{}
	json.Unmarshal([]byte(val), &cached)
}
```

```python Python
import json
import redis

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

def cache_aircraft(aircraft, ttl=300):
    key = f"aircraft:{aircraft['hex']}"
    r.setex(key, ttl, json.dumps(aircraft))

def get_aircraft(hex_code):
    data = r.get(f"aircraft:{hex_code}")
    return json.loads(data) if data else None

def get_all_aircraft():
    keys = r.keys("aircraft:*")
    return [json.loads(r.get(k)) for k in keys]

# Example usage
cache_aircraft({"hex": "A12345", "flight": "RCH419", "alt": 35000})
print(get_aircraft("A12345"))
```

```javascript JavaScript
const Redis = require('ioredis');

const redis = new Redis();

async function cacheAircraft(aircraft, ttl = 300) {
    const key = `aircraft:${aircraft.hex}`;
    await redis.setex(key, ttl, JSON.stringify(aircraft));
}

async function getAircraft(hexCode) {
    const data = await redis.get(`aircraft:${hexCode}`);
    return data ? JSON.parse(data) : null;
}

async function getAllAircraft() {
    const keys = await redis.keys('aircraft:*');
    const values = await Promise.all(keys.map(k => redis.get(k)));
    return values.map(v => JSON.parse(v));
}

// Example usage
await cacheAircraft({ hex: 'A12345', flight: 'RCH419', alt: 35000 });
console.log(await getAircraft('A12345'));
```

```json Response Example
{"hex": "A12345", "flight": "RCH419", "alt": 35000, "type": "C-17"}
```

# Connect to Redis

<!-- shell@1 -->
<!-- go@1-14 -->
<!-- python@1-4 -->
<!-- javascript@1-3 -->

Connect to your Redis server. Default is localhost:6379. Use Redis for sub-millisecond lookups.

# Cache Aircraft Data

<!-- shell@1 -->
<!-- go@16-21 -->
<!-- python@6-8 -->
<!-- javascript@5-8 -->

Store aircraft with hex code as key. Set TTL (5 minutes) to auto-expire stale data.

# Retrieve Cached Data

<!-- shell@2 -->
<!-- go@23-26 -->
<!-- python@10-15 -->
<!-- javascript@10-18 -->

Get individual aircraft by hex or scan all cached aircraft. Redis KEYS pattern matching finds all aircraft.
