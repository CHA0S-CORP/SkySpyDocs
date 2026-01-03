---
title: "Quick Start"
slug: "quick-start"
excerpt: "Get SkySpy up and running using Docker Compose."
hidden: false
---

This guide will help you deploy SkySpy using Docker Compose, which is the recommended method for most users.

## Prerequisites

* Docker & Docker Compose
* An ADS-B receiver (Ultrafeeder, readsb, or dump1090) available on your network

## Installation

1.  **Clone the repository**
    ```bash
    git clone [https://github.com/your-org/skyspy.git](https://github.com/your-org/skyspy.git)
    cd skyspy
    ```

2.  **Configure Environment**
    Copy the sample environment file to `.env`:
    ```bash
    cp .env.test.sample .env
    ```

3.  **Edit Configuration**
    Open `.env` and configure the following required variables:
    * `FEEDER_LAT`: Your receiver's latitude.
    * `FEEDER_LON`: Your receiver's longitude.
    * `ULTRAFEEDER_HOST`: The hostname or IP of your ADS-B receiver.

4.  **Start Services**
    ```bash
    docker compose up -d
    ```

## Accessing the Dashboard

Once the containers are running, you can access the services at:

| Service | URL |
| :--- | :--- |
| **Web Dashboard** | `http://localhost:3000` |
| **API Docs (Swagger)** | `http://localhost:5000/docs` |

## Local Development

If you wish to contribute or run the services locally without Docker:

**Backend (adsb-api)**
```bash
cd adsb-api
pip install -e ".[dev]"
uvicorn app.main:app --host 0.0.0.0 --port 5000 --reload

```

**Frontend (web)**

```bash
cd web
npm install
npm run dev

```