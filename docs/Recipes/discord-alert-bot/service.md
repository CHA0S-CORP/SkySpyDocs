---
title: "Run as Service"
slug: "discord-alert-bot/service"
excerpt: "Deploy the bot with systemd or Docker."
hidden: false
---

## Systemd (Linux)

Create `/etc/systemd/system/skyspy-discord.service`:

```ini
[Unit]
Description=SkySpy Discord Alert Bot
After=network.target

[Service]
Type=simple
User=your-user
WorkingDirectory=/path/to/bot
Environment="DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/..."
Environment="SKYSPY_URL=http://localhost:5000"
ExecStart=/usr/bin/python3 discord_bot.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl enable skyspy-discord
sudo systemctl start skyspy-discord
```

---

## Docker

Create a `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY discord_bot.py .
RUN pip install sseclient-py requests discord-webhook

CMD ["python", "discord_bot.py"]
```

Run:

```bash
docker build -t skyspy-discord .
docker run -d \
  -e DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..." \
  -e SKYSPY_URL="http://skyspy-api:5000" \
  --name skyspy-discord \
  skyspy-discord
```
