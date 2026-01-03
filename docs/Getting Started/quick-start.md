---
title: "Quick Start"
slug: "quick-start"
excerpt: "Get SkySpy up and running using Docker Compose."
hidden: false
---

Deploy SkySpy using Docker Compose in under 5 minutes.

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart LR
    A["📥 Clone Repo"] --> B["⚙️ Configure .env"]
    B --> C["🐳 Docker Compose Up"]
    C --> D["🖥️ Open Dashboard"]

    style A fill:#0d4f8b,stroke:#3b82f6,stroke-width:2px,color:#fff
    style B fill:#7c4a03,stroke:#f59e0b,stroke-width:2px,color:#fff
    style C fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
    style D fill:#5b2168,stroke:#a855f7,stroke-width:2px,color:#fff
```

## Prerequisites

<Check>
**Docker & Docker Compose** — [Install Docker](https://docs.docker.com/get-docker/)
</Check>

<Check>
**An ADS-B receiver** — Running [Ultrafeeder](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder), readsb, or dump1090 on your network
</Check>

## Installation

<Steps>
  <Step title="Clone the repository">
    ```bash
    git clone https://github.com/your-org/skyspy.git
    cd skyspy
    ```
  </Step>

  <Step title="Configure environment">
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

    > 📘 **Finding your coordinates**
    >
    > Use [Google Maps](https://maps.google.com) — right-click your location and copy the coordinates.
  </Step>

  <Step title="Start the services">
    ```bash
    docker compose up -d
    ```

    Wait for the containers to start (usually 10-30 seconds), then verify they're running:

    ```bash
    docker compose ps
    ```
  </Step>
</Steps>

## Access the Dashboard

Once the containers are running, you can access:

<CardGroup cols={2}>
  <Card title="Web Dashboard" icon="map" href="http://localhost:3000">
    Interactive aircraft radar at **localhost:3000**
  </Card>
  <Card title="API Documentation" icon="code" href="http://localhost:5000/docs">
    Swagger/OpenAPI reference at **localhost:5000/docs**
  </Card>
</CardGroup>

<Success>
**You should see aircraft appearing on the map within a few seconds** if your receiver is working properly.
</Success>

## What You'll See

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': { 'primaryColor': '#1e3a5f', 'primaryTextColor': '#fff', 'primaryBorderColor': '#3b82f6', 'lineColor': '#60a5fa'}}}%%
flowchart TB
    subgraph Dashboard["🖥️ Web Dashboard"]
        direction LR
        RADAR["🗺️ Radar Display"]
        LIST["📋 Aircraft List"]
        DETAIL["✈️ Aircraft Details"]
    end

    subgraph Data["📡 Live Data"]
        POS["📍 Position Updates"]
        SAFE["🛡️ Safety Alerts"]
        WX["🌦️ Weather Data"]
    end

    Data --> Dashboard

    style Dashboard fill:#065f46,stroke:#10b981,stroke-width:2px,color:#fff
    style Data fill:#0d4f8b,stroke:#3b82f6,stroke-width:2px,color:#fff
```

The dashboard shows:
- **Radar Display** — Aircraft positions on an interactive map
- **Aircraft List** — Sortable table of all tracked aircraft
- **Detail Panel** — Click any aircraft for photos, flight info, and track history
- **Safety Alerts** — Real-time notifications for TCAS, proximity, and emergencies

## Troubleshooting

<AccordionGroup>
  <Accordion title="No aircraft appearing on the map" icon="plane-slash">
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

  <Accordion title="Container fails to start" icon="triangle-exclamation">
    1. Check the logs for the failing container:
       ```bash
       docker compose logs <service-name>
       ```
    2. Verify your `.env` file has all required variables set
    3. Ensure ports 3000 and 5000 are not in use by other services
  </Accordion>

  <Accordion title="Database connection errors" icon="database">
    1. Ensure PostgreSQL container is running:
       ```bash
       docker compose ps postgres
       ```
    2. Check database logs:
       ```bash
       docker compose logs postgres
       ```
    3. Verify `DATABASE_URL` in your `.env` matches the Docker service name
  </Accordion>
</AccordionGroup>

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

## Deployment Checklist

<Checklist>
  - [ ] Docker and Docker Compose installed
  - [ ] ADS-B receiver accessible on network
  - [ ] `.env` file configured with coordinates
  - [ ] Ports 3000 and 5000 available
  - [ ] Containers running (`docker compose ps`)
  - [ ] Aircraft visible on dashboard
</Checklist>

## Next Steps

<Cards columns={2}>
  <Card title="Configuration" icon="gear" href="/docs/configuration">
    Customize polling, safety thresholds, and integrations
  </Card>
  <Card title="Safety & Alerts" icon="bell" href="/docs/safety-and-alerts">
    Set up custom alert rules and notifications
  </Card>
</Cards>
