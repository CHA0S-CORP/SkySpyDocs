---
title: "Authentication"
excerpt: "OIDC and API key authentication for secure server connections"
---

# Authentication

The SkySpy CLI supports multiple authentication methods for connecting to secured SkySpy servers.

## Authentication Methods

| Method | Use Case |
|--------|----------|
| **OIDC** | Interactive login via web browser (Google, Auth0, etc.) |
| **API Key** | Automated/scripted access without browser |
| **Public** | No authentication required (server in public mode) |

## OIDC Authentication

OIDC (OpenID Connect) authentication uses your identity provider (Google, Auth0, Okta, etc.) for secure login.

### Login Flow

1. Run the login command
2. Browser opens to your identity provider
3. Complete authentication in browser
4. CLI receives and stores tokens automatically

### Login Command

```bash
# Login to default server
skyspy login

# Login to specific server
skyspy login --host myserver.com --port 443
```

### What Happens

1. CLI contacts the server to get OIDC configuration
2. A local callback server starts on a random port
3. Browser opens to the authorization URL
4. After you authenticate, the browser redirects to the local callback
5. CLI exchanges the authorization code for tokens
6. Tokens are encrypted and stored locally

### Example Session

```
$ skyspy login --host skyspy.example.com --port 443
Connecting to skyspy.example.com:443...
Starting authentication with Google...
Opening browser for authentication...
Waiting for authentication (timeout: 5 minutes)...
Successfully authenticated as user@example.com
```

## API Key Authentication

For automated scripts or environments without a browser, use API key authentication.

### Using API Key

```bash
# Via command line flag
skyspy --api-key sk_live_xxxxxxxxxxxxx

# Via environment variable
export SKYSPY_API_KEY=sk_live_xxxxxxxxxxxxx
skyspy
```

### API Key Format

API keys typically follow the format:
- `sk_live_xxxxx` - Production keys
- `sk_test_xxxxx` - Test/development keys

### Environment Variable

The `SKYSPY_API_KEY` environment variable is checked automatically if no `--api-key` flag is provided:

```bash
# In your shell profile (.bashrc, .zshrc, etc.)
export SKYSPY_API_KEY=sk_live_xxxxxxxxxxxxx

# Then just run without flags
skyspy --host myserver.com
```

## Token Storage

### Storage Location

OIDC tokens are stored in:
```
~/.config/skyspy/credentials/
```

Each server has its own token file, named by host and port:
```
~/.config/skyspy/credentials/myserver.com_443.json
```

### Security

- Tokens are encrypted using AES-256-GCM
- Encryption key is derived from machine-specific data
- Files are created with restrictive permissions (0600)
- Tokens include refresh tokens for automatic renewal

### Token Lifecycle

1. **Initial login**: Access token + refresh token stored
2. **Token expiry**: Automatically refreshed using refresh token
3. **Refresh expiry**: User prompted to login again
4. **Logout**: Tokens securely deleted

## Checking Authentication Status

View your current authentication state:

```bash
skyspy auth status
```

### Sample Output (Authenticated)

```
Server: skyspy.example.com:443

Server Configuration:
  Auth Mode: oidc
  Auth Required: true
  OIDC: enabled (via Google)

Authentication Status:
  Status: Authenticated
  User: user@example.com
  Token Expires: 2024-01-15T10:30:00Z
```

### Sample Output (API Key)

```
Server: skyspy.example.com:443

Server Configuration:
  Auth Mode: api_key
  Auth Required: true
  OIDC: disabled

Authentication Status:
  Status: Authenticated (API Key)
  Key: sk_live_xxx...
```

### Sample Output (Not Authenticated)

```
Server: skyspy.example.com:443

Server Configuration:
  Auth Mode: oidc
  Auth Required: true
  OIDC: enabled (via Google)

Authentication Status:
  Status: Not authenticated

  Run 'skyspy login' to authenticate
```

## Logging Out

Clear stored credentials:

```bash
# Logout from default server
skyspy logout

# Logout from specific server
skyspy logout --host myserver.com --port 443
```

This deletes the encrypted token file for that server.

## Server Authentication Modes

Servers can be configured in different authentication modes:

| Mode | Description |
|------|-------------|
| `public` | No authentication required |
| `oidc` | OIDC authentication required |
| `api_key` | API key authentication required |
| `mixed` | OIDC or API key accepted |

The CLI automatically detects the server's authentication requirements.

## Troubleshooting

### Browser Doesn't Open

If the browser doesn't open automatically, copy the URL from the terminal:

```
Could not open browser automatically.
Please open this URL in your browser:

https://accounts.google.com/o/oauth2/v2/auth?...
```

### Authentication Timeout

The login process has a 5-minute timeout. If you don't complete authentication in time:

```bash
Authentication cancelled
```

Simply run `skyspy login` again.

### Token Refresh Failed

If token refresh fails, you'll see:

```
Token expired and refresh failed
Run 'skyspy login' to re-authenticate
```

### Permission Errors

If you see permission errors for the credentials directory:

```bash
# Fix permissions
chmod 700 ~/.config/skyspy/credentials
chmod 600 ~/.config/skyspy/credentials/*.json
```

### Server Connection Failed

If the server is unreachable:

```
Cannot connect to server
Error: connection refused
```

Check:
- Server hostname and port are correct
- Server is running and accessible
- No firewall blocking the connection

## Security Best Practices

1. **Protect API keys**: Never commit API keys to version control
2. **Use environment variables**: Store API keys in environment variables, not scripts
3. **Rotate keys**: Periodically rotate API keys
4. **Minimal permissions**: Request only necessary scopes during OIDC login
5. **Logout when done**: Run `skyspy logout` on shared systems
6. **Secure credentials directory**: Ensure `~/.config/skyspy/credentials` has proper permissions
