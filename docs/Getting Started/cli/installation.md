---
title: "Installation"
excerpt: "Installing the SkySpy CLI from source or binary"
---

# Installation

The SkySpy CLI can be installed from source or downloaded as a pre-built binary.

<Tabs>
  <Tab title="Pre-built Binary (Recommended)">

Pre-built binaries are available on the releases page. Download the appropriate binary for your platform:

| Platform | Architecture | Binary |
|----------|--------------|--------|
| Linux | x86_64 | `skyspy-linux-amd64` |
| Linux | ARM64 | `skyspy-linux-arm64` |
| macOS | Intel | `skyspy-darwin-amd64` |
| macOS | Apple Silicon | `skyspy-darwin-arm64` |
| Windows | x86_64 | `skyspy-windows-amd64.exe` |

**Install on Linux/macOS:**

```bash
# Download and make executable
chmod +x skyspy-linux-amd64
sudo mv skyspy-linux-amd64 /usr/local/bin/skyspy

# Verify installation
skyspy --help
```

**Install on Windows:**

1. Download `skyspy-windows-amd64.exe`
2. Rename to `skyspy.exe`
3. Add to your PATH or run directly

  </Tab>
  <Tab title="Build from Source">

**Prerequisites:**
- Go 1.21 or later
- Git

**Clone and Build:**

```bash
# Clone the repository
git clone https://github.com/skyspy/skyspy.git
cd skyspy/skyspy-go

# Build the binary
go build -o skyspy ./cmd/skyspy

# Install to your PATH (optional)
go install ./cmd/skyspy
```

**Build with Version Information:**

```bash
go build -ldflags "-X main.version=1.0.0 -X main.commit=$(git rev-parse --short HEAD)" -o skyspy ./cmd/skyspy
```

  </Tab>
</Tabs>

## Cross-Platform Builds

The CLI can be compiled for multiple platforms from a single machine.

```bash Linux (AMD64)
GOOS=linux GOARCH=amd64 go build -o skyspy-linux-amd64 ./cmd/skyspy
```
```bash Linux (ARM64)
GOOS=linux GOARCH=arm64 go build -o skyspy-linux-arm64 ./cmd/skyspy
```
```bash macOS (Intel)
GOOS=darwin GOARCH=amd64 go build -o skyspy-darwin-amd64 ./cmd/skyspy
```
```bash macOS (Apple Silicon)
GOOS=darwin GOARCH=arm64 go build -o skyspy-darwin-arm64 ./cmd/skyspy
```
```bash Windows
GOOS=windows GOARCH=amd64 go build -o skyspy-windows-amd64.exe ./cmd/skyspy
```

## Using Make

If a Makefile is provided, you can use make commands:

```bash
# Build for current platform
make build

# Build for all platforms
make build-all

# Run tests
make test

# Install locally
make install
```

## Verifying Installation

After installation, verify the CLI is working:

```bash
# Check version and help
skyspy --help

# List available themes
skyspy --list-themes

# Test connection to a server
skyspy --host your-server.com --port 80
```

## Updating

To update to the latest version:

```bash
# If installed via go install
go install github.com/skyspy/skyspy-go/cmd/skyspy@latest

# If built from source
cd skyspy/skyspy-go
git pull
go build -o skyspy ./cmd/skyspy
```

## Troubleshooting

### Build Errors

If you encounter build errors, ensure you have the correct Go version:

```bash
go version  # Should be 1.21 or later
```

### Permission Denied

On Linux/macOS, ensure the binary is executable:

```bash
chmod +x skyspy
```

### Windows Terminal Issues

For the best experience on Windows, use Windows Terminal or another terminal that supports 256 colors and Unicode characters.
