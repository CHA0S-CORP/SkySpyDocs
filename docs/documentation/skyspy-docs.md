---
title: SkySpy Documentation
hidden: false
---

<div align="center">

# ✈️ SkySpy Documentation

![Version](https://img.shields.io/badge/version-0.2.0-blue?style=for-the-badge)
![Django](https://img.shields.io/badge/Django-5.0-green?style=for-the-badge&logo=django)
![React](https://img.shields.io/badge/React-18+-61DAFB?style=for-the-badge&logo=react)
![Go](https://img.shields.io/badge/Go-1.23-00ADD8?style=for-the-badge&logo=go)
![License](https://img.shields.io/badge/license-MIT-purple?style=for-the-badge)

**Enterprise-grade ADS-B aircraft tracking and monitoring platform**

[Quick Start](../getting-started/quick-start) · [API Reference](../api-reference/rest-api) · [Deployment](../operations/deployment)

</div>

---

## 🎯 What is SkySpy?

SkySpy is a comprehensive aircraft tracking platform that processes ADS-B, ACARS, and other aviation data sources. It provides real-time visualization, safety monitoring, custom alerts, and rich analytics.

```mermaid
flowchart LR
    subgraph Input["📡 Data Sources"]
        ADSB[ADS-B<br/>Receivers]
        ACARS[ACARS<br/>Decoders]
    end

    subgraph Core["⚙️ SkySpy"]
        API[Django API]
        WS[WebSocket]
    end

    subgraph Output["🖥️ Clients"]
        WEB[Web Dashboard]
        CLI[Go CLI]
        EXT[Your App]
    end

    ADSB --> API
    ACARS --> API
    API --> WS
    WS --> WEB
    WS --> CLI
    API --> EXT
```

---

## 📚 Documentation Index

### 🚀 Getting Started

> 📘 **New to SkySpy?** Start here!

| | Document | Description |
|:---:|:---------|:------------|
| 🏁 | [**Quick Start**](../getting-started/quick-start) | Get running in 5 minutes with Docker |
| 🏗️ | [**Overview**](../getting-started/overview) | Architecture, tech stack, core concepts |
| ⚙️ | [**Configuration**](../getting-started/config-reference) | Complete environment and settings reference |

---

### 🔐 Core Features

| | Document | Description |
|:---:|:---------|:------------|
| 🔑 | [**Authentication**](../core-features/authentication) | JWT, API keys, OIDC, permissions |
| 🗄️ | [**Database**](../core-features/database) | Models, schema, relationships, migrations |

---

### 🔌 API Reference

> 💡 **Building an integration?** These docs are for you.

| | Document | Description |
|:---:|:---------|:------------|
| 🌐 | [**REST API**](../api-reference/rest-api) | Complete HTTP endpoint reference |
| ⚡ | [**WebSocket API**](../api-reference/websocket-api) | Real-time streaming, channels, events |

---

### 🧩 Components

| | Document | Description |
|:---:|:---------|:------------|
| 🐹 | [**Go Services**](../components/go-services) | CLI application, radar display |
| ⚛️ | [**Frontend**](../components/frontend) | React web application architecture |
| ⏰ | [**Background Tasks**](../components/background-tasks) | Celery workers, scheduled jobs |

---

### ✨ Features

| | Document | Description |
|:---:|:---------|:------------|
| 🗺️ | [**Map & Aviation**](../features/map-aviation) | Map layers, weather, aviation data |
| 🚨 | [**Safety & Alerts**](../features/safety-alerts) | Safety monitoring, alert rules |
| 📡 | [**ACARS**](../features/acars) | ACARS/VDL2 message integration |
| 🎯 | [**Cannonball Mode**](../features/cannonball-mode) | Mobile proximity detection |
| 📊 | [**Statistics**](../features/statistics) | Analytics, gamification, exports |

---

### 🛠️ Operations

| | Document | Description |
|:---:|:---------|:------------|
| 🚀 | [**Deployment**](../operations/deployment) | Docker, production, Raspberry Pi |
| 🧪 | [**Testing**](../operations/testing) | Running and writing tests |

---

### 👨‍💻 Development

| | Document | Description |
|:---:|:---------|:------------|
| 🤝 | [**Contributing**](../development/contributing) | Dev setup, code style, PR process |
| 🔧 | [**Troubleshooting**](../development/troubleshooting) | Common issues, debugging, FAQ |

---

## 🏃 Quick Commands

```bash
# 🐳 Start with Docker
docker compose up -d

# 📊 View logs
docker compose logs -f api

# 🧪 Run tests
docker compose run --rm api pytest

# 🔄 Update
git pull && docker compose up -d --build
```

---

## 🔗 Quick Links

<div align="center">

| | Resource | Description |
|:---:|:---------|:------------|
| 📖 | [Swagger UI](/api/docs/) | Interactive API explorer |
| 📋 | [ReDoc](/api/redoc/) | API reference documentation |
| 🖥️ | [Admin](/admin/) | Django admin interface |
| ❤️ | [Health](/health) | System health check |

</div>

---

## 💬 Support

> ⚠️ **Having issues?** Check the [Troubleshooting Guide](../development/troubleshooting) first!

- 🐛 **Bugs**: [GitHub Issues](https://github.com/your-org/skyspy/issues)
- 💡 **Features**: [GitHub Discussions](https://github.com/your-org/skyspy/discussions)
- 📧 **Contact**: support@skyspy.io

---

<div align="center">

**SkySpy v0.2.0** · Built with ❤️ for the aviation community

*Documentation generated with comprehensive codebase analysis*

</div>
