---
title: "Cannonball Mode Integration"
excerpt: "Integrate real-time threat detection in a mobile app with GPS tracking and voice alerts."
hidden: true
recipe:
  color: '#EF4444'
  icon: crosshairs
---
```shell Shell
npm install
# React Native with Expo
npx expo install expo-location
# Or standard React Native
npm install @react-native-community/geolocation
```

```swift Swift
// Add to Info.plist:
// NSLocationWhenInUseUsageDescription
// NSLocationAlwaysAndWhenInUseUsageDescription (optional)
import CoreLocation
import AVFoundation

class CannonballManager: NSObject, CLLocationManagerDelegate {
    private var socket: URLSessionWebSocketTask?
    private let locationManager = CLLocationManager()
    private let synthesizer = AVSpeechSynthesizer()

    func start() {
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.requestWhenInUseAuthorization()

        let url = URL(string: "ws://localhost:8000/ws/cannonball/")!
        socket = URLSession.shared.webSocketTask(with: url)
        socket?.resume()
        receiveMessage()

        locationManager.startUpdatingLocation()
    }

    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        let message = ["type": "position_update", "lat": location.coordinate.latitude, "lon": location.coordinate.longitude]
        if let data = try? JSONSerialization.data(withJSONObject: message) {
            socket?.send(.string(String(data: data, encoding: .utf8)!)) { _ in }
        }
    }

    private func receiveMessage() {
        socket?.receive { [weak self] result in
            if case .success(let message) = result, case .string(let text) = message,
               let data = text.data(using: .utf8),
               let json = try? JSONSerialization.jsonObject(with: data) as? [String: Any],
               json["type"] as? String == "threats",
               let threats = json["data"] as? [[String: Any]], !threats.isEmpty {
                self?.announceThreats(threats)
            }
            self?.receiveMessage()
        }
    }

    private func announceThreats(_ threats: [[String: Any]]) {
        guard let threat = threats.first,
              let category = threat["category"] as? String,
              let distance = threat["distance_nm"] as? Double,
              let direction = threat["direction"] as? String else { return }

        let text = "\(category), \(String(format: "%.1f", distance)) miles to the \(direction)"
        let utterance = AVSpeechUtterance(string: text)
        utterance.rate = 0.52
        synthesizer.speak(utterance)
    }
}
```

```kotlin Kotlin
// AndroidManifest.xml permissions:
// ACCESS_FINE_LOCATION, ACCESS_COARSE_LOCATION, INTERNET
class CannonballService : Service() {
    private var socket: WebSocket? = null
    private var fusedLocationClient: FusedLocationProviderClient? = null
    private var tts: TextToSpeech? = null

    override fun onCreate() {
        super.onCreate()
        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)
        tts = TextToSpeech(this) { status -> if (status == TextToSpeech.SUCCESS) tts?.language = Locale.US }
        connectWebSocket()
        startLocationUpdates()
    }

    private fun connectWebSocket() {
        val client = OkHttpClient.Builder().readTimeout(0, TimeUnit.MILLISECONDS).build()
        val request = Request.Builder().url("ws://10.0.2.2:8000/ws/cannonball/").build()
        socket = client.newWebSocket(request, object : WebSocketListener() {
            override fun onMessage(webSocket: WebSocket, text: String) {
                val json = JSONObject(text)
                if (json.getString("type") == "threats") {
                    val threats = json.getJSONArray("data")
                    if (threats.length() > 0) announceThreats(threats)
                }
            }
        })
    }

    private fun startLocationUpdates() {
        val request = LocationRequest.create().setPriority(Priority.PRIORITY_HIGH_ACCURACY).setInterval(3000)
        fusedLocationClient?.requestLocationUpdates(request, object : LocationCallback() {
            override fun onLocationResult(result: LocationResult) {
                result.lastLocation?.let { loc ->
                    val msg = JSONObject().put("type", "position_update").put("lat", loc.latitude).put("lon", loc.longitude)
                    socket?.send(msg.toString())
                }
            }
        }, Looper.getMainLooper())
    }

    private fun announceThreats(threats: JSONArray) {
        val threat = threats.getJSONObject(0)
        val text = "${threat.getString("category")}, ${threat.getDouble("distance_nm")} miles ${threat.getString("direction")}"
        tts?.speak(text, TextToSpeech.QUEUE_FLUSH, null, "threat")
    }
}
```

```javascript React Native
import { useState, useEffect, useRef, useCallback } from 'react';
import * as Location from 'expo-location';
import * as Speech from 'expo-speech';

export function useCannonball({ apiHost = 'ws://localhost:8000', enabled = true }) {
  const [threats, setThreats] = useState([]);
  const [connected, setConnected] = useState(false);
  const wsRef = useRef(null);
  const announcedRef = useRef(new Set());

  useEffect(() => {
    if (!enabled) return;

    (async () => {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') return;

      const ws = new WebSocket(`${apiHost}/ws/cannonball/`);
      wsRef.current = ws;

      ws.onopen = () => setConnected(true);
      ws.onclose = () => setConnected(false);
      ws.onmessage = (e) => {
        const msg = JSON.parse(e.data);
        if (msg.type === 'threats') {
          setThreats(msg.data);
          announceNewThreats(msg.data);
        }
      };

      const subscription = await Location.watchPositionAsync(
        { accuracy: Location.Accuracy.High, timeInterval: 3000 },
        (loc) => {
          if (ws.readyState === WebSocket.OPEN) {
            ws.send(JSON.stringify({
              type: 'position_update',
              lat: loc.coords.latitude,
              lon: loc.coords.longitude,
              heading: loc.coords.heading,
            }));
          }
        }
      );

      return () => { ws.close(); subscription.remove(); };
    })();
  }, [enabled, apiHost]);

  const announceNewThreats = useCallback((threats) => {
    threats.filter(t => !announcedRef.current.has(t.hex)).forEach(t => {
      announcedRef.current.add(t.hex);
      Speech.speak(`${t.category}, ${t.distance_nm.toFixed(1)} miles ${t.direction}`);
      setTimeout(() => announcedRef.current.delete(t.hex), 30000);
    });
  }, []);

  return { threats, connected, threatCount: threats.length };
}
```

```json Response Example
{"type":"threats","data":[{"hex":"A12345","category":"Police Aviation","distance_nm":2.5,"bearing":45,"direction":"NE","trend":"approaching","threat_level":"warning","is_law_enforcement":true}],"count":1}
```

# step1

<!-- shell@ -->
<!-- swift@ -->
<!-- kotlin@ -->
<!-- javascript@ -->

Cannonball Mode provides real-time threat detection for mobile apps. It uses GPS to track your position and alerts you when law enforcement or surveillance aircraft are nearby. This recipe shows how to integrate it into iOS, Android, and React Native apps.

# step2

The integration requires three components:

1. **Location Permission** - Request GPS access from the user
2. **WebSocket Connection** - Connect to `/ws/cannonball/` and handle messages
3. **Voice Alerts** - Use text-to-speech to announce threats hands-free

> 📘 Note
>
> For production apps, implement exponential backoff for WebSocket reconnection and handle background location updates appropriately for each platform.

<!-- shell@ -->
<!-- swift@ -->
<!-- kotlin@ -->
<!-- javascript@ -->

# step3

The server responds with threats sorted by urgency. Each threat includes:

- **distance_nm** - Distance in nautical miles
- **direction** - Cardinal direction (N, NE, E, etc.)
- **trend** - `approaching`, `departing`, or `holding`
- **threat_level** - `critical`, `warning`, or `info`
- **category** - Type of aircraft (Police Aviation, Sheriff, Helicopter, etc.)

> 🚧 Warning
>
> Voice alerts should be rate-limited to avoid overwhelming the user. The example code tracks announced threats and waits 30 seconds before re-announcing the same aircraft.

<!-- shell@ -->
<!-- swift@ -->
<!-- kotlin@ -->
<!-- javascript@ -->

# step4

## GPS Permission Handling

Each platform requires specific permission handling:

**iOS**: Add `NSLocationWhenInUseUsageDescription` to `Info.plist`. For background tracking, also add `NSLocationAlwaysAndWhenInUseUsageDescription` and enable Background Modes > Location updates.

**Android**: Add `ACCESS_FINE_LOCATION` to `AndroidManifest.xml`. For Android 10+, also request `ACCESS_BACKGROUND_LOCATION` if needed.

**React Native (Expo)**: Use `expo-location` which handles permissions cross-platform. For bare React Native, use `@react-native-community/geolocation`.

<!-- shell@ -->
<!-- swift@ -->
<!-- kotlin@ -->
<!-- javascript@ -->

# step5

## Threat Level Color Coding

Use these colors to indicate threat severity in your UI:

| Level | Color | Meaning |
| :--- | :--- | :--- |
| critical | Red (#EF4444) | Law enforcement within 2nm, approaching |
| warning | Orange (#F59E0B) | Law enforcement within 5nm |
| info | Blue (#3B82F6) | Aircraft of interest at distance |

<!-- shell@ -->
<!-- swift@ -->
<!-- kotlin@ -->
<!-- javascript@ -->

# step6

## Configuring the Detection Radius

Adjust the threat detection radius based on your use case:

```javascript
// Set radius to 15 nautical miles
ws.send(JSON.stringify({
  type: 'set_radius',
  radius_nm: 15
}));
```

Recommended values:
- **Urban**: 5-10nm (more aircraft, focused alerts)
- **Suburban**: 15-25nm (default, balanced)
- **Rural/Highway**: 25-50nm (early warning)

<!-- shell@ -->
<!-- swift@ -->
<!-- kotlin@ -->
<!-- javascript@ -->
