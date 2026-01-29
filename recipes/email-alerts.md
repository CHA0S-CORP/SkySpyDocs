---
title: Email Alerts
excerpt: Send aircraft notifications via SMTP email.
hidden: false
recipe:
  color: '#EA4335'
  icon: 📧
difficulty: beginner
tags: [notifications, email, smtp, alerts]
---

## Prerequisites

- SkySpy running locally or accessible at a URL
- SMTP server credentials (Gmail, SendGrid, or your own)
- Python: `pip install sseclient-py requests`

## What You'll Build

Email notifications with HTML formatting for aircraft alerts. Useful for alerts that need to be archived or shared with non-technical users.

```shell Shell
pip install sseclient-py requests
export SMTP_HOST="smtp.gmail.com"
export SMTP_PORT="587"
export SMTP_USER="your-email@gmail.com"
export SMTP_PASS="your-app-password"
export EMAIL_TO="recipient@example.com"
python email_alerts.py
```

```go Go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "net/http"
    "net/smtp"
    "os"
    "strings"
)

func main() {
    smtpHost := os.Getenv("SMTP_HOST")
    smtpPort := os.Getenv("SMTP_PORT")
    smtpUser := os.Getenv("SMTP_USER")
    smtpPass := os.Getenv("SMTP_PASS")
    emailTo := os.Getenv("EMAIL_TO")
    skyspyURL := os.Getenv("SKYSPY_URL")
    if skyspyURL == "" {
        skyspyURL = "http://localhost:5000"
    }

    alerted := make(map[string]bool)
    auth := smtp.PlainAuth("", smtpUser, smtpPass, smtpHost)

    for {
        resp, _ := http.Get(skyspyURL + "/api/v1/map/sse")
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

                        if alerted[hex] || ac["military"] != true {
                            continue
                        }

                        subject := fmt.Sprintf("Military Aircraft: %s", ac["flight"])
                        body := fmt.Sprintf("Type: %s\nAltitude: %v ft\nSpeed: %v kts\nICAO: %s",
                            ac["type"], ac["alt"], ac["gs"], hex)

                        msg := []byte(fmt.Sprintf("Subject: %s\r\n\r\n%s", subject, body))
                        smtp.SendMail(smtpHost+":"+smtpPort, auth, smtpUser, []string{emailTo}, msg)
                        alerted[hex] = true
                        fmt.Printf("Emailed: %s\n", ac["flight"])
                    }
                }
            }
        }
        resp.Body.Close()
    }
}
```

```python Python
import json
import os
import sys
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime
import requests
import sseclient

SKYSPY_URL = os.getenv("SKYSPY_URL", "http://localhost:5000")
SMTP_HOST = os.getenv("SMTP_HOST", "smtp.gmail.com")
SMTP_PORT = int(os.getenv("SMTP_PORT", "587"))
SMTP_USER = os.getenv("SMTP_USER")
SMTP_PASS = os.getenv("SMTP_PASS")
EMAIL_FROM = os.getenv("EMAIL_FROM", SMTP_USER)
EMAIL_TO = os.getenv("EMAIL_TO")

if not all([SMTP_USER, SMTP_PASS, EMAIL_TO]):
    print("Error: SMTP_USER, SMTP_PASS, and EMAIL_TO required")
    sys.exit(1)

alerted = set()


def send_email(aircraft):
    hex_code = aircraft.get("hex")
    flight = aircraft.get("flight", hex_code)

    msg = MIMEMultipart("alternative")
    msg["Subject"] = f"🎖️ Military Aircraft Alert: {flight}"
    msg["From"] = EMAIL_FROM
    msg["To"] = EMAIL_TO

    # Plain text version
    text = f"""
Military Aircraft Detected

Callsign: {flight}
Type: {aircraft.get('type', 'Unknown')}
Altitude: {aircraft.get('alt', 0):,} ft
Speed: {aircraft.get('gs', 0)} kts
Distance: {aircraft.get('distance', 0):.1f} nm
ICAO: {hex_code}

Time: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}

Track on FlightRadar24: https://www.flightradar24.com/{hex_code}
Track on ADS-B Exchange: https://globe.adsbexchange.com/?icao={hex_code}
"""

    # HTML version
    html = f"""
<html>
<head>
    <style>
        body {{ font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; }}
        .header {{ background: #6366f1; color: white; padding: 20px; }}
        .content {{ padding: 20px; }}
        .field {{ margin: 10px 0; }}
        .label {{ font-weight: bold; color: #666; }}
        .value {{ font-size: 1.2em; }}
        .button {{ display: inline-block; padding: 10px 20px; background: #6366f1; color: white; text-decoration: none; border-radius: 5px; margin: 5px; }}
    </style>
</head>
<body>
    <div class="header">
        <h1>🎖️ Military Aircraft Alert</h1>
    </div>
    <div class="content">
        <div class="field">
            <span class="label">Callsign:</span>
            <span class="value">{flight}</span>
        </div>
        <div class="field">
            <span class="label">Type:</span>
            <span class="value">{aircraft.get('type', 'Unknown')}</span>
        </div>
        <div class="field">
            <span class="label">Altitude:</span>
            <span class="value">{aircraft.get('alt', 0):,} ft</span>
        </div>
        <div class="field">
            <span class="label">Speed:</span>
            <span class="value">{aircraft.get('gs', 0)} kts</span>
        </div>
        <div class="field">
            <span class="label">Distance:</span>
            <span class="value">{aircraft.get('distance', 0):.1f} nm</span>
        </div>
        <div class="field">
            <span class="label">ICAO:</span>
            <span class="value">{hex_code}</span>
        </div>
        <br>
        <a href="https://www.flightradar24.com/{hex_code}" class="button">View on FlightRadar24</a>
        <a href="https://globe.adsbexchange.com/?icao={hex_code}" class="button">View on ADS-B Exchange</a>
    </div>
</body>
</html>
"""

    msg.attach(MIMEText(text, "plain"))
    msg.attach(MIMEText(html, "html"))

    try:
        with smtplib.SMTP(SMTP_HOST, SMTP_PORT) as server:
            server.starttls()
            server.login(SMTP_USER, SMTP_PASS)
            server.sendmail(EMAIL_FROM, EMAIL_TO, msg.as_string())
        print(f"Emailed: {flight} ({hex_code})")
    except Exception as e:
        print(f"Failed to send email: {e}")


def main():
    print(f"Email alerts connected to {SKYSPY_URL}...")

    while True:
        try:
            response = requests.get(f"{SKYSPY_URL}/api/v1/map/sse", stream=True, timeout=30)
            client = sseclient.SSEClient(response)

            for event in client.events():
                if event.event in ["aircraft_update", "aircraft_new"]:
                    data = json.loads(event.data)
                    for aircraft in data.get("aircraft", []):
                        hex_code = aircraft.get("hex")
                        if hex_code in alerted:
                            continue
                        if aircraft.get("military"):
                            send_email(aircraft)
                            alerted.add(hex_code)
        except Exception as e:
            print(f"Error: {e}, reconnecting...")
            import time
            time.sleep(5)


if __name__ == "__main__":
    main()
```

```javascript JavaScript
const EventSource = require('eventsource');
const nodemailer = require('nodemailer');

const SKYSPY_URL = process.env.SKYSPY_URL || 'http://localhost:5000';
const SMTP_HOST = process.env.SMTP_HOST || 'smtp.gmail.com';
const SMTP_PORT = parseInt(process.env.SMTP_PORT || '587');
const SMTP_USER = process.env.SMTP_USER;
const SMTP_PASS = process.env.SMTP_PASS;
const EMAIL_TO = process.env.EMAIL_TO;

if (!SMTP_USER || !SMTP_PASS || !EMAIL_TO) {
  console.error('Error: SMTP_USER, SMTP_PASS, and EMAIL_TO required');
  process.exit(1);
}

const transporter = nodemailer.createTransport({
  host: SMTP_HOST,
  port: SMTP_PORT,
  secure: false,
  auth: { user: SMTP_USER, pass: SMTP_PASS }
});

const alerted = new Set();

async function sendEmail(aircraft) {
  const hex = aircraft.hex;
  const flight = aircraft.flight || hex;

  await transporter.sendMail({
    from: SMTP_USER,
    to: EMAIL_TO,
    subject: `🎖️ Military Aircraft Alert: ${flight}`,
    text: `Military Aircraft Detected\n\nCallsign: ${flight}\nType: ${aircraft.type || 'Unknown'}\nAltitude: ${aircraft.alt || 0} ft`,
    html: `<h1>🎖️ Military Aircraft: ${flight}</h1><p>Type: ${aircraft.type || 'Unknown'}<br>Altitude: ${aircraft.alt || 0} ft</p>`
  });

  console.log(`Emailed: ${flight}`);
}

const es = new EventSource(`${SKYSPY_URL}/api/v1/map/sse`);

es.addEventListener('aircraft_update', async (e) => {
  const data = JSON.parse(e.data);

  for (const aircraft of data.aircraft || []) {
    if (alerted.has(aircraft.hex)) continue;
    if (aircraft.military) {
      await sendEmail(aircraft);
      alerted.add(aircraft.hex);
    }
  }
});
```

```json Response Example
{
  "accepted": ["recipient@example.com"],
  "messageId": "<abc123@smtp.gmail.com>",
  "response": "250 Message accepted"
}
```

## Gmail Setup

1. Enable 2-factor authentication on your Google account
2. Generate an App Password: Google Account → Security → App Passwords
3. Use the 16-character app password as `SMTP_PASS`

## Send HTML Emails

<!-- python@35-90 -->

HTML emails look professional with styled headers, colored fields, and clickable buttons. The code sends both plain text and HTML versions for compatibility.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SMTP_HOST` | SMTP server hostname | `smtp.gmail.com` |
| `SMTP_PORT` | SMTP server port | `587` |
| `SMTP_USER` | SMTP username/email | Required |
| `SMTP_PASS` | SMTP password/app password | Required |
| `EMAIL_FROM` | Sender address | Same as SMTP_USER |
| `EMAIL_TO` | Recipient address | Required |
| `SKYSPY_URL` | SkySpy server URL | `http://localhost:5000` |

### Multiple Recipients

Send to multiple addresses:

```python
EMAIL_TO = "user1@example.com,user2@example.com"
# Or as a list
recipients = EMAIL_TO.split(",")
server.sendmail(EMAIL_FROM, recipients, msg.as_string())
```

### Daily Digest

Instead of immediate emails, collect alerts and send a daily summary:

```python
alerts_today = []

def add_to_digest(aircraft):
    alerts_today.append(aircraft)

def send_digest():
    if alerts_today:
        # Send summary email
        alerts_today.clear()

# Schedule with APScheduler or similar
```

## Testing & Verification

1. **Test SMTP connection**:
   ```python
   import smtplib
   with smtplib.SMTP("smtp.gmail.com", 587) as s:
       s.starttls()
       s.login("user@gmail.com", "app-password")
       print("Connected!")
   ```
2. **Start the script** and verify it connects
3. **Wait for aircraft** or temporarily broaden the filter
4. **Check inbox** (and spam folder) for formatted emails

## Troubleshooting

### Authentication Failed

**Problem:** `SMTPAuthenticationError`
**Solution:**
- For Gmail: Use an App Password, not your regular password
- Enable "Less secure apps" or use App Passwords
- Verify credentials are correct

### Connection Refused

**Problem:** `Connection refused` error
**Solution:**
- Verify SMTP host and port
- Check firewall isn't blocking outbound SMTP
- Try port 465 with SSL instead of 587 with TLS

### Emails Going to Spam

**Problem:** Emails land in spam folder
**Solution:**
- Use a reputable SMTP service (SendGrid, SES)
- Set up SPF/DKIM records for your domain
- Avoid spam trigger words in subject lines

### Rate Limiting

**Problem:** Too many emails being sent
**Solution:**
- Add cooldown between emails
- Use digest mode for high-volume alerts
- Consider using a transactional email service

## Related Recipes

- [Telegram Bot](/docs/telegram-bot) - Instant mobile alerts
- [Pushover Notifications](/docs/pushover-notifications) - Push notifications
- [Slack Integration](/docs/slack-integration) - Team notifications
- [Discord Alert Bot](/docs/discord-alert-bot) - Discord webhooks
