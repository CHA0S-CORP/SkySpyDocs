---
title: MongoDB Storage
excerpt: NoSQL aircraft logging with MongoDB
hidden: false
recipe:
  color: '#47A248'
  icon: "🍃"
difficulty: intermediate
tags: [data, mongodb, database, nosql]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- MongoDB 5.0+ running (local, Docker, or cloud)
- Python: `pip install sseclient-py requests pymongo`
- Go: `go get go.mongodb.org/mongo-driver/mongo`
- Node.js: `npm install eventsource mongodb`

## What You'll Build

A flexible NoSQL data pipeline that streams aircraft positions to MongoDB for schema-less storage, powerful aggregation queries, and automatic data expiration with TTL indexes.

```shell Shell
# Install dependencies
pip install sseclient-py requests pymongo

# Set environment variables
export MONGODB_URI="mongodb://localhost:27017"
export MONGODB_DB="skyspy"
export MONGODB_COLLECTION="aircraft_positions"
export SKYSPY_URL="http://localhost:5000"

# Run the logger
python mongodb_logger.py
```

```python Python
import json
import os
import sys
import time
from datetime import datetime
import requests
import sseclient
from pymongo import MongoClient, ASCENDING, DESCENDING
from pymongo.errors import ConnectionFailure

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
MONGODB_URI = os.getenv("MONGODB_URI", "mongodb://localhost:27017")
MONGODB_DB = os.getenv("MONGODB_DB", "skyspy")
MONGODB_COLLECTION = os.getenv("MONGODB_COLLECTION", "aircraft_positions")
LOG_INTERVAL = int(os.getenv("LOG_INTERVAL", "60"))
TTL_DAYS = int(os.getenv("TTL_DAYS", "30"))

# Rate limiting per aircraft
last_logged = {}


def get_connection():
    """Create MongoDB connection."""
    client = MongoClient(MONGODB_URI)
    try:
        client.admin.command('ping')
        return client
    except ConnectionFailure:
        print("Failed to connect to MongoDB")
        sys.exit(1)


def init_database(db):
    """Initialize database schema and indexes."""
    collection = db[MONGODB_COLLECTION]

    # Create indexes for common queries
    collection.create_index([("hex", ASCENDING)])
    collection.create_index([("timestamp", DESCENDING)])
    collection.create_index([("military", ASCENDING)])
    collection.create_index([("flight", ASCENDING)])
    collection.create_index([("hex", ASCENDING), ("timestamp", DESCENDING)])
    collection.create_index([("location", "2dsphere")])

    # TTL index for automatic cleanup
    collection.create_index(
        [("timestamp", ASCENDING)],
        expireAfterSeconds=TTL_DAYS * 24 * 60 * 60,
        name="ttl_cleanup"
    )

    print(f"Database initialized with TTL of {TTL_DAYS} days")


def log_aircraft(collection, aircraft):
    """Log aircraft position to database."""
    hex_code = aircraft.get("hex")
    now = time.time()

    # Rate limit per aircraft
    if hex_code in last_logged:
        if now - last_logged[hex_code] < LOG_INTERVAL:
            return False

    last_logged[hex_code] = now

    # Build document with GeoJSON location
    doc = {
        "timestamp": datetime.utcnow(),
        "hex": hex_code,
        "flight": aircraft.get("flight", "").strip() or None,
        "aircraft_type": aircraft.get("type"),
        "altitude": aircraft.get("alt"),
        "speed": aircraft.get("gs"),
        "track": aircraft.get("track"),
        "distance": aircraft.get("distance"),
        "military": aircraft.get("military", False),
        "squawk": aircraft.get("squawk"),
        "category": aircraft.get("category"),
        "vertical_rate": aircraft.get("baro_rate"),
    }

    # Add GeoJSON location if coordinates available
    if aircraft.get("lat") and aircraft.get("lon"):
        doc["location"] = {
            "type": "Point",
            "coordinates": [aircraft.get("lon"), aircraft.get("lat")]
        }
        doc["lat"] = aircraft.get("lat")
        doc["lon"] = aircraft.get("lon")

    collection.insert_one(doc)
    return True


def main():
    print(f"MongoDB Logger connected to {SKYSPY_URL}")
    print(f"Database: {MONGODB_URI}/{MONGODB_DB}")

    client = get_connection()
    db = client[MONGODB_DB]
    init_database(db)
    collection = db[MONGODB_COLLECTION]

    logged_count = 0

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            sse_client = sseclient.SSEClient(response)

            for event in sse_client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)

                    for aircraft in data.get("aircraft", []):
                        if log_aircraft(collection, aircraft):
                            logged_count += 1
                            print(f"Logged: {aircraft.get('flight', aircraft.get('hex'))} (total: {logged_count})")

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
	"strings"
	"time"

	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

var lastLogged = make(map[string]time.Time)
var logInterval = 60 * time.Second

type AircraftPosition struct {
	Timestamp    time.Time   `bson:"timestamp"`
	Hex          string      `bson:"hex"`
	Flight       *string     `bson:"flight,omitempty"`
	AircraftType *string     `bson:"aircraft_type,omitempty"`
	Altitude     interface{} `bson:"altitude,omitempty"`
	Speed        interface{} `bson:"speed,omitempty"`
	Track        interface{} `bson:"track,omitempty"`
	Lat          interface{} `bson:"lat,omitempty"`
	Lon          interface{} `bson:"lon,omitempty"`
	Location     *GeoJSON    `bson:"location,omitempty"`
	Distance     interface{} `bson:"distance,omitempty"`
	Military     bool        `bson:"military"`
	Squawk       *string     `bson:"squawk,omitempty"`
	Category     *string     `bson:"category,omitempty"`
	VerticalRate interface{} `bson:"vertical_rate,omitempty"`
}

type GeoJSON struct {
	Type        string    `bson:"type"`
	Coordinates []float64 `bson:"coordinates"`
}

func main() {
	ctx := context.Background()

	skyspyURL := os.Getenv("SKYSPY_URL")
	if skyspyURL == "" {
		skyspyURL = "http://localhost:5000"
	}

	mongoURI := os.Getenv("MONGODB_URI")
	if mongoURI == "" {
		mongoURI = "mongodb://localhost:27017"
	}
	mongoDB := os.Getenv("MONGODB_DB")
	if mongoDB == "" {
		mongoDB = "skyspy"
	}
	mongoCollection := os.Getenv("MONGODB_COLLECTION")
	if mongoCollection == "" {
		mongoCollection = "aircraft_positions"
	}

	client, err := mongo.Connect(ctx, options.Client().ApplyURI(mongoURI))
	if err != nil {
		fmt.Printf("Failed to connect to MongoDB: %v\n", err)
		os.Exit(1)
	}
	defer client.Disconnect(ctx)

	// Ping to verify connection
	if err := client.Ping(ctx, nil); err != nil {
		fmt.Printf("Failed to ping MongoDB: %v\n", err)
		os.Exit(1)
	}

	collection := client.Database(mongoDB).Collection(mongoCollection)

	// Create indexes
	collection.Indexes().CreateMany(ctx, []mongo.IndexModel{
		{Keys: bson.D{{Key: "hex", Value: 1}}},
		{Keys: bson.D{{Key: "timestamp", Value: -1}}},
		{Keys: bson.D{{Key: "military", Value: 1}}},
		{Keys: bson.D{{Key: "location", Value: "2dsphere"}}},
	})

	// Create TTL index (30 days)
	ttlSeconds := int32(30 * 24 * 60 * 60)
	collection.Indexes().CreateOne(ctx, mongo.IndexModel{
		Keys:    bson.D{{Key: "timestamp", Value: 1}},
		Options: options.Index().SetExpireAfterSeconds(ttlSeconds).SetName("ttl_cleanup"),
	})

	fmt.Printf("MongoDB Logger connected to %s\n", skyspyURL)
	fmt.Printf("Database: %s/%s\n", mongoURI, mongoDB)

	loggedCount := 0

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
					now := time.Now()

					for _, a := range aircraft {
						ac := a.(map[string]interface{})
						hex := fmt.Sprint(ac["hex"])

						// Rate limit
						if last, exists := lastLogged[hex]; exists {
							if now.Sub(last) < logInterval {
								continue
							}
						}
						lastLogged[hex] = now

						doc := AircraftPosition{
							Timestamp: now,
							Hex:       hex,
							Military:  ac["military"] == true,
						}

						if flight := strings.TrimSpace(fmt.Sprint(ac["flight"])); flight != "" && flight != "<nil>" {
							doc.Flight = &flight
						}
						if acType := fmt.Sprint(ac["type"]); acType != "<nil>" {
							doc.AircraftType = &acType
						}

						doc.Altitude = ac["alt"]
						doc.Speed = ac["gs"]
						doc.Track = ac["track"]
						doc.Distance = ac["distance"]
						doc.VerticalRate = ac["baro_rate"]

						if lat, ok := ac["lat"].(float64); ok {
							if lon, ok := ac["lon"].(float64); ok {
								doc.Lat = lat
								doc.Lon = lon
								doc.Location = &GeoJSON{
									Type:        "Point",
									Coordinates: []float64{lon, lat},
								}
							}
						}

						collection.InsertOne(ctx, doc)
						loggedCount++
						if loggedCount%100 == 0 {
							fmt.Printf("Total logged: %d\n", loggedCount)
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
const { MongoClient } = require('mongodb');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const MONGODB_URI = process.env.MONGODB_URI || 'mongodb://localhost:27017';
const MONGODB_DB = process.env.MONGODB_DB || 'skyspy';
const MONGODB_COLLECTION = process.env.MONGODB_COLLECTION || 'aircraft_positions';
const LOG_INTERVAL = parseInt(process.env.LOG_INTERVAL || '60') * 1000;
const TTL_DAYS = parseInt(process.env.TTL_DAYS || '30');

const lastLogged = new Map();
let loggedCount = 0;
let collection;

async function initDatabase(db) {
  collection = db.collection(MONGODB_COLLECTION);

  // Create indexes for common queries
  await collection.createIndex({ hex: 1 });
  await collection.createIndex({ timestamp: -1 });
  await collection.createIndex({ military: 1 });
  await collection.createIndex({ flight: 1 });
  await collection.createIndex({ hex: 1, timestamp: -1 });
  await collection.createIndex({ location: '2dsphere' });

  // TTL index for automatic cleanup
  await collection.createIndex(
    { timestamp: 1 },
    { expireAfterSeconds: TTL_DAYS * 24 * 60 * 60, name: 'ttl_cleanup' }
  );

  console.log(`Database initialized with TTL of ${TTL_DAYS} days`);
}

async function logAircraft(aircraft) {
  const hex = aircraft.hex;
  const now = Date.now();

  // Rate limit per aircraft
  if (lastLogged.has(hex) && now - lastLogged.get(hex) < LOG_INTERVAL) {
    return;
  }
  lastLogged.set(hex, now);

  try {
    const doc = {
      timestamp: new Date(),
      hex: hex,
      flight: aircraft.flight?.trim() || null,
      aircraft_type: aircraft.type || null,
      altitude: aircraft.alt || null,
      speed: aircraft.gs || null,
      track: aircraft.track || null,
      distance: aircraft.distance || null,
      military: aircraft.military || false,
      squawk: aircraft.squawk || null,
      category: aircraft.category || null,
      vertical_rate: aircraft.baro_rate || null,
    };

    // Add GeoJSON location if coordinates available
    if (aircraft.lat && aircraft.lon) {
      doc.lat = aircraft.lat;
      doc.lon = aircraft.lon;
      doc.location = {
        type: 'Point',
        coordinates: [aircraft.lon, aircraft.lat]
      };
    }

    await collection.insertOne(doc);

    loggedCount++;
    console.log(`Logged: ${aircraft.flight || hex} (total: ${loggedCount})`);
  } catch (err) {
    console.error('Insert error:', err.message);
  }
}

async function main() {
  const client = new MongoClient(MONGODB_URI);
  await client.connect();

  console.log(`MongoDB Logger connected to ${SKYSPY_URL}`);
  console.log(`Database: ${MONGODB_URI}/${MONGODB_DB}`);

  const db = client.db(MONGODB_DB);
  await initDatabase(db);

  const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

  es.addEventListener('aircraft_update', async (e) => {
    const data = JSON.parse(e.data);
    for (const aircraft of data.aircraft || []) {
      await logAircraft(aircraft);
    }
  });

  es.addEventListener('aircraft_new', async (e) => {
    const data = JSON.parse(e.data);
    for (const aircraft of data.aircraft || []) {
      await logAircraft(aircraft);
    }
  });

  es.onerror = (err) => console.error('SSE error:', err);

  // Graceful shutdown
  process.on('SIGINT', async () => {
    console.log('Shutting down...');
    await client.close();
    process.exit(0);
  });
}

main().catch(console.error);
```

## Document Schema

MongoDB stores aircraft positions as flexible documents with embedded GeoJSON for geospatial queries.

```json
{
  "_id": ObjectId("..."),
  "timestamp": ISODate("2024-01-15T14:30:00Z"),
  "hex": "A12345",
  "flight": "UAL123",
  "aircraft_type": "B738",
  "altitude": 35000,
  "speed": 450,
  "track": 270.5,
  "lat": 40.7128,
  "lon": -74.0060,
  "location": {
    "type": "Point",
    "coordinates": [-74.0060, 40.7128]
  },
  "distance": 15.5,
  "military": false,
  "squawk": "1200",
  "category": "A3",
  "vertical_rate": 0
}
```

| Field | Type | Description |
|-------|------|-------------|
| `_id` | ObjectId | Auto-generated unique identifier |
| `timestamp` | Date | When the position was logged (TTL indexed) |
| `hex` | String | ICAO hex code (indexed) |
| `flight` | String | Callsign/flight number |
| `aircraft_type` | String | Aircraft type code |
| `altitude` | Number | Altitude in feet |
| `speed` | Number | Ground speed in knots |
| `track` | Number | Track angle in degrees |
| `lat` | Number | Latitude |
| `lon` | Number | Longitude |
| `location` | GeoJSON | Point geometry for geospatial queries |
| `distance` | Number | Distance from receiver |
| `military` | Boolean | Military aircraft flag |
| `squawk` | String | Transponder code |
| `category` | String | Aircraft category |
| `vertical_rate` | Number | Climb/descent rate in ft/min |

## Docker Quick Start

```shell
# Start MongoDB with Docker
docker run -d --name skyspy-mongodb \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=skyspy \
  -e MONGO_INITDB_ROOT_PASSWORD=skyspy123 \
  -v skyspy-mongodata:/data/db \
  mongo:7

# Verify connection
docker exec -it skyspy-mongodb mongosh --eval "db.runCommand({ping: 1})"

# Connect with mongosh
docker exec -it skyspy-mongodb mongosh -u skyspy -p skyspy123
```

### Docker Compose

```yaml
version: '3.8'
services:
  mongodb:
    image: mongo:7
    container_name: skyspy-mongodb
    environment:
      MONGO_INITDB_ROOT_USERNAME: skyspy
      MONGO_INITDB_ROOT_PASSWORD: skyspy123
    ports:
      - "27017:27017"
    volumes:
      - mongodata:/data/db
    restart: unless-stopped

  mongo-express:
    image: mongo-express
    container_name: skyspy-mongo-express
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: skyspy
      ME_CONFIG_MONGODB_ADMINPASSWORD: skyspy123
      ME_CONFIG_MONGODB_URL: mongodb://skyspy:skyspy123@mongodb:27017/
    ports:
      - "8081:8081"
    depends_on:
      - mongodb

volumes:
  mongodata:
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `MONGODB_URI` | MongoDB connection string | `mongodb://localhost:27017` |
| `MONGODB_DB` | Database name | `skyspy` |
| `MONGODB_COLLECTION` | Collection name | `aircraft_positions` |
| `LOG_INTERVAL` | Seconds between logs per aircraft | `60` |
| `TTL_DAYS` | Days before automatic document deletion | `30` |

## Aggregation Pipeline Queries

### Unique Aircraft Today

```javascript
db.aircraft_positions.aggregate([
  { $match: { timestamp: { $gte: new Date(new Date().setHours(0,0,0,0)) } } },
  { $group: { _id: "$hex" } },
  { $count: "unique_aircraft" }
])
```

### Most Seen Aircraft (Last 7 Days)

```javascript
db.aircraft_positions.aggregate([
  { $match: { timestamp: { $gte: new Date(Date.now() - 7*24*60*60*1000) } } },
  { $group: {
      _id: { hex: "$hex", flight: "$flight", type: "$aircraft_type" },
      sightings: { $sum: 1 },
      first_seen: { $min: "$timestamp" },
      last_seen: { $max: "$timestamp" }
  }},
  { $sort: { sightings: -1 } },
  { $limit: 20 }
])
```

### Flight Path for Specific Aircraft

```javascript
db.aircraft_positions.aggregate([
  { $match: {
      hex: "A12345",
      timestamp: { $gte: new Date(Date.now() - 24*60*60*1000) }
  }},
  { $project: {
      _id: 0,
      timestamp: 1,
      lat: 1,
      lon: 1,
      altitude: 1,
      speed: 1,
      track: 1
  }},
  { $sort: { timestamp: 1 } }
])
```

### Military Aircraft Activity

```javascript
db.aircraft_positions.aggregate([
  { $match: {
      military: true,
      timestamp: { $gte: new Date(Date.now() - 24*60*60*1000) }
  }},
  { $group: {
      _id: { hex: "$hex", flight: "$flight", type: "$aircraft_type" },
      positions: { $sum: 1 },
      first_seen: { $min: "$timestamp" },
      last_seen: { $max: "$timestamp" }
  }},
  { $sort: { last_seen: -1 } }
])
```

### Hourly Statistics

```javascript
db.aircraft_positions.aggregate([
  { $match: { timestamp: { $gte: new Date(Date.now() - 24*60*60*1000) } } },
  { $group: {
      _id: {
        year: { $year: "$timestamp" },
        month: { $month: "$timestamp" },
        day: { $dayOfMonth: "$timestamp" },
        hour: { $hour: "$timestamp" }
      },
      positions: { $sum: 1 },
      unique_aircraft: { $addToSet: "$hex" },
      avg_altitude: { $avg: "$altitude" },
      avg_speed: { $avg: "$speed" }
  }},
  { $project: {
      _id: 1,
      positions: 1,
      unique_aircraft: { $size: "$unique_aircraft" },
      avg_altitude: { $round: ["$avg_altitude", 0] },
      avg_speed: { $round: ["$avg_speed", 0] }
  }},
  { $sort: { "_id.year": -1, "_id.month": -1, "_id.day": -1, "_id.hour": -1 } }
])
```

### Aircraft by Type

```javascript
db.aircraft_positions.aggregate([
  { $match: {
      aircraft_type: { $ne: null },
      timestamp: { $gte: new Date(Date.now() - 7*24*60*60*1000) }
  }},
  { $group: {
      _id: "$aircraft_type",
      aircraft_list: { $addToSet: "$hex" },
      total_positions: { $sum: 1 }
  }},
  { $project: {
      _id: 1,
      aircraft_count: { $size: "$aircraft_list" },
      total_positions: 1
  }},
  { $sort: { aircraft_count: -1 } },
  { $limit: 20 }
])
```

### Geospatial Query - Aircraft Near Location

```javascript
db.aircraft_positions.aggregate([
  { $geoNear: {
      near: { type: "Point", coordinates: [-74.0060, 40.7128] },
      distanceField: "distance_meters",
      maxDistance: 50000,  // 50km radius
      query: { timestamp: { $gte: new Date(Date.now() - 60*60*1000) } },
      spherical: true
  }},
  { $group: {
      _id: "$hex",
      flight: { $first: "$flight" },
      closest_distance: { $min: "$distance_meters" },
      last_seen: { $max: "$timestamp" }
  }},
  { $sort: { closest_distance: 1 } },
  { $limit: 20 }
])
```

### Aircraft Within Bounding Box

```javascript
db.aircraft_positions.find({
  location: {
    $geoWithin: {
      $box: [
        [-75.0, 40.0],  // bottom-left [lon, lat]
        [-73.0, 42.0]   // top-right [lon, lat]
      ]
    }
  },
  timestamp: { $gte: new Date(Date.now() - 60*60*1000) }
}).sort({ timestamp: -1 }).limit(100)
```

## TTL Indexes for Auto-Cleanup

MongoDB TTL indexes automatically delete documents after a specified time period, eliminating the need for manual cleanup scripts.

### Creating TTL Index

```javascript
// Delete documents 30 days after timestamp
db.aircraft_positions.createIndex(
  { "timestamp": 1 },
  { expireAfterSeconds: 30 * 24 * 60 * 60, name: "ttl_cleanup" }
)
```

### Modifying TTL Duration

```javascript
// Change TTL to 7 days
db.runCommand({
  collMod: "aircraft_positions",
  index: {
    name: "ttl_cleanup",
    expireAfterSeconds: 7 * 24 * 60 * 60
  }
})
```

### Checking TTL Index Status

```javascript
// View all indexes including TTL
db.aircraft_positions.getIndexes()

// Check TTL thread status
db.serverStatus().metrics.ttl
```

### Manual Cleanup (Alternative)

```javascript
// Delete records older than 30 days manually
db.aircraft_positions.deleteMany({
  timestamp: { $lt: new Date(Date.now() - 30*24*60*60*1000) }
})
```

## Testing & Verification

1. **Start MongoDB** and verify connection:
   ```shell
   mongosh --eval "db.runCommand({ping: 1})"
   ```

2. **Run the logger** and check output for successful inserts

3. **Verify data is being stored**:
   ```javascript
   db.aircraft_positions.countDocuments()
   db.aircraft_positions.find().sort({timestamp: -1}).limit(5)
   ```

4. **Check index usage**:
   ```javascript
   db.aircraft_positions.find({hex: "A12345"}).explain("executionStats")
   ```

5. **Monitor collection stats**:
   ```javascript
   db.aircraft_positions.stats()
   ```

6. **Verify TTL is working**:
   ```javascript
   db.serverStatus().metrics.ttl
   ```

## Troubleshooting

### Connection Refused

**Problem:** `MongoNetworkError: connect ECONNREFUSED`
**Solution:**
- Verify MongoDB is running: `mongosh --eval "db.runCommand({ping: 1})"`
- Check host/port configuration
- Ensure MongoDB is bound to correct interface in `mongod.conf`

### Authentication Failed

**Problem:** `MongoServerError: Authentication failed`
**Solution:**
- Verify username and password
- Include authSource in connection string: `mongodb://user:pass@host:27017/skyspy?authSource=admin`
- Check user has correct database permissions:
  ```javascript
  db.createUser({
    user: "skyspy",
    pwd: "skyspy123",
    roles: [{ role: "readWrite", db: "skyspy" }]
  })
  ```

### Slow Insert Performance

**Problem:** Inserts are slow with many aircraft
**Solution:**
- Use bulk inserts with `insertMany`
- Increase `LOG_INTERVAL` to reduce write frequency
- Consider write concern adjustments for non-critical data:
  ```javascript
  collection.insertOne(doc, { writeConcern: { w: 0 } })
  ```

### Collection Growing Too Large

**Problem:** Database size increasing rapidly
**Solution:**
- Verify TTL index is working correctly
- Check TTL monitor is running: `db.serverStatus().metrics.ttl`
- Increase `LOG_INTERVAL` to log less frequently
- Consider capped collections for fixed-size storage:
  ```javascript
  db.createCollection("aircraft_positions", {
    capped: true,
    size: 1073741824,  // 1GB
    max: 1000000       // 1M documents
  })
  ```

### TTL Index Not Deleting Documents

**Problem:** Old documents not being removed
**Solution:**
- TTL monitor runs every 60 seconds by default
- Check timestamp field is a Date type, not string
- Verify index exists: `db.aircraft_positions.getIndexes()`
- Check TTL stats: `db.serverStatus().metrics.ttl`

### High Memory Usage

**Problem:** MongoDB using too much memory
**Solution:**
- Configure WiredTiger cache size in `mongod.conf`:
  ```yaml
  storage:
    wiredTiger:
      engineConfig:
        cacheSizeGB: 1
  ```
- Create appropriate indexes to avoid collection scans
- Use projections to return only needed fields

## Related Recipes

- [PostgreSQL Archive](/docs/postgresql-archive) - Relational database storage
- [SQLite Local Database](/docs/sqlite-db) - Lightweight local storage
- [InfluxDB Logging](/docs/influxdb-logging) - Time-series database
- [Grafana Dashboard](/docs/grafana-dashboard) - Visualize MongoDB data
