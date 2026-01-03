---
title: "Quick Start"
slug: "quick-start"
excerpt: "Get SkySpy up and running using Docker Compose."
hidden: false
---

Deploy SkySpy using Docker Compose in under 5 minutes.

## Prerequisites

Before you begin, make sure you have:

- **Docker & Docker Compose** — [Install Docker](https://docs.docker.com/get-docker/)
- **An ADS-B receiver** — Running [Ultrafeeder](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder), readsb, or dump1090 on your network

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/your-org/skyspy.git
cd skyspy
```

### 2. Configure environment

Copy the sample environment file and edit it with your settings:

```bash
cp .env.test.sample .env
```

Open `.env` and set these required variables:

| Variable | Description | Example |
| :--- | :--- | :--- |
| `FEEDER_LAT` | Your receiver's latitude | `47.9377` |
| `FEEDER_LON` | Your receiver's longitude | `-121.9687` |
| `ULTRAFEEDER_HOST` | Hostname or IP of your ADS-B receiver | `ultrafeeder` or `192.168.1.100` |

> 📘 Finding your coordinates
>
> Use [Google Maps](https://maps.google.com) — right-click your location and copy the coordinates.

### 3. Start the services

```bash
docker compose up -d
```

Wait for the containers to start (usually 10-30 seconds), then verify they're running:

```bash
docker compose ps
```

## Access the Dashboard

| Service | URL | Description |
| :--- | :--- | :--- |
| **Web Dashboard** | [http://localhost:3000](http://localhost:3000) | Interactive aircraft map |
| **API Docs** | [http://localhost:5000/docs](http://localhost:5000/docs) | Swagger/OpenAPI reference |

> ✅ Success
>
> You should see aircraft appearing on the map within a few seconds if your receiver is working properly.

## Troubleshooting

<Accordion title="No aircraft appearing on the map">

1. Verify your receiver is running and accessible:
   ```bash
   curl http://ULTRAFEEDER_HOST/tar1090/data/aircraft.json
   ```
2. Check that `ULTRAFEEDER_HOST` in your `.env` matches your receiver's address
3. View the API logs for connection errors:
   ```bash
   docker compose logs adsb-api
   ```

</Accordion>

<Accordion title="Container fails to start">

1. Check the logs for the failing container:
   ```bash
   docker compose logs <service-name>
   ```
2. Verify your `.env` file has all required variables set
3. Ensure ports 3000 and 5000 are not in use by other services

</Accordion>

## Local Development

For contributors who want to run services without Docker:

<Tabs>
  <Tab title="Backend">

  ```bash
  cd adsb-api
  pip install -e ".[dev]"
  uvicorn app.main:app --host 0.0.0.0 --port 5000 --reload
  ```

  </Tab>
  <Tab title="Frontend">

  ```bash
  cd web
  npm install
  npm run dev
  ```

  </Tab>
</Tabs>

## Next Steps

<Cards columns={2}>
  <Card title="Configuration" icon="gear" href="/docs/configuration">
    Customize polling, safety thresholds, and integrations
  </Card>
  <Card title="Safety & Alerts" icon="bell" href="/docs/safety-and-alerts">
    Set up custom alert rules and notifications
  </Card>
</Cards>