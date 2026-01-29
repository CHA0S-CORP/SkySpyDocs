---
title: Alexa Announcements
excerpt: Voice announcements via Amazon Echo devices.
hidden: false
recipe:
  color: '#00CAFF'
  icon: "🔊"
difficulty: intermediate
tags: [iot, alexa, voice, smart-home]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- Amazon Echo device(s) on your network
- One of: Alexa Notify Me skill, Home Assistant with Alexa integration, or Alexa Media Player

## What You'll Build

Voice announcements through your Alexa devices when aircraft events occur, such as military aircraft overhead, emergency squawks, or specific flights you're tracking.

```shell Shell
# Using Alexa Notify Me skill
pip install sseclient-py requests
export NOTIFY_ME_ACCESS_CODE="your-access-code"
python alexa_announcements.py

# Using Home Assistant
export HA_URL="http://homeassistant.local:8123"
export HA_TOKEN="your-long-lived-access-token"
python ha_alexa_announcements.py
```

```python Python
import json
import os
import sys
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
NOTIFY_ME_ACCESS_CODE = os.getenv("NOTIFY_ME_ACCESS_CODE")
HA_URL = os.getenv("HA_URL")
HA_TOKEN = os.getenv("HA_TOKEN")

# Track announced aircraft to avoid duplicates
announced_aircraft = set()


def announce_via_notify_me(message):
    """Send announcement via Alexa Notify Me skill."""
    if not NOTIFY_ME_ACCESS_CODE:
        print("Error: NOTIFY_ME_ACCESS_CODE required")
        return False

    try:
        response = requests.post(
            "https://api.notifymyecho.com/v1/NotifyMe",
            json={
                "notification": message,
                "accessCode": NOTIFY_ME_ACCESS_CODE
            }
        )
        return response.status_code == 200
    except Exception as e:
        print(f"Notify Me error: {e}")
        return False


def announce_via_home_assistant(message, target="media_player.living_room_echo"):
    """Send announcement via Home Assistant Alexa integration."""
    if not HA_URL or not HA_TOKEN:
        print("Error: HA_URL and HA_TOKEN required")
        return False

    headers = {
        "Authorization": f"Bearer {HA_TOKEN}",
        "Content-Type": "application/json"
    }

    try:
        # Using Alexa Media Player integration
        response = requests.post(
            f"{HA_URL}/api/services/notify/alexa_media",
            headers=headers,
            json={
                "message": message,
                "target": target,
                "data": {"type": "announce"}
            }
        )
        return response.status_code in [200, 201]
    except Exception as e:
        print(f"Home Assistant error: {e}")
        return False


def format_aircraft_message(aircraft):
    """Format aircraft data into a spoken announcement."""
    flight = aircraft.get("flight", "").strip() or "Unknown flight"
    alt = aircraft.get("alt", 0)
    distance = aircraft.get("distance", 0)
    ac_type = aircraft.get("type", "aircraft")

    message = f"Aircraft alert. {flight}"

    if ac_type and ac_type != "Unknown":
        message += f", a {ac_type},"

    if alt:
        message += f" at {alt} feet"

    if distance:
        message += f", {round(distance, 1)} miles away"

    if aircraft.get("military"):
        message = "Military " + message.lower()

    return message


def process_aircraft(aircraft_list):
    """Process aircraft and announce relevant ones."""
    for aircraft in aircraft_list:
        hex_code = aircraft.get("hex")

        # Skip if already announced
        if hex_code in announced_aircraft:
            continue

        # Announce military aircraft
        if aircraft.get("military"):
            message = format_aircraft_message(aircraft)
            if NOTIFY_ME_ACCESS_CODE:
                announce_via_notify_me(message)
            elif HA_URL:
                announce_via_home_assistant(message)
            announced_aircraft.add(hex_code)
            print(f"Announced: {message}")

        # Announce emergency squawks
        squawk = aircraft.get("squawk", "")
        if squawk in ["7500", "7600", "7700"]:
            emergency_types = {
                "7500": "hijacking",
                "7600": "radio failure",
                "7700": "general emergency"
            }
            message = f"Emergency alert. {aircraft.get('flight', 'Aircraft')} is squawking {squawk}, indicating {emergency_types[squawk]}"
            if NOTIFY_ME_ACCESS_CODE:
                announce_via_notify_me(message)
            elif HA_URL:
                announce_via_home_assistant(message)
            announced_aircraft.add(hex_code)
            print(f"Emergency announced: {message}")


def main():
    print(f"Alexa Announcements connected to {SKYSPY_URL}...")

    if NOTIFY_ME_ACCESS_CODE:
        print("Using Alexa Notify Me skill")
    elif HA_URL:
        print(f"Using Home Assistant at {HA_URL}")
    else:
        print("Error: Configure NOTIFY_ME_ACCESS_CODE or HA_URL/HA_TOKEN")
        sys.exit(1)

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    aircraft_list = data.get("aircraft", [])
                    if aircraft_list:
                        process_aircraft(aircraft_list)

        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            import time
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const fetch = require('node-fetch');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const NOTIFY_ME_ACCESS_CODE = process.env.NOTIFY_ME_ACCESS_CODE;
const HA_URL = process.env.HA_URL;
const HA_TOKEN = process.env.HA_TOKEN;

const announcedAircraft = new Set();

async function announceViaNotifyMe(message) {
  if (!NOTIFY_ME_ACCESS_CODE) return false;

  try {
    const response = await fetch('https://api.notifymyecho.com/v1/NotifyMe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        notification: message,
        accessCode: NOTIFY_ME_ACCESS_CODE
      })
    });
    return response.ok;
  } catch (e) {
    console.error('Notify Me error:', e.message);
    return false;
  }
}

async function announceViaHomeAssistant(message, target = 'media_player.living_room_echo') {
  if (!HA_URL || !HA_TOKEN) return false;

  try {
    const response = await fetch(`${HA_URL}/api/services/notify/alexa_media`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${HA_TOKEN}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        message,
        target,
        data: { type: 'announce' }
      })
    });
    return response.ok;
  } catch (e) {
    console.error('Home Assistant error:', e.message);
    return false;
  }
}

function formatAircraftMessage(aircraft) {
  const flight = (aircraft.flight || '').trim() || 'Unknown flight';
  const alt = aircraft.alt || 0;
  const distance = aircraft.distance || 0;
  const acType = aircraft.type || 'aircraft';

  let message = `Aircraft alert. ${flight}`;

  if (acType && acType !== 'Unknown') {
    message += `, a ${acType},`;
  }
  if (alt) message += ` at ${alt} feet`;
  if (distance) message += `, ${distance.toFixed(1)} miles away`;

  if (aircraft.military) {
    message = 'Military ' + message.toLowerCase();
  }

  return message;
}

async function processAircraft(aircraftList) {
  for (const aircraft of aircraftList) {
    const hexCode = aircraft.hex;
    if (announcedAircraft.has(hexCode)) continue;

    // Announce military aircraft
    if (aircraft.military) {
      const message = formatAircraftMessage(aircraft);
      if (NOTIFY_ME_ACCESS_CODE) {
        await announceViaNotifyMe(message);
      } else if (HA_URL) {
        await announceViaHomeAssistant(message);
      }
      announcedAircraft.add(hexCode);
      console.log('Announced:', message);
    }

    // Announce emergency squawks
    const squawk = aircraft.squawk || '';
    if (['7500', '7600', '7700'].includes(squawk)) {
      const emergencyTypes = {
        '7500': 'hijacking',
        '7600': 'radio failure',
        '7700': 'general emergency'
      };
      const message = `Emergency alert. ${aircraft.flight || 'Aircraft'} is squawking ${squawk}, indicating ${emergencyTypes[squawk]}`;
      if (NOTIFY_ME_ACCESS_CODE) {
        await announceViaNotifyMe(message);
      } else if (HA_URL) {
        await announceViaHomeAssistant(message);
      }
      announcedAircraft.add(hexCode);
      console.log('Emergency announced:', message);
    }
  }
}

console.log(`Alexa Announcements connected to ${SKYSPY_URL}...`);

if (NOTIFY_ME_ACCESS_CODE) {
  console.log('Using Alexa Notify Me skill');
} else if (HA_URL) {
  console.log(`Using Home Assistant at ${HA_URL}`);
} else {
  console.error('Error: Configure NOTIFY_ME_ACCESS_CODE or HA_URL/HA_TOKEN');
  process.exit(1);
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);
  const aircraft = data.aircraft || [];
  if (aircraft.length) {
    await processAircraft(aircraft);
  }
});

es.onerror = (e) => {
  console.error('SSE connection error, reconnecting...');
};
```

## Setup Alexa Notify Me

1. Enable the "Notify Me" skill in the Alexa app
2. Say "Alexa, open Notify Me" to get your access code
3. The access code will be sent to your email
4. Set the `NOTIFY_ME_ACCESS_CODE` environment variable

## Setup Home Assistant Method

<!-- python@37-60 -->

If you have Home Assistant with the Alexa Media Player integration:

1. Install [Alexa Media Player](https://github.com/custom-components/alexa_media_player) via HACS
2. Configure it with your Amazon account
3. Use the `notify.alexa_media` service to send announcements

## Format Messages

<!-- python@64-82 -->

The message formatter creates natural-sounding announcements. You can customize the format for different announcement types.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |
| `NOTIFY_ME_ACCESS_CODE` | Access code from Notify Me skill | Required (if not using HA) |
| `HA_URL` | Home Assistant URL | Required (if not using Notify Me) |
| `HA_TOKEN` | Home Assistant long-lived token | Required (if using HA) |
| `ALEXA_TARGET` | Target Echo device entity ID | `media_player.living_room_echo` |

### Announcement Triggers

Customize which events trigger announcements:

| Trigger | Description | Default |
|---------|-------------|---------|
| Military aircraft | Any military aircraft detected | Enabled |
| Emergency squawk | 7500, 7600, 7700 transponder codes | Enabled |
| Specific flights | Track specific callsigns | Disabled |
| Low altitude | Aircraft below threshold | Disabled |

## Testing & Verification

1. **Test Notify Me directly**:
   ```shell
   curl -X POST https://api.notifymyecho.com/v1/NotifyMe \
     -H "Content-Type: application/json" \
     -d '{"notification":"Test from SkySpy","accessCode":"YOUR_CODE"}'
   ```

2. **Test Home Assistant**:
   ```shell
   curl -X POST $HA_URL/api/services/notify/alexa_media \
     -H "Authorization: Bearer $HA_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"message":"Test from SkySpy","target":"media_player.living_room_echo","data":{"type":"announce"}}'
   ```

3. **Start the script** and wait for aircraft events
4. **Verify announcements** play on your Echo devices

## Troubleshooting

### Notify Me Not Working

**Problem:** Announcements don't play via Notify Me
**Solution:**
- Verify access code is correct
- Check the Notify Me skill is enabled
- Ensure Echo device is online and not in Do Not Disturb mode
- Test with a direct API call first

### Home Assistant Announcements Silent

**Problem:** HA method doesn't produce sound
**Solution:**
- Verify Alexa Media Player integration is configured
- Check entity ID matches your device (`media_player.xxx`)
- Confirm Echo is linked in Home Assistant
- Check HA logs for service call errors

### Duplicate Announcements

**Problem:** Same aircraft announced multiple times
**Solution:**
- The script tracks announced aircraft by hex code
- Restart the script to clear the tracking set
- Adjust the tracking logic for your use case

### Rate Limiting

**Problem:** Too many announcements or API errors
**Solution:**
- Notify Me has rate limits; space out announcements
- Add a cooldown period between announcements
- Filter to only announce high-priority events

## Related Recipes

- [Home Assistant Integration](/docs/home-assistant) - Full HA integration
- [Pushover Notifications](/docs/pushover-notifications) - Mobile push alerts
- [Discord Bot](/docs/discord-bot) - Discord notifications
- [Telegram Bot](/docs/telegram-bot) - Telegram alerts
