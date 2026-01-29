---
title: Email Alerts
description: Send aircraft notifications via SMTP email.
hidden: false
recipe:
  color: '#EA4335'
  icon: 📧
---
```shell Shell
export SMTP_HOST="smtp.gmail.com"
export SMTP_PORT="587"
export SMTP_USER="your-email@gmail.com"
export SMTP_PASS="your-app-password"
export EMAIL_TO="alerts@example.com"
```

```go Go
package main

import (
	"fmt"
	"net/smtp"
	"os"
)

func main() {
	host := os.Getenv("SMTP_HOST")
	port := os.Getenv("SMTP_PORT")
	user := os.Getenv("SMTP_USER")
	pass := os.Getenv("SMTP_PASS")
	to := os.Getenv("EMAIL_TO")

	auth := smtp.PlainAuth("", user, pass, host)

	subject := "🎖️ Military Aircraft Alert"
	body := "Aircraft RCH419 (C-17) detected at 35,000 ft"

	msg := []byte(fmt.Sprintf(
		"To: %s\r\nSubject: %s\r\nContent-Type: text/plain; charset=UTF-8\r\n\r\n%s",
		to, subject, body,
	))

	smtp.SendMail(host+":"+port, auth, user, []string{to}, msg)
}
```

```python Python
import os
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

SMTP_HOST = os.getenv("SMTP_HOST", "smtp.gmail.com")
SMTP_PORT = int(os.getenv("SMTP_PORT", "587"))
SMTP_USER = os.getenv("SMTP_USER")
SMTP_PASS = os.getenv("SMTP_PASS")
EMAIL_TO = os.getenv("EMAIL_TO")

def send_email_alert(aircraft):
    msg = MIMEMultipart()
    msg["From"] = SMTP_USER
    msg["To"] = EMAIL_TO
    msg["Subject"] = f"🎖️ Military Aircraft: {aircraft.get('flight', 'Unknown')}"

    body = f"""
Aircraft Detected:
- Callsign: {aircraft.get('flight', 'N/A')}
- Type: {aircraft.get('type', 'Unknown')}
- Altitude: {aircraft.get('alt', 0):,} ft
- Speed: {aircraft.get('gs', 0)} kts
    """
    msg.attach(MIMEText(body, "plain"))

    with smtplib.SMTP(SMTP_HOST, SMTP_PORT) as server:
        server.starttls()
        server.login(SMTP_USER, SMTP_PASS)
        server.send_message(msg)

# Example usage
send_email_alert({"flight": "RCH419", "type": "C-17", "alt": 35000, "gs": 450})
```

```javascript JavaScript
const nodemailer = require('nodemailer');

const transporter = nodemailer.createTransport({
    host: process.env.SMTP_HOST || 'smtp.gmail.com',
    port: process.env.SMTP_PORT || 587,
    secure: false,
    auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS,
    },
});

async function sendEmailAlert(aircraft) {
    await transporter.sendMail({
        from: process.env.SMTP_USER,
        to: process.env.EMAIL_TO,
        subject: `🎖️ Military Aircraft: ${aircraft.flight || 'Unknown'}`,
        text: `
Aircraft Detected:
- Callsign: ${aircraft.flight || 'N/A'}
- Type: ${aircraft.type || 'Unknown'}
- Altitude: ${aircraft.alt?.toLocaleString() || 0} ft
- Speed: ${aircraft.gs || 0} kts
        `,
    });
}

// Example usage
sendEmailAlert({ flight: 'RCH419', type: 'C-17', alt: 35000, gs: 450 });
```

```json Response Example
{"accepted": ["alerts@example.com"], "messageId": "<abc123@gmail.com>"}
```

# Configure SMTP Settings

<!-- shell@1-5 -->
<!-- go@1-14 -->
<!-- python@1-10 -->
<!-- javascript@1-12 -->

Set up your SMTP credentials. For Gmail, use an App Password instead of your regular password. Enable 2FA and generate one at [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).

# Build Email Message

<!-- go@16-23 -->
<!-- python@12-25 -->
<!-- javascript@14-25 -->

Create the email with aircraft details. Include the callsign in the subject line for easy filtering. The body contains full flight information.

# Send via SMTP

<!-- go@25-26 -->
<!-- python@27-30 -->
<!-- javascript@27-28 -->

Connect to the SMTP server with TLS and send the message. The connection is closed automatically after sending.
