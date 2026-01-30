---
title: Contributing Guide
hidden: false
---

# 🤝 Contributing to SkySpy

Welcome to the SkySpy contributor community! We're thrilled you're interested in helping build the future of aircraft tracking.

> 📖 **New to Contributing?** This guide covers everything you need to know, from setting up your environment to getting your first PR merged.

---

## 🚀 Quick Start

[block:embed]
{
  "html": false,
  "url": "https://github.com/your-org/skyspy",
  "title": "SkySpy Repository",
  "favicon": "https://github.com/favicon.ico"
}
[/block]

[block:html]
{
  "html": "<div style=\"display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 20px;\">\n  <img src=\"https://img.shields.io/badge/PRs-welcome-brightgreen.svg\" alt=\"PRs Welcome\">\n  <img src=\"https://img.shields.io/badge/License-MIT-blue.svg\" alt=\"License\">\n  <img src=\"https://img.shields.io/badge/code%20style-black-000000.svg\" alt=\"Code Style: Black\">\n  <img src=\"https://img.shields.io/badge/code%20style-prettier-ff69b4.svg\" alt=\"Code Style: Prettier\">\n</div>"
}
[/block]

[block:callout]
{
  "type": "success",
  "title": "Ways to Contribute",
  "body": "🐛 **Bug Reports** - Found an issue? Let us know!\n💡 **Feature Ideas** - Have a great idea? We'd love to hear it!\n💻 **Code** - Submit PRs for bug fixes or new features\n📝 **Documentation** - Help improve our docs\n🧪 **Testing** - Write tests and improve coverage\n👀 **Code Review** - Review open pull requests"
}
[/block]

---

## 🗺️ Project Architecture

SkySpy is a sophisticated real-time ADS-B aircraft tracking platform with multiple components:

[block:parameters]
{
  "data": {
    "h-0": "Component",
    "h-1": "Technology",
    "h-2": "Location",
    "h-3": "Badge",
    "0-0": "🐍 Backend API",
    "0-1": "Django 5.0+ / Django Channels",
    "0-2": "`skyspy_django/`",
    "0-3": "![Python](https://img.shields.io/badge/python-3.12+-blue?logo=python&logoColor=white)",
    "1-0": "⚛️ Web Dashboard",
    "1-1": "React 18 / Vite 5",
    "1-2": "`web/`",
    "1-3": "![React](https://img.shields.io/badge/react-18+-61dafb?logo=react&logoColor=white)",
    "2-0": "🔷 CLI Tools",
    "2-1": "Go 1.23+",
    "2-2": "`skyspy-go/`",
    "2-3": "![Go](https://img.shields.io/badge/go-1.23+-00add8?logo=go&logoColor=white)",
    "3-0": "📦 Shared Libraries",
    "3-1": "Python",
    "3-2": "`skyspy_common/`",
    "3-3": "![Python](https://img.shields.io/badge/python-lib-blue?logo=python&logoColor=white)",
    "4-0": "🧪 Mock Services",
    "4-1": "Docker",
    "4-2": "`test/`",
    "4-3": "![Docker](https://img.shields.io/badge/docker-compose-2496ed?logo=docker&logoColor=white)"
  },
  "cols": 4,
  "rows": 5
}
[/block]

---

## 📋 Before You Start

[block:callout]
{
  "type": "warning",
  "title": "Prerequisites Checklist",
  "body": "- [ ] Read through this contributing guide\n- [ ] Check existing issues and PRs for similar work\n- [ ] For major changes, open an issue first to discuss\n- [ ] Sign off on the Developer Certificate of Origin (DCO)"
}
[/block]

---

## 🛠️ Development Environment Setup

### Step 1: Install Prerequisites

[block:parameters]
{
  "data": {
    "h-0": "Tool",
    "h-1": "Version",
    "h-2": "Purpose",
    "h-3": "Required",
    "0-0": "🐳 Docker",
    "0-1": "Latest",
    "0-2": "Container orchestration",
    "0-3": "✅ Yes",
    "1-0": "🐳 Docker Compose",
    "1-1": "v2+",
    "1-2": "Multi-container development",
    "1-3": "✅ Yes",
    "2-0": "🐍 Python",
    "2-1": "3.12+",
    "2-2": "Backend development",
    "2-3": "✅ Yes",
    "3-0": "📦 Node.js",
    "3-1": "20+",
    "3-2": "Frontend development",
    "3-3": "✅ Yes",
    "4-0": "🔷 Go",
    "4-1": "1.23+",
    "4-2": "CLI development",
    "4-3": "⚪ Optional",
    "5-0": "🔀 Git",
    "5-1": "Latest",
    "5-2": "Version control",
    "5-3": "✅ Yes"
  },
  "cols": 4,
  "rows": 6
}
[/block]

### Step 2: Clone & Configure

```bash
# Clone the repository
git clone https://github.com/your-org/skyspy.git
cd skyspy

# Copy environment template
cp .env.example .env
```

[block:callout]
{
  "type": "info",
  "title": "🔧 Essential Environment Variables",
  "body": "Edit `.env` with your configuration:\n\n```bash\n# Required: Feeder location (your antenna position)\nFEEDER_LAT=47.9377\nFEEDER_LON=-121.9687\n\n# Required: ADS-B receiver host\nULTRAFEEDER_HOST=ultrafeeder\nULTRAFEEDER_PORT=80\n\n# For development, use public auth mode\nAUTH_MODE=public\n```"
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "📄 Full Configuration",
  "body": "The `.env.example` file contains extensive documentation for all available options including authentication, OIDC SSO, notifications, transcription, and external data sources."
}
[/block]

### Step 3: Database Setup

[block:callout]
{
  "type": "success",
  "title": "💡 Recommended: Docker Setup",
  "body": "The development environment automatically provisions PostgreSQL and Redis:\n\n```bash\n# Start all services including database\nmake dev\n\n# Database is automatically migrated on startup\n```"
}
[/block]

<details>
<summary>📌 Alternative: Local PostgreSQL</summary>

If running PostgreSQL locally:

```bash
# Set your database URL
export DATABASE_URL=postgresql://user:password@localhost:5432/skyspy

# Run migrations
cd skyspy_django
python manage.py migrate
```

</details>

### Step 4: Start Development Services

[block:code]
{
  "codes": [
    {
      "code": "# Start full development environment with mock data\nmake dev\n\n# Services available:\n#   Dashboard:     http://localhost:3000\n#   Django API:    http://localhost:8000\n#   PostgreSQL:    localhost:5432 (via pgbouncer)\n#   Redis:         localhost:6379\n#   Ultrafeeder:   http://localhost:18080\n#   Dump978:       http://localhost:18081\n\n# View logs\nmake dev-logs\n\n# Stop services\nmake dev-down",
      "language": "bash",
      "name": "Quick Start (Docker)"
    }
  ]
}
[/block]

<details>
<summary>🔧 Running Components Individually</summary>

#### 🐍 Backend API (Django)

```bash
cd skyspy_django

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -e ../skyspy_common
pip install -r requirements.txt

# Run migrations
python manage.py migrate

# Start development server
python manage.py runserver 0.0.0.0:8000

# Or with Daphne for WebSocket support
daphne -b 0.0.0.0 -p 8000 skyspy.asgi:application
```

#### ⚡ Celery Worker (Background Tasks)

```bash
# In a separate terminal
cd skyspy_django
celery -A skyspy worker --loglevel=info --pool=gevent --concurrency=10
```

#### ⏰ Celery Beat (Scheduled Tasks)

```bash
# In a separate terminal
cd skyspy_django
celery -A skyspy beat --loglevel=info
```

#### ⚛️ Frontend Dashboard (React)

```bash
cd web

# Install dependencies
npm install

# Start development server
npm run dev

# Dashboard available at http://localhost:3000
```

#### 🔷 Go CLI

```bash
cd skyspy-go

# Download dependencies
make deps

# Build binary
make build

# Run
./bin/skyspy --help
```

</details>

---

## 🎨 Code Style Guidelines

### 🐍 Python (Backend)

[block:html]
{
  "html": "<div style=\"display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 16px;\">\n  <img src=\"https://img.shields.io/badge/code%20style-black-000000.svg\" alt=\"Code Style: Black\">\n  <img src=\"https://img.shields.io/badge/linting-ruff-261230.svg?logo=ruff\" alt=\"Linting: Ruff\">\n  <img src=\"https://img.shields.io/badge/typing-mypy-blue.svg\" alt=\"Typing: mypy\">\n  <img src=\"https://img.shields.io/badge/python-3.12+-blue.svg?logo=python&logoColor=white\" alt=\"Python 3.12+\">\n</div>"
}
[/block]

[block:parameters]
{
  "data": {
    "h-0": "Tool",
    "h-1": "Purpose",
    "h-2": "Command",
    "0-0": "Ruff",
    "0-1": "Linting + import sorting",
    "0-2": "`ruff check .`",
    "1-0": "Black",
    "1-1": "Code formatting",
    "1-2": "`black .`",
    "2-0": "mypy",
    "2-1": "Static type checking",
    "2-2": "`mypy .`"
  },
  "cols": 3,
  "rows": 3
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "⚙️ Configuration (pyproject.toml)",
  "body": "```toml\n[tool.ruff]\ntarget-version = \"py312\"\nline-length = 120\n\n[tool.black]\ntarget-version = [\"py312\"]\nline-length = 120\n```"
}
[/block]

**📏 Key Rules:**
- Line length: 120 characters maximum
- Use type hints for function signatures
- Follow PEP 8 naming conventions
- Avoid unused imports and variables
- Use `isort`-compatible import ordering

[block:code]
{
  "codes": [
    {
      "code": "# Check for issues\nruff check .\nblack --check --diff .\nmypy --ignore-missing-imports .\n\n# Auto-fix issues\nruff check --fix .\nblack .",
      "language": "bash",
      "name": "Python Linting Commands"
    }
  ]
}
[/block]

---

### ⚛️ JavaScript/React (Frontend)

[block:html]
{
  "html": "<div style=\"display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 16px;\">\n  <img src=\"https://img.shields.io/badge/code%20style-prettier-ff69b4.svg\" alt=\"Code Style: Prettier\">\n  <img src=\"https://img.shields.io/badge/linting-eslint-4B32C3.svg?logo=eslint\" alt=\"Linting: ESLint\">\n  <img src=\"https://img.shields.io/badge/react-18-61dafb.svg?logo=react&logoColor=white\" alt=\"React 18\">\n</div>"
}
[/block]

[block:parameters]
{
  "data": {
    "h-0": "Tool",
    "h-1": "Config File",
    "h-2": "Command",
    "0-0": "ESLint",
    "0-1": "`.eslintrc.cjs`",
    "0-2": "`npm run lint`",
    "1-0": "Prettier",
    "1-1": "`.prettierrc`",
    "1-2": "`npm run format`"
  },
  "cols": 3,
  "rows": 2
}
[/block]

<details>
<summary>📋 Prettier Configuration</summary>

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

</details>

**📏 ESLint Rules:**
- ✅ React Hooks rules enforced (`rules-of-hooks`, `exhaustive-deps`)
- ❌ No `console.log` (use `console.warn` or `console.error`)
- ✅ Prefer `const` over `let`
- ✅ Use strict equality (`===`)

[block:code]
{
  "codes": [
    {
      "code": "cd web\n\n# Check for issues\nnpm run lint\nnpm run format:check\n\n# Auto-fix issues\nnpm run lint:fix\nnpm run format",
      "language": "bash",
      "name": "JavaScript Linting Commands"
    }
  ]
}
[/block]

---

### 🔷 Go (CLI)

[block:html]
{
  "html": "<div style=\"display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 16px;\">\n  <img src=\"https://img.shields.io/badge/linting-golangci--lint-00add8.svg?logo=go&logoColor=white\" alt=\"Linting: golangci-lint\">\n  <img src=\"https://img.shields.io/badge/go-1.23+-00add8.svg?logo=go&logoColor=white\" alt=\"Go 1.23+\">\n</div>"
}
[/block]

**🔍 Enabled Linters:**

| Linter | Purpose |
|--------|---------|
| `errcheck` | Check for unchecked errors |
| `gosec` | Security-oriented checks |
| `gocyclo` | Cyclomatic complexity (max 20) |
| `dupl` | Duplicate code detection |
| `misspell` | Common misspellings |
| `revive` | Go best practices |

[block:code]
{
  "codes": [
    {
      "code": "cd skyspy-go\n\n# Run all linters\nmake lint\n\n# Format code\nmake fmt\n\n# Check formatting\nmake fmt-check\n\n# Run go vet\nmake vet",
      "language": "bash",
      "name": "Go Linting Commands"
    }
  ]
}
[/block]

---

## 🔀 Git Workflow

### Branch Strategy

```mermaid
gitGraph
    commit id: "main"
    branch develop
    checkout develop
    commit id: "integration"
    branch feature/new-alerts
    checkout feature/new-alerts
    commit id: "add alerts"
    commit id: "add tests"
    checkout develop
    merge feature/new-alerts
    branch fix/websocket-bug
    checkout fix/websocket-bug
    commit id: "fix bug"
    checkout develop
    merge fix/websocket-bug
    checkout main
    merge develop tag: "v2.6.0"
```

[block:parameters]
{
  "data": {
    "h-0": "Branch",
    "h-1": "Purpose",
    "h-2": "Protection",
    "0-0": "`main`",
    "0-1": "🚀 Production-ready code",
    "0-2": "🔒 Protected, requires PR",
    "1-0": "`develop`",
    "1-1": "🔧 Integration branch",
    "1-2": "🔒 Protected, requires PR",
    "2-0": "`feature/*`",
    "2-1": "✨ New features",
    "2-2": "🔓 None",
    "3-0": "`fix/*`",
    "3-1": "🐛 Bug fixes",
    "3-2": "🔓 None",
    "4-0": "`docs/*`",
    "4-1": "📝 Documentation updates",
    "4-2": "🔓 None",
    "5-0": "`refactor/*`",
    "5-1": "♻️ Code refactoring",
    "5-2": "🔓 None"
  },
  "cols": 3,
  "rows": 6
}
[/block]

### Creating a Branch

```bash
# Update your local main
git checkout main
git pull origin main

# Create a feature branch
git checkout -b feature/amazing-feature

# Or for bug fixes
git checkout -b fix/bug-description
```

---

## 📝 Commit Messages

Follow **Conventional Commits** for clear history:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Commit Types

[block:parameters]
{
  "data": {
    "h-0": "Type",
    "h-1": "Emoji",
    "h-2": "Description",
    "h-3": "Example",
    "0-0": "`feat`",
    "0-1": "✨",
    "0-2": "New feature",
    "0-3": "Add proximity alert configuration",
    "1-0": "`fix`",
    "1-1": "🐛",
    "1-2": "Bug fix",
    "1-3": "Handle reconnection on timeout",
    "2-0": "`docs`",
    "2-1": "📝",
    "2-2": "Documentation only",
    "2-3": "Update API endpoint examples",
    "3-0": "`style`",
    "3-1": "🎨",
    "3-2": "Formatting, no code change",
    "3-3": "Format with Black",
    "4-0": "`refactor`",
    "4-1": "♻️",
    "4-2": "Code restructuring",
    "4-3": "Extract safety monitoring logic",
    "5-0": "`test`",
    "5-1": "🧪",
    "5-2": "Adding or updating tests",
    "5-3": "Add WebSocket integration tests",
    "6-0": "`chore`",
    "6-1": "🔧",
    "6-2": "Build process, tooling",
    "6-3": "Update CI workflow",
    "7-0": "`perf`",
    "7-1": "⚡",
    "7-2": "Performance improvement",
    "7-3": "Optimize database queries"
  },
  "cols": 4,
  "rows": 8
}
[/block]

### Commit Examples

[block:code]
{
  "codes": [
    {
      "code": "# Feature\ngit commit -m \"feat(alerts): add proximity alert distance configuration\"\n\n# Bug fix\ngit commit -m \"fix(websocket): handle reconnection on network timeout\"\n\n# Documentation\ngit commit -m \"docs(api): update alert rule endpoint examples\"\n\n# Refactoring\ngit commit -m \"refactor(services): extract safety monitoring logic\"",
      "language": "bash",
      "name": "Example Commits"
    }
  ]
}
[/block]

---

## 🔄 Pull Request Process

### PR Workflow

```mermaid
flowchart LR
    A[🔧 Make Changes] --> B[🧪 Run Tests]
    B --> C[📤 Push Branch]
    C --> D[📝 Create PR]
    D --> E[🤖 CI Checks]
    E --> F{Pass?}
    F -->|No| A
    F -->|Yes| G[👀 Code Review]
    G --> H{Approved?}
    H -->|No| A
    H -->|Yes| I[✅ Merge]
```

### ✅ PR Checklist

[block:callout]
{
  "type": "warning",
  "title": "Before Submitting Your PR",
  "body": "- [ ] 🧪 All tests pass locally (`make test`)\n- [ ] 🎨 Code follows project style guidelines\n- [ ] 📝 Documentation updated (if needed)\n- [ ] 🚫 No `console.log` or debug code\n- [ ] 🔒 No hardcoded secrets or credentials\n- [ ] ⚡ Code is performant (no N+1 queries)\n- [ ] ✨ New functionality has tests"
}
[/block]

### Running Tests Before PR

[block:code]
{
  "codes": [
    {
      "code": "# Full test suite in Docker (recommended)\nmake test\n\n# Backend tests only\ncd skyspy_django && pytest\n\n# Frontend linting\ncd web && npm run lint\n\n# Go tests\ncd skyspy-go && make test",
      "language": "bash",
      "name": "Test Commands"
    },
    {
      "code": "# Python\nruff check . && black --check .\n\n# JavaScript\ncd web && npm run lint && npm run format:check\n\n# Go\ncd skyspy-go && make lint",
      "language": "bash",
      "name": "Lint Commands"
    }
  ]
}
[/block]

### PR Description Template

```markdown
## Summary

Brief description of changes and motivation.

## Changes

- Bullet points of specific changes
- Include file paths for major modifications

## Testing

- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed

## Documentation

- [ ] Code comments updated
- [ ] README/docs updated (if applicable)
- [ ] API documentation updated (if applicable)

## Screenshots (if UI changes)

Before/after screenshots for visual changes.
```

### PR Requirements

[block:parameters]
{
  "data": {
    "h-0": "Requirement",
    "h-1": "Description",
    "h-2": "Status",
    "0-0": "🤖 CI Pipeline",
    "0-1": "All automated checks pass",
    "0-2": "Required",
    "1-0": "👀 Code Review",
    "1-1": "At least one approving review",
    "1-2": "Required",
    "2-0": "📊 Coverage",
    "2-1": "Maintain 40% minimum (60% target)",
    "2-2": "Required",
    "3-0": "🔀 No Conflicts",
    "3-1": "Mergeable with target branch",
    "3-2": "Required"
  },
  "cols": 3,
  "rows": 4
}
[/block]

---

## 👀 Code Review Process

### Review Flow

```mermaid
flowchart TB
    subgraph Author
        A1[Submit PR] --> A2[Address Feedback]
        A2 --> A3[Push Updates]
    end

    subgraph Reviewer
        R1[Review Code] --> R2{Issues Found?}
        R2 -->|Yes| R3[Request Changes]
        R2 -->|No| R4[Approve PR]
    end

    subgraph CI
        C1[Run Tests] --> C2[Run Linters]
        C2 --> C3[Security Scan]
    end

    A1 --> R1
    A1 --> C1
    R3 --> A2
    A3 --> R1
    R4 --> M[Merge]
    C3 --> M
```

### For Authors

[block:callout]
{
  "type": "info",
  "title": "✍️ Before Requesting Review",
  "body": "- [ ] Code follows project style guidelines\n- [ ] All tests pass locally\n- [ ] New functionality has tests\n- [ ] Documentation updated\n- [ ] No debug code or console.log statements\n- [ ] No hardcoded secrets or credentials\n- [ ] Error handling is appropriate\n- [ ] Code is performant (no N+1 queries)"
}
[/block]

### For Reviewers

[block:callout]
{
  "type": "success",
  "title": "👀 Review Checklist",
  "body": "**Functionality**\n- [ ] Code does what it claims to do\n- [ ] Edge cases are handled\n- [ ] Error conditions handled gracefully\n\n**Code Quality**\n- [ ] Code is readable and maintainable\n- [ ] No code duplication\n- [ ] Functions are focused and appropriately sized\n\n**Testing**\n- [ ] Tests are meaningful and comprehensive\n- [ ] Test coverage is adequate\n\n**Security**\n- [ ] No SQL injection vulnerabilities\n- [ ] No XSS vulnerabilities\n- [ ] Auth/authz is correct"
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "💬 Review Etiquette",
  "body": "- Be constructive and specific\n- Explain the \"why\" behind suggestions\n- Distinguish between required changes and suggestions\n- Approve when satisfied, request changes when needed"
}
[/block]

---

## 🐛 Issue Guidelines

### Bug Reports

[block:callout]
{
  "type": "danger",
  "title": "🐛 Bug Report Template",
  "body": "**Title:** `[BUG] Brief description`\n\n**Include:**\n- Environment (OS, Docker version, Browser)\n- SkySpy version or commit hash\n- Steps to reproduce\n- Expected vs actual behavior\n- Relevant logs/screenshots"
}
[/block]

### Feature Requests

[block:callout]
{
  "type": "info",
  "title": "💡 Feature Request Template",
  "body": "**Title:** `[FEATURE] Brief description`\n\n**Include:**\n- Problem statement\n- Proposed solution\n- Alternatives considered\n- Mockups or examples (if applicable)"
}
[/block]

### Issue Labels

[block:parameters]
{
  "data": {
    "h-0": "Label",
    "h-1": "Description",
    "h-2": "Color",
    "0-0": "`bug`",
    "0-1": "🐛 Something isn't working",
    "0-2": "🔴 Red",
    "1-0": "`enhancement`",
    "1-1": "✨ New feature or improvement",
    "1-2": "🔵 Blue",
    "2-0": "`documentation`",
    "2-1": "📝 Documentation updates",
    "2-2": "🟢 Green",
    "3-0": "`good first issue`",
    "3-1": "👋 Good for newcomers",
    "3-2": "🟣 Purple",
    "4-0": "`help wanted`",
    "4-1": "🆘 Extra attention needed",
    "4-2": "🟡 Yellow",
    "5-0": "`priority: high`",
    "5-1": "🔥 Critical issue",
    "5-2": "🔴 Red"
  },
  "cols": 3,
  "rows": 6
}
[/block]

---

## 🚀 Release Process

### Semantic Versioning

```mermaid
flowchart LR
    subgraph Version[" "]
        direction TB
        M[MAJOR x.0.0] --- |Breaking changes| M1[API incompatible]
        N[MINOR 0.x.0] --- |Features| N1[Backward compatible]
        P[PATCH 0.0.x] --- |Fixes| P1[Backward compatible]
    end
```

| Version | Change Type | Example |
|---------|-------------|---------|
| **MAJOR** (x.0.0) | Breaking API changes | `1.0.0` → `2.0.0` |
| **MINOR** (0.x.0) | New features | `2.5.0` → `2.6.0` |
| **PATCH** (0.0.x) | Bug fixes | `2.6.0` → `2.6.1` |

### CI/CD Pipeline

```mermaid
flowchart LR
    subgraph Trigger
        PR[PR/Push]
        Tag[Git Tag]
    end

    subgraph Tests
        UT[Unit Tests]
        LT[Lint]
        E2E[E2E Tests]
        SEC[Security Scan]
    end

    subgraph Deploy
        BUILD[Build Images]
        PUSH[Push to GHCR]
    end

    PR --> UT --> LT --> E2E --> SEC
    Tag --> BUILD --> PUSH
```

[block:parameters]
{
  "data": {
    "h-0": "Stage",
    "h-1": "Trigger",
    "h-2": "Actions",
    "0-0": "🧪 Test",
    "0-1": "All PRs, pushes",
    "0-2": "Run pytest, coverage",
    "1-0": "🐍 Lint Python",
    "1-1": "All PRs, pushes",
    "1-2": "Ruff, Black, mypy",
    "2-0": "⚛️ Lint Frontend",
    "2-1": "All PRs, pushes",
    "2-2": "ESLint, Prettier",
    "3-0": "🔷 Test Go",
    "3-1": "All PRs, pushes",
    "3-2": "golangci-lint, go test",
    "4-0": "🎭 E2E Tests",
    "4-1": "After unit tests",
    "4-2": "Playwright tests",
    "5-0": "🔒 Security Scan",
    "5-1": "All PRs, pushes",
    "5-2": "Bandit, pip-audit, npm audit",
    "6-0": "🐳 Build & Push",
    "6-1": "Push to main/develop, tags",
    "6-2": "Multi-arch images to GHCR"
  },
  "cols": 3,
  "rows": 7
}
[/block]

### Docker Images

[block:callout]
{
  "type": "success",
  "title": "📦 Published Images",
  "body": "- `ghcr.io/{org}/skyspy` - Main API image\n- `ghcr.io/{org}/skyspy-rtl-airband-uploader` - RTL-Airband uploader\n- `ghcr.io/{org}/skyspy-1090-mock` - Mock ADS-B receiver\n- `ghcr.io/{org}/skyspy-acarshub-mock` - Mock ACARS hub"
}
[/block]

---

## 📚 Quick Reference

### Common Commands

[block:parameters]
{
  "data": {
    "h-0": "Task",
    "h-1": "Command",
    "0-0": "🚀 Start dev environment",
    "0-1": "`make dev`",
    "1-0": "📋 View logs",
    "1-1": "`make dev-logs`",
    "2-0": "🛑 Stop services",
    "2-1": "`make dev-down`",
    "3-0": "🧪 Run all tests",
    "3-1": "`make test`",
    "4-0": "🐍 Python lint",
    "4-1": "`ruff check . && black .`",
    "5-0": "⚛️ JS lint",
    "5-1": "`cd web && npm run lint`",
    "6-0": "🔷 Go lint",
    "6-1": "`cd skyspy-go && make lint`",
    "7-0": "🐳 Build images",
    "7-1": "`make build`"
  },
  "cols": 2,
  "rows": 8
}
[/block]

### Service URLs (Development)

[block:parameters]
{
  "data": {
    "h-0": "Service",
    "h-1": "URL",
    "h-2": "Port",
    "0-0": "🌐 Dashboard",
    "0-1": "http://localhost:3000",
    "0-2": "3000",
    "1-0": "🔌 Django API",
    "1-1": "http://localhost:8000",
    "1-2": "8000",
    "2-0": "🐘 PostgreSQL",
    "2-1": "localhost:5432",
    "2-2": "5432",
    "3-0": "📮 Redis",
    "3-1": "localhost:6379",
    "3-2": "6379",
    "4-0": "📡 Ultrafeeder",
    "4-1": "http://localhost:18080",
    "4-2": "18080"
  },
  "cols": 3,
  "rows": 5
}
[/block]

---

## 🌟 Community Guidelines

[block:callout]
{
  "type": "success",
  "title": "💚 Be Welcoming",
  "body": "We welcome contributors of all experience levels. Everyone was new once!"
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "🤝 Be Respectful",
  "body": "Treat everyone with respect. Constructive criticism is welcome; personal attacks are not."
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "📣 Communicate Clearly",
  "body": "When in doubt, over-communicate. Ask questions if something is unclear."
}
[/block]

---

## 🆘 Getting Help

[block:parameters]
{
  "data": {
    "h-0": "Resource",
    "h-1": "Description",
    "h-2": "Link",
    "0-0": "📚 Documentation",
    "0-1": "Check the docs/ directory",
    "0-2": "README",
    "1-0": "🐛 Issues",
    "1-1": "Search existing issues first",
    "1-2": "GitHub Issues",
    "2-0": "💬 Discussions",
    "2-1": "Ask questions",
    "2-2": "GitHub Discussions",
    "3-0": "📜 Code of Conduct",
    "3-1": "Be respectful and inclusive",
    "3-2": "CODE_OF_CONDUCT.md"
  },
  "cols": 3,
  "rows": 4
}
[/block]

---

[block:callout]
{
  "type": "success",
  "title": "🎉 Thank You!",
  "body": "Thank you for contributing to SkySpy! Your contributions help make aircraft tracking better for everyone."
}
[/block]
