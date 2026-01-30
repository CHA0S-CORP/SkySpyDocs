---
title: Authentication & Authorization
hidden: false
---

# 🔐 Authentication and Authorization

SkySpy provides a comprehensive, enterprise-ready authentication and authorization system supporting multiple authentication methods, role-based access control (RBAC), and fine-grained feature permissions.

---

## ⚡ Quick Reference

> 📋 **TL;DR** - Everything you need to get started in 30 seconds

| What | Where | Example |
|------|-------|---------|
| 🔑 **Login** | `POST /api/v1/auth/login` | `{"username": "...", "password": "..."}` |
| 🔄 **Refresh Token** | `POST /api/v1/auth/refresh` | `{"refresh": "eyJ..."}` |
| 📤 **Use Token** | `Authorization` header | `Bearer eyJ0eXAiOiJKV1Q...` |
| 🔗 **API Key** | `X-API-Key` header | `sk_a1B2c3D4e5F6g7H8...` |
| 🌐 **SSO/OIDC** | `GET /api/v1/auth/oidc/authorize` | Redirects to IdP |
| 👤 **Current User** | `GET /api/v1/auth/profile` | Returns user + permissions |

[block:parameters]
{
  "data": {
    "h-0": "Auth Method",
    "h-1": "Best For",
    "h-2": "Token Lifetime",
    "0-0": "🔑 JWT",
    "0-1": "Web apps, SPAs",
    "0-2": "60 min (access) / 2 days (refresh)",
    "1-0": "🔗 API Key",
    "1-1": "Scripts, integrations, CI/CD",
    "1-2": "Custom (up to years)",
    "2-0": "🌐 OIDC/SSO",
    "2-1": "Enterprise, corporate IdP",
    "2-2": "Follows IdP settings"
  },
  "cols": 3,
  "rows": 3
}
[/block]

---

## 🏗️ Authentication Overview

SkySpy's authentication system is designed with flexibility and security in mind. It supports three operational modes and multiple authentication methods to accommodate various deployment scenarios.

### Architecture Diagram

```mermaid
flowchart TB
    subgraph Frontend["🖥️ Frontend"]
        WEB[Web App<br/>React]
        AUTH_CTX[AuthContext<br/>JWT + OIDC]
    end

    subgraph API["⚙️ Backend API"]
        REST[REST API<br/>Django REST]
        WS[WebSocket<br/>Channels]
    end

    subgraph AuthLayer["🔐 Auth Layer"]
        JWT_AUTH[JWT Auth]
        API_KEY[API Key Auth]
        OIDC[OIDC/SSO]
    end

    subgraph Permissions["🛡️ Authorization"]
        RBAC[Role-Based<br/>Access Control]
        FEATURE[Feature-Based<br/>Permissions]
    end

    WEB --> AUTH_CTX
    AUTH_CTX --> REST
    AUTH_CTX --> WS
    REST --> JWT_AUTH
    REST --> API_KEY
    REST --> OIDC
    WS --> JWT_AUTH
    WS --> API_KEY
    JWT_AUTH --> RBAC
    API_KEY --> RBAC
    OIDC --> RBAC
    RBAC --> FEATURE
```

### 🎛️ Authentication Modes

SkySpy operates in one of three authentication modes, configured via the `AUTH_MODE` environment variable:

[block:parameters]
{
  "data": {
    "h-0": "Mode",
    "h-1": "Description",
    "h-2": "Use Case",
    "h-3": "Security Level",
    "0-0": "🟢 `public`",
    "0-1": "No authentication required",
    "0-2": "Development, demos, public kiosks",
    "0-3": "⚠️ Low",
    "1-0": "🔴 `private`",
    "1-1": "Authentication required for all endpoints",
    "1-2": "Enterprise, security-sensitive deployments",
    "1-3": "✅ High",
    "2-0": "🟡 `hybrid`",
    "2-1": "Per-feature configuration **(default)**",
    "2-2": "Most production deployments",
    "2-3": "✅ Flexible"
  },
  "cols": 4,
  "rows": 3
}
[/block]

```bash
# Environment variable configuration
AUTH_MODE=hybrid  # Options: public, private, hybrid
```

---

## 🔑 Authentication Methods

### Method Comparison

[block:callout]
{
  "type": "info",
  "title": "🤔 Which method should I use?",
  "body": "**Web Applications** → JWT tokens with automatic refresh\n**Scripts & Automation** → API keys with scoped access\n**Enterprise/Corporate** → OIDC/SSO with your identity provider"
}
[/block]

[block:parameters]
{
  "data": {
    "h-0": "Feature",
    "h-1": "🔑 JWT",
    "h-2": "🔗 API Key",
    "h-3": "🌐 OIDC",
    "0-0": "**Stateless**",
    "0-1": "✅",
    "0-2": "✅",
    "0-3": "✅",
    "1-0": "**Auto Refresh**",
    "1-1": "✅",
    "1-2": "❌",
    "1-3": "✅",
    "2-0": "**Scoped Access**",
    "2-1": "❌",
    "2-2": "✅",
    "2-3": "✅",
    "3-0": "**User Context**",
    "3-1": "✅",
    "3-2": "✅",
    "3-3": "✅",
    "4-0": "**External IdP**",
    "4-1": "❌",
    "4-2": "❌",
    "4-3": "✅",
    "5-0": "**WebSocket Support**",
    "5-1": "✅",
    "5-2": "✅",
    "5-3": "✅"
  },
  "cols": 4,
  "rows": 6
}
[/block]

---

### 1️⃣ JWT Token Authentication

SkySpy uses JSON Web Tokens (JWT) for stateless authentication. The implementation is built on `djangorestframework-simplejwt`.

#### 🧬 Token Structure Visual

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              JWT ACCESS TOKEN                                │
├─────────────────────────────────────────────────────────────────────────────┤
│  HEADER              │  PAYLOAD                    │  SIGNATURE              │
│  ───────             │  ────────                   │  ──────────             │
│  {                   │  {                          │                         │
│    "typ": "JWT",     │    "token_type": "access",  │  HMACSHA256(            │
│    "alg": "HS256"    │    "exp": 1704067200,       │    base64(header) +     │
│  }                   │    "user_id": 42,           │    base64(payload),     │
│                      │    "jti": "unique-id"       │    secret               │
│                      │  }                          │  )                       │
├─────────────────────────────────────────────────────────────────────────────┤
│  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIi4uLn0   │
│  ─────────────────────────────────────────────────────────────────────────── │
│  🔵 Header (Base64)    🟢 Payload (Base64)              🔴 Signature         │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### ⚙️ Configuration

```bash
# JWT Settings (environment variables)
JWT_SECRET_KEY=your-secret-key          # Separate from Django SECRET_KEY recommended
JWT_ACCESS_TOKEN_LIFETIME_MINUTES=60    # Access token validity (default: 60 min)
JWT_REFRESH_TOKEN_LIFETIME_DAYS=2       # Refresh token validity (default: 2 days)
JWT_AUTH_COOKIE=false                   # Enable httpOnly cookie storage
```

#### 📡 Token Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/auth/login` | POST | 🔓 Obtain access and refresh tokens |
| `/api/v1/auth/refresh` | POST | 🔄 Refresh access token |
| `/api/v1/auth/logout` | POST | 🚪 Blacklist refresh token |

#### 🔐 JWT Authentication Flow

```mermaid
sequenceDiagram
    autonumber
    participant User as 👤 User
    participant App as 🖥️ Frontend
    participant API as ⚙️ SkySpy API
    participant DB as 🗄️ Database

    User->>App: Enter credentials
    App->>API: POST /auth/login<br/>{username, password}
    API->>DB: Validate credentials
    DB-->>API: User verified ✅
    API-->>App: {access_token, refresh_token, user}
    App->>App: Store tokens in localStorage

    Note over User,DB: 🔄 Making Authenticated Requests

    App->>API: GET /aircraft<br/>Authorization: Bearer {token}
    API->>API: Validate JWT signature
    API-->>App: Aircraft data 📡

    Note over User,DB: ⏰ Token Refresh (before expiry)

    App->>API: POST /auth/refresh<br/>{refresh_token}
    API->>DB: Validate & rotate token
    DB-->>API: New tokens generated
    API-->>App: {new_access_token, new_refresh_token}
```

#### 💻 Code Examples

[block:code]
{
  "codes": [
    {
      "code": "# Login Request\ncurl -X POST https://your-skyspy-instance/api/v1/auth/login \\\n  -H \"Content-Type: application/json\" \\\n  -d '{\n    \"username\": \"operator\",\n    \"password\": \"secure-password\"\n  }'",
      "language": "bash",
      "name": "cURL"
    },
    {
      "code": "// Login Request\nconst response = await fetch('https://your-skyspy-instance/api/v1/auth/login', {\n  method: 'POST',\n  headers: { 'Content-Type': 'application/json' },\n  body: JSON.stringify({\n    username: 'operator',\n    password: 'secure-password'\n  })\n});\n\nconst { access, refresh, user } = await response.json();\nlocalStorage.setItem('skyspy_access_token', access);\nlocalStorage.setItem('skyspy_refresh_token', refresh);",
      "language": "javascript",
      "name": "JavaScript"
    },
    {
      "code": "import requests\n\n# Login Request\nresponse = requests.post(\n    'https://your-skyspy-instance/api/v1/auth/login',\n    json={\n        'username': 'operator',\n        'password': 'secure-password'\n    }\n)\n\ntokens = response.json()\naccess_token = tokens['access']\nrefresh_token = tokens['refresh']",
      "language": "python",
      "name": "Python"
    }
  ]
}
[/block]

#### ✅ Login Response

```json
{
  "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "user": {
    "id": 42,
    "username": "operator",
    "email": "operator@example.com",
    "display_name": "John Operator",
    "permissions": ["aircraft.view", "alerts.create", "alerts.edit"],
    "roles": ["operator"]
  }
}
```

#### 🔄 Using JWT Tokens

[block:code]
{
  "codes": [
    {
      "code": "# Include the access token in the Authorization header\ncurl https://your-skyspy-instance/api/v1/aircraft \\\n  -H \"Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...\"",
      "language": "bash",
      "name": "cURL"
    },
    {
      "code": "// Using the token in fetch requests\nconst token = localStorage.getItem('skyspy_access_token');\n\nconst response = await fetch('https://your-skyspy-instance/api/v1/aircraft', {\n  headers: {\n    'Authorization': `Bearer ${token}`\n  }\n});",
      "language": "javascript",
      "name": "JavaScript"
    },
    {
      "code": "import requests\n\nheaders = {\n    'Authorization': f'Bearer {access_token}'\n}\n\nresponse = requests.get(\n    'https://your-skyspy-instance/api/v1/aircraft',\n    headers=headers\n)",
      "language": "python",
      "name": "Python"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "success",
  "title": "🔒 Security Note",
  "body": "Refresh tokens are **rotated on use** and the old token is blacklisted. This provides protection against token theft and replay attacks."
}
[/block]

---

### 2️⃣ API Key Authentication

API keys provide programmatic access for integrations, scripts, and third-party applications.

#### 🔗 Key Format

```
sk_a1B2c3D4e5F6g7H8i9J0k1L2m3N4o5P6q7R8s9T0
└┬┘ └──────────────────────────────────────┘
 │              Random characters
 │
 └── Prefix identifier
```

[block:callout]
{
  "type": "warning",
  "title": "⚠️ Important",
  "body": "The full API key is **only returned once** at creation time. Store it securely immediately - you cannot retrieve it later!"
}
[/block]

#### 🔨 Creating API Keys

[block:code]
{
  "codes": [
    {
      "code": "curl -X POST https://your-skyspy-instance/api/v1/auth/api-keys \\\n  -H \"Authorization: Bearer <access-token>\" \\\n  -H \"Content-Type: application/json\" \\\n  -d '{\n    \"name\": \"CI/CD Pipeline\",\n    \"scopes\": [\"aircraft\", \"alerts\"],\n    \"expires_at\": \"2025-12-31T23:59:59Z\"\n  }'",
      "language": "bash",
      "name": "cURL"
    },
    {
      "code": "const response = await authFetch('/api/v1/auth/api-keys', {\n  method: 'POST',\n  headers: { 'Content-Type': 'application/json' },\n  body: JSON.stringify({\n    name: 'CI/CD Pipeline',\n    scopes: ['aircraft', 'alerts'],\n    expires_at: '2025-12-31T23:59:59Z'\n  })\n});\n\nconst { key } = await response.json();\n// ⚠️ Store this immediately - shown only once!\nconsole.log('API Key:', key);",
      "language": "javascript",
      "name": "JavaScript"
    },
    {
      "code": "from skyspy.models import APIKey\nfrom django.utils import timezone\nfrom datetime import timedelta\n\napi_key = APIKey.objects.create(\n    user=user,\n    name=\"Read-only Aircraft Data\",\n    scopes=[\"aircraft\", \"history\"],\n    expires_at=timezone.now() + timedelta(days=90)\n)",
      "language": "python",
      "name": "Django"
    }
  ]
}
[/block]

#### 📤 Using API Keys

[block:code]
{
  "codes": [
    {
      "code": "# Using Authorization header (recommended)\ncurl https://your-skyspy-instance/api/v1/aircraft \\\n  -H \"Authorization: ApiKey sk_a1B2c3D4e5F6g7H8i9J0...\"\n\n# Using X-API-Key header\ncurl https://your-skyspy-instance/api/v1/aircraft \\\n  -H \"X-API-Key: sk_a1B2c3D4e5F6g7H8i9J0...\"",
      "language": "bash",
      "name": "cURL"
    },
    {
      "code": "const API_KEY = 'sk_a1B2c3D4e5F6g7H8i9J0...';\n\n// Using Authorization header\nconst response = await fetch('https://your-skyspy-instance/api/v1/aircraft', {\n  headers: {\n    'Authorization': `ApiKey ${API_KEY}`\n  }\n});\n\n// Or using X-API-Key header\nconst response2 = await fetch('https://your-skyspy-instance/api/v1/aircraft', {\n  headers: {\n    'X-API-Key': API_KEY\n  }\n});",
      "language": "javascript",
      "name": "JavaScript"
    }
  ]
}
[/block]

#### 📋 Scope-Based Access

| Scope | Access Granted | Example Endpoints |
|-------|----------------|-------------------|
| 🛫 `aircraft` | Aircraft tracking data | `/api/v1/aircraft/*` |
| 🔔 `alerts` | Alert rules and history | `/api/v1/alerts/*` |
| ⚠️ `safety` | Safety event data | `/api/v1/safety/*` |
| 🎵 `audio` | Audio transmissions | `/api/v1/audio/*` |
| 📡 `acars` | ACARS messages | `/api/v1/acars/*` |
| 📜 `history` | Historical data | `/api/v1/history/*` |
| 🖥️ `system` | System status and metrics | `/api/v1/system/*` |

---

### 3️⃣ OIDC/SSO Authentication

SkySpy supports OpenID Connect (OIDC) for enterprise single sign-on integration with identity providers like Okta, Auth0, Azure AD, and Keycloak.

#### ⚙️ Configuration

```bash
# OIDC Settings
OIDC_ENABLED=true
OIDC_PROVIDER_URL=https://your-idp.example.com
OIDC_CLIENT_ID=skyspy-client-id
OIDC_CLIENT_SECRET=your-client-secret
OIDC_PROVIDER_NAME=Corporate SSO       # Display name for UI
OIDC_SCOPES=openid profile email groups
OIDC_DEFAULT_ROLE=viewer               # Role for new OIDC users
```

#### 🔄 OIDC Authentication Flow

```mermaid
sequenceDiagram
    autonumber
    participant User as 👤 User
    participant App as 🖥️ SkySpy Frontend
    participant API as ⚙️ SkySpy Backend
    participant IdP as 🏢 Identity Provider

    User->>App: Click "Login with SSO"
    App->>API: GET /auth/oidc/authorize
    API-->>App: Authorization URL + state
    App->>IdP: Redirect to IdP login

    Note over IdP: User authenticates<br/>with corporate credentials

    IdP->>User: Login form
    User->>IdP: Enter credentials
    IdP-->>App: Redirect with auth code
    App->>API: GET /auth/oidc/callback?code=xxx

    API->>IdP: Exchange code for tokens
    IdP-->>API: ID token + access token
    API->>IdP: Fetch user info
    IdP-->>API: User claims (email, groups, etc.)

    API->>API: Create/update user<br/>Map claims to roles
    API-->>App: SkySpy JWT tokens + user
    App->>App: Store tokens, redirect to dashboard
```

#### 🗺️ Claim-Based Role Mapping

Map your IdP groups to SkySpy roles automatically:

```json
{
  "name": "Admin Group Mapping",
  "claim_name": "groups",
  "match_type": "exact",
  "claim_value": "skyspy-admins",
  "role": "admin",
  "priority": 10,
  "is_active": true
}
```

| Match Type | Description | Example |
|------------|-------------|---------|
| `exact` | Claim value must match exactly | `"skyspy-admins"` matches `"skyspy-admins"` |
| `contains` | Claim must contain the string | `"admin"` matches `"skyspy-admins"` |
| `regex` | Claim must match the pattern | `"skyspy-.*"` matches `"skyspy-operators"` |

[block:callout]
{
  "type": "danger",
  "title": "🚨 Security Warning",
  "body": "**Email linking** (`OIDC_ALLOW_EMAIL_LINKING`) is disabled by default. Enabling it allows existing accounts to be linked to OIDC based on matching email addresses.\n\n**Risk**: An attacker who controls an OIDC provider with matching email addresses could gain access to existing accounts."
}
[/block]

---

### 4️⃣ Session-Based Authentication (Admin Only)

Django admin uses session-based authentication. This is separate from the API authentication system.

```python
# settings.py - Session configuration
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Lax'
SESSION_COOKIE_SECURE = True  # In production
```

---

## 👥 User Roles and Permissions

### 🎭 Role-Based Access Control (RBAC)

SkySpy implements RBAC with the following default roles:

[block:parameters]
{
  "data": {
    "h-0": "Role",
    "h-1": "Priority",
    "h-2": "Description",
    "h-3": "Typical User",
    "0-0": "👁️ `viewer`",
    "0-1": "10",
    "0-2": "Read-only access to allowed features",
    "0-3": "Public dashboards, guests",
    "1-0": "⚙️ `operator`",
    "1-1": "20",
    "1-2": "Create/manage own alerts, acknowledge safety events",
    "1-3": "Daily users, shift workers",
    "2-0": "📊 `analyst`",
    "2-1": "30",
    "2-2": "Extended access with export and transcription",
    "2-3": "Data analysts, researchers",
    "3-0": "🔧 `admin`",
    "3-1": "40",
    "3-2": "Full feature access with limited user management",
    "3-3": "Team leads, managers",
    "4-0": "👑 `superadmin`",
    "4-1": "100",
    "4-2": "Full access including user and role management",
    "4-3": "System administrators"
  },
  "cols": 4,
  "rows": 5
}
[/block]

### 📋 Permission Matrix

Permissions follow the format `feature.action`:

```
aircraft.view           # View aircraft data
aircraft.view_military  # View military aircraft
alerts.create          # Create alert rules
alerts.delete          # Delete alert rules
alerts.manage_all      # Manage all users' alerts
safety.acknowledge     # Acknowledge safety events
users.create           # Create new users
roles.edit             # Modify roles
```

### 🔐 Role Permission Comparison

[block:parameters]
{
  "data": {
    "h-0": "Permission",
    "h-1": "👁️ Viewer",
    "h-2": "⚙️ Operator",
    "h-3": "📊 Analyst",
    "h-4": "🔧 Admin",
    "h-5": "👑 Super",
    "0-0": "**aircraft.view**",
    "0-1": "✅",
    "0-2": "✅",
    "0-3": "✅",
    "0-4": "✅",
    "0-5": "✅",
    "1-0": "**aircraft.view_military**",
    "1-1": "❌",
    "1-2": "❌",
    "1-3": "✅",
    "1-4": "✅",
    "1-5": "✅",
    "2-0": "**alerts.view**",
    "2-1": "✅",
    "2-2": "✅",
    "2-3": "✅",
    "2-4": "✅",
    "2-5": "✅",
    "3-0": "**alerts.create**",
    "3-1": "❌",
    "3-2": "✅",
    "3-3": "✅",
    "3-4": "✅",
    "3-5": "✅",
    "4-0": "**alerts.manage_all**",
    "4-1": "❌",
    "4-2": "❌",
    "4-3": "❌",
    "4-4": "✅",
    "4-5": "✅",
    "5-0": "**safety.acknowledge**",
    "5-1": "❌",
    "5-2": "✅",
    "5-3": "✅",
    "5-4": "✅",
    "5-5": "✅",
    "6-0": "**audio.transcribe**",
    "6-1": "❌",
    "6-2": "❌",
    "6-3": "✅",
    "6-4": "✅",
    "6-5": "✅",
    "7-0": "**history.export**",
    "7-1": "❌",
    "7-2": "❌",
    "7-3": "✅",
    "7-4": "✅",
    "7-5": "✅",
    "8-0": "**users.view**",
    "8-1": "❌",
    "8-2": "❌",
    "8-3": "❌",
    "8-4": "✅",
    "8-5": "✅",
    "9-0": "**users.create**",
    "9-1": "❌",
    "9-2": "❌",
    "9-3": "❌",
    "9-4": "❌",
    "9-5": "✅",
    "10-0": "**roles.edit**",
    "10-1": "❌",
    "10-2": "❌",
    "10-3": "❌",
    "10-4": "❌",
    "10-5": "✅"
  },
  "cols": 6,
  "rows": 11
}
[/block]

### 🆕 Creating Custom Roles

[block:code]
{
  "codes": [
    {
      "code": "curl -X POST https://your-skyspy-instance/api/v1/roles \\\n  -H \"Authorization: Bearer <admin-token>\" \\\n  -H \"Content-Type: application/json\" \\\n  -d '{\n    \"name\": \"shift_supervisor\",\n    \"display_name\": \"Shift Supervisor\",\n    \"description\": \"Can manage alerts and acknowledge safety events\",\n    \"permissions\": [\n      \"aircraft.view\",\n      \"aircraft.view_details\",\n      \"alerts.view\",\n      \"alerts.create\",\n      \"alerts.edit\",\n      \"alerts.delete\",\n      \"safety.view\",\n      \"safety.acknowledge\",\n      \"safety.manage\"\n    ],\n    \"priority\": 25\n  }'",
      "language": "bash",
      "name": "cURL"
    },
    {
      "code": "const newRole = await authFetch('/api/v1/roles', {\n  method: 'POST',\n  headers: { 'Content-Type': 'application/json' },\n  body: JSON.stringify({\n    name: 'shift_supervisor',\n    display_name: 'Shift Supervisor',\n    description: 'Can manage alerts and acknowledge safety events',\n    permissions: [\n      'aircraft.view',\n      'aircraft.view_details',\n      'alerts.view',\n      'alerts.create',\n      'alerts.edit',\n      'alerts.delete',\n      'safety.view',\n      'safety.acknowledge',\n      'safety.manage'\n    ],\n    priority: 25\n  })\n});",
      "language": "javascript",
      "name": "JavaScript"
    }
  ]
}
[/block]

### ⏰ Role Assignment with Expiration

Roles can be assigned with optional expiration for temporary access:

```bash
curl -X POST https://your-skyspy-instance/api/v1/user-roles \
  -H "Authorization: Bearer <admin-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "user": 42,
    "role": 3,
    "expires_at": "2024-02-01T00:00:00Z"
  }'
```

[block:callout]
{
  "type": "info",
  "title": "💡 Tip: Temporary Access",
  "body": "Use role expiration for:\n- **Contractors** with limited engagement periods\n- **Trainees** who need elevated access during onboarding\n- **Incident response** requiring temporary admin privileges"
}
[/block]

---

## 🎛️ Feature-Based Access Control

Each feature can be configured with independent access levels:

### 📊 Access Levels

| Level | Description | Icon |
|-------|-------------|------|
| `public` | No authentication required | 🟢 |
| `authenticated` | Any logged-in user | 🟡 |
| `permission` | Specific permission required | 🔴 |

### ⚙️ Configuration Example

```bash
curl -X PATCH https://your-skyspy-instance/api/v1/feature-access/aircraft \
  -H "Authorization: Bearer <admin-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "read_access": "public",
    "write_access": "permission",
    "is_enabled": true
  }'
```

### 🗺️ Hybrid Mode Setup Example

```json
{
  "aircraft": {
    "read_access": "public",      // 🟢 Anyone can view
    "write_access": "permission", // 🔴 Requires permission
    "is_enabled": true
  },
  "alerts": {
    "read_access": "authenticated", // 🟡 Must be logged in
    "write_access": "permission",   // 🔴 Requires permission
    "is_enabled": true
  },
  "safety": {
    "read_access": "authenticated", // 🟡 Must be logged in
    "write_access": "permission",   // 🔴 Requires permission
    "is_enabled": true
  },
  "users": {
    "read_access": "permission",  // 🔴 Admin only
    "write_access": "permission", // 🔴 Admin only
    "is_enabled": true
  }
}
```

---

## 🔌 WebSocket Authentication

WebSocket connections support both JWT tokens and API keys for real-time data streaming.

### 🔐 WebSocket Authentication Flow

```mermaid
sequenceDiagram
    autonumber
    participant Client as 🖥️ Client
    participant WS as 🔌 WebSocket Server
    participant Auth as 🔐 Auth Middleware
    participant Consumer as 📡 Consumer

    Client->>WS: Connect with token<br/>(Sec-WebSocket-Protocol)
    WS->>Auth: Validate token

    alt Token Valid ✅
        Auth-->>WS: User authenticated
        WS->>Consumer: Accept connection
        Consumer-->>Client: Connection accepted

        loop Real-time Updates
            Consumer->>Client: Aircraft data
            Consumer->>Client: Alert notifications
        end
    else Token Invalid ❌
        Auth-->>WS: Authentication failed
        WS-->>Client: Close (4001)
    end
```

### 🔧 Authentication Methods

[block:callout]
{
  "type": "success",
  "title": "✅ Recommended: Sec-WebSocket-Protocol Header",
  "body": "```javascript\nconst ws = new WebSocket('wss://your-skyspy/ws/aircraft', ['Bearer', accessToken]);\n```"
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "⚠️ Not Recommended: Query String",
  "body": "```javascript\nconst ws = new WebSocket('wss://your-skyspy/ws/aircraft?token=eyJ...');\n```\n\n**Why?** Tokens may appear in server logs and browser history."
}
[/block]

### 📋 Topic-Based Permissions

| Topic | Required Permission | Description |
|-------|---------------------|-------------|
| 🛫 `aircraft` | `aircraft.view` | Live aircraft positions |
| 🎖️ `military` | `aircraft.view_military` | Military aircraft data |
| 🔔 `alerts` | `alerts.view` | Alert notifications |
| ⚠️ `safety` | `safety.view` | Safety events |
| 📡 `acars` | `acars.view` | ACARS messages |
| 🎵 `audio` | `audio.view` | Audio stream notifications |
| 🖥️ `system` | `system.view_status` | System status updates |

### ❌ Handling Connection Rejection

```javascript
const ws = new WebSocket('wss://your-skyspy/ws/aircraft', ['Bearer', token]);

ws.onclose = (event) => {
  switch (event.code) {
    case 4001:
      console.error('🔐 Authentication failed - invalid or expired token');
      // Trigger re-authentication
      break;
    case 4003:
      console.error('🚫 Permission denied - insufficient access');
      break;
    default:
      console.log('Connection closed:', event.code);
  }
};
```

---

## 🖥️ Frontend Authentication Flow

The React frontend uses the `AuthContext` provider for authentication state management.

### 📦 AuthContext API

```jsx
import { useAuth } from '../contexts/AuthContext';

function MyComponent() {
  const {
    // 📊 State
    status,           // 'loading' | 'anonymous' | 'authenticated'
    user,             // Current user object
    config,           // Auth configuration
    error,            // Last error message
    isLoading,        // Boolean shorthand
    isAuthenticated,  // Boolean shorthand
    isAnonymous,      // Boolean shorthand

    // 🎬 Actions
    login,            // (username, password) => Promise
    logout,           // () => Promise
    loginWithOIDC,    // () => Promise
    refreshAccessToken, // () => Promise<boolean>
    authFetch,        // Authenticated fetch wrapper

    // 🔐 Permission checks
    hasPermission,    // (permission) => boolean
    hasAnyPermission, // ([permissions]) => boolean
    hasAllPermissions,// ([permissions]) => boolean
    canAccessFeature, // (feature, action?) => boolean

    // 🔑 Token access
    getAccessToken,   // () => string | null

    // ❌ Error handling
    clearError,       // () => void
  } = useAuth();
}
```

### 🔐 Frontend Login Flow

```mermaid
flowchart LR
    subgraph Login["🔐 Login"]
        A[User enters credentials] --> B[Call login]
        B --> C{Success?}
        C -->|Yes| D[Store tokens]
        C -->|No| E[Show error]
        D --> F[Redirect to dashboard]
    end
```

[block:code]
{
  "codes": [
    {
      "code": "const { login, error } = useAuth();\n\nasync function handleLogin(username, password) {\n  const result = await login(username, password);\n  if (result.success) {\n    navigate('/dashboard');\n  } else {\n    console.error(result.error);\n  }\n}",
      "language": "javascript",
      "name": "Standard Login"
    },
    {
      "code": "const { loginWithOIDC, config } = useAuth();\n\nasync function handleOIDCLogin() {\n  try {\n    const result = await loginWithOIDC();\n    if (result.success) {\n      navigate('/dashboard');\n    }\n  } catch (err) {\n    // User closed popup or login timed out\n    console.error(err.message);\n  }\n}\n\n// Show OIDC button if enabled\n{config.oidcEnabled && (\n  <button onClick={handleOIDCLogin}>\n    Login with {config.oidcProviderName}\n  </button>\n)}",
      "language": "javascript",
      "name": "OIDC/SSO Login"
    }
  ]
}
[/block]

### 🛡️ Permission Checking

```jsx
const { hasPermission, canAccessFeature } = useAuth();

// Check specific permission
if (hasPermission('alerts.create')) {
  return <CreateAlertButton />;
}

// Check feature access
if (canAccessFeature('safety', 'write')) {
  return <AcknowledgeButton />;
}

// Check multiple permissions
if (hasAllPermissions(['alerts.view', 'alerts.edit'])) {
  return <AlertManagement />;
}
```

### 🔄 Automatic Token Refresh

```
┌────────────────────────────────────────────────────────────────────┐
│                    TOKEN REFRESH TIMELINE                          │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Token Created        Refresh Scheduled       Token Expires        │
│       │                      │                      │              │
│       ▼                      ▼                      ▼              │
│  ────────────────────────────────────────────────────►  Time       │
│  │                          │                      │               │
│  └──────── 59 min 30s ──────┘                      │               │
│                             └───── 30s buffer ─────┘               │
│                                                                    │
│  ✅ New token obtained before expiry = seamless user experience    │
└────────────────────────────────────────────────────────────────────┘
```

### 💾 Token Storage

| Key | Value | Description |
|-----|-------|-------------|
| `skyspy_access_token` | JWT access token | Used for API requests |
| `skyspy_refresh_token` | JWT refresh token | Used to obtain new access tokens |
| `skyspy_user` | Serialized user object | Cached user info |

---

## 🛡️ Security Best Practices

### ✅ Security Checklist

[block:callout]
{
  "type": "success",
  "title": "🔒 Production Security Checklist",
  "body": "Before going to production, verify these settings:\n\n**Environment & Secrets**\n- [ ] ✅ `DEBUG=false`\n- [ ] ✅ Strong, unique `DJANGO_SECRET_KEY`\n- [ ] ✅ Separate `JWT_SECRET_KEY` from Django secret\n- [ ] ✅ Secrets not committed to version control\n\n**Authentication**\n- [ ] ✅ `AUTH_MODE=hybrid` or `private`\n- [ ] ✅ Rate limiting enabled on auth endpoints\n- [ ] ✅ JWT token lifetimes appropriate for use case\n\n**Cookies & Transport**\n- [ ] ✅ `SESSION_COOKIE_SECURE=true`\n- [ ] ✅ `CSRF_COOKIE_SECURE=true`\n- [ ] ✅ HTTPS enforced\n- [ ] ✅ CORS restricted to known origins\n\n**API Keys**\n- [ ] ✅ Scoped to minimum required permissions\n- [ ] ✅ Expiration dates set\n- [ ] ✅ Query parameter auth disabled"
}
[/block]

### 🔧 Environment Variables

```bash
# Production settings
DEBUG=false
DJANGO_SECRET_KEY=<strong-random-key>
JWT_SECRET_KEY=<different-strong-key>
AUTH_MODE=hybrid

# Enable secure cookies
SESSION_COOKIE_SECURE=true
CSRF_COOKIE_SECURE=true
JWT_AUTH_COOKIE=true
```

### ⏱️ Rate Limiting

[block:parameters]
{
  "data": {
    "h-0": "Endpoint",
    "h-1": "Rate Limit",
    "h-2": "Purpose",
    "0-0": "`/api/v1/auth/login`",
    "0-1": "5/minute",
    "0-2": "Prevent brute force attacks",
    "1-0": "`/api/v1/auth/refresh`",
    "1-1": "5/minute",
    "1-2": "Prevent token abuse",
    "2-0": "Anonymous requests",
    "2-1": "100/minute",
    "2-2": "General protection",
    "3-0": "Authenticated requests",
    "3-1": "1000/minute",
    "3-2": "Fair usage"
  },
  "cols": 3,
  "rows": 4
}
[/block]

### 🔐 Token Security Best Practices

[block:parameters]
{
  "data": {
    "h-0": "Practice",
    "h-1": "Implementation",
    "h-2": "Why It Matters",
    "0-0": "🔑 **Separate JWT Secret**",
    "0-1": "Use different `JWT_SECRET_KEY` than `DJANGO_SECRET_KEY`",
    "0-2": "Limits blast radius if one key is compromised",
    "1-0": "⏱️ **Short Access Tokens**",
    "1-1": "Default 60 minutes, adjust as needed",
    "1-2": "Limits window for stolen tokens",
    "2-0": "🔄 **Token Rotation**",
    "2-1": "Refresh tokens blacklisted on use",
    "2-2": "Prevents replay attacks",
    "3-0": "🍪 **Secure Storage**",
    "3-1": "Use httpOnly cookies (`JWT_AUTH_COOKIE=true`)",
    "3-2": "Prevents XSS token theft"
  },
  "cols": 3,
  "rows": 4
}
[/block]

### 🔗 API Key Security

[block:callout]
{
  "type": "warning",
  "title": "⚠️ API Key Best Practices",
  "body": "1. **No Query Parameters** - API keys cannot be passed in URLs (prevents logging/leakage)\n2. **Hashed Storage** - Only SHA-256 hash stored in database\n3. **Scoped Access** - Limit API keys to required features only\n4. **Expiration** - Always set expiration dates on API keys\n5. **Rotation** - Rotate keys periodically and after team changes"
}
[/block]

### 🌐 CORS Configuration

```bash
CORS_ALLOW_ALL_ORIGINS=false
CORS_ALLOW_CREDENTIALS=true
CORS_ALLOWED_ORIGINS=https://your-frontend-domain.com
```

---

## 📚 Configuration Reference

### 🔐 Authentication Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `AUTH_MODE` | `hybrid` | Authentication mode (public/private/hybrid) |
| `LOCAL_AUTH_ENABLED` | `true` | Enable username/password login |
| `API_KEY_ENABLED` | `true` | Enable API key authentication |
| `JWT_SECRET_KEY` | `SECRET_KEY` | Key for signing JWTs |
| `JWT_ACCESS_TOKEN_LIFETIME_MINUTES` | `60` | Access token validity |
| `JWT_REFRESH_TOKEN_LIFETIME_DAYS` | `2` | Refresh token validity |
| `JWT_AUTH_COOKIE` | `false` | Store tokens in httpOnly cookies |

### 🌐 OIDC Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `OIDC_ENABLED` | `false` | Enable OIDC authentication |
| `OIDC_PROVIDER_URL` | - | Base URL of the OIDC provider |
| `OIDC_CLIENT_ID` | - | OAuth client ID |
| `OIDC_CLIENT_SECRET` | - | OAuth client secret |
| `OIDC_PROVIDER_NAME` | `SSO` | Display name for UI |
| `OIDC_SCOPES` | `openid profile email groups` | OAuth scopes to request |
| `OIDC_DEFAULT_ROLE` | `viewer` | Default role for new OIDC users |
| `OIDC_ALLOW_EMAIL_LINKING` | `false` | Allow linking by email ⚠️ |

### ⏱️ Rate Limiting Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `DEFAULT_THROTTLE_RATES.anon` | `100/minute` | Anonymous user rate limit |
| `DEFAULT_THROTTLE_RATES.user` | `1000/minute` | Authenticated user rate limit |
| Auth endpoints | `5/minute` | Login/refresh rate limit |

---

## 📡 API Endpoints Reference

### 🔐 Authentication Endpoints

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/v1/auth/config` | GET | ❌ | Get auth configuration |
| `/api/v1/auth/login` | POST | ❌ | Login with credentials |
| `/api/v1/auth/logout` | POST | ✅ | Logout and blacklist token |
| `/api/v1/auth/refresh` | POST | ❌ | Refresh access token |
| `/api/v1/auth/profile` | GET | ✅ | Get current user profile |
| `/api/v1/auth/profile` | PATCH | ✅ | Update current user profile |
| `/api/v1/auth/password` | POST | ✅ | Change password |
| `/api/v1/auth/oidc/authorize` | GET | ❌ | Get OIDC authorization URL |
| `/api/v1/auth/oidc/callback` | GET | ❌ | OIDC callback handler |
| `/api/v1/auth/permissions` | GET | ❌ | List all permissions |
| `/api/v1/auth/my-permissions` | GET | ✅ | Get current user permissions |

### 👥 User Management Endpoints

| Endpoint | Method | Permission | Description |
|----------|--------|------------|-------------|
| `/api/v1/users` | GET | `users.view` | List users |
| `/api/v1/users` | POST | `users.create` | Create user |
| `/api/v1/users/{id}` | GET | `users.view` | Get user details |
| `/api/v1/users/{id}` | PATCH | `users.edit` | Update user |
| `/api/v1/users/{id}` | DELETE | `users.delete` | Delete user |

### 🎭 Role Management Endpoints

| Endpoint | Method | Permission | Description |
|----------|--------|------------|-------------|
| `/api/v1/roles` | GET | `roles.view` | List roles |
| `/api/v1/roles` | POST | `roles.create` | Create role |
| `/api/v1/roles/{id}` | GET | `roles.view` | Get role details |
| `/api/v1/roles/{id}` | PATCH | `roles.edit` | Update role |
| `/api/v1/roles/{id}` | DELETE | `roles.delete` | Delete role |

### 🔗 API Key Management Endpoints

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/v1/api-keys` | GET | ✅ | List user's API keys |
| `/api/v1/api-keys` | POST | ✅ | Create API key |
| `/api/v1/api-keys/{id}` | DELETE | ✅ | Delete API key |

---

## 🔧 Troubleshooting

### ❓ Common Issues

[block:callout]
{
  "type": "danger",
  "title": "🔴 401 Unauthorized",
  "body": "**Possible causes:**\n- Token is expired\n- Invalid Authorization header format (should be `Bearer <token>`)\n- User account is deactivated\n\n**Solutions:**\n1. Check token expiration with a JWT decoder\n2. Verify header format: `Authorization: Bearer eyJ...`\n3. Confirm user `is_active=True` in database"
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "🟡 403 Forbidden",
  "body": "**Possible causes:**\n- User lacks required permission\n- Feature is disabled\n- API key scope doesn't include the feature\n\n**Solutions:**\n1. Check user permissions via `/api/v1/auth/my-permissions`\n2. Verify feature is enabled in admin\n3. Check API key scopes if using API key auth"
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "🔵 OIDC Login Failed",
  "body": "**Possible causes:**\n- Incorrect `OIDC_CLIENT_SECRET`\n- Redirect URIs not configured in IdP\n- Scopes not allowed by IdP\n\n**Solutions:**\n1. Double-check client secret in environment\n2. Add callback URL to IdP allowed redirects: `https://your-domain/api/v1/auth/oidc/callback`\n3. Verify scopes are enabled in IdP application settings"
}
[/block]

[block:callout]
{
  "type": "danger",
  "title": "🔴 WebSocket Connection Rejected (4001)",
  "body": "**Possible causes:**\n- Token is invalid or expired\n- Using query string instead of header\n- `WS_REJECT_INVALID_TOKENS` is enabled\n\n**Solutions:**\n1. Refresh token before connecting\n2. Use `Sec-WebSocket-Protocol` header: `new WebSocket(url, ['Bearer', token])`\n3. Check WebSocket middleware configuration"
}
[/block]

### 🐛 Debug Logging

Enable debug logging for authentication issues:

```python
LOGGING = {
    'loggers': {
        'skyspy.auth': {
            'level': 'DEBUG',
            'handlers': ['console'],
        },
    },
}
```

---

## 📖 Next Steps

[block:callout]
{
  "type": "info",
  "title": "🚀 Ready to integrate?",
  "body": "- **Quick Start**: Try the `/api/v1/auth/login` endpoint with your credentials\n- **API Keys**: Create a scoped API key for your integration\n- **Enterprise**: Configure OIDC with your identity provider\n- **Support**: Check our GitHub issues or contact support"
}
[/block]
