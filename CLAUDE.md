# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive Google Workspace MCP (Model Context Protocol) server that provides AI assistants with complete access to Google Workspace services including Gmail, Calendar, Drive, Docs, Sheets, Slides, Forms, Tasks, and Chat. It's built with FastMCP and features advanced OAuth 2.0 authentication, service caching, and both stdio and HTTP transport modes.

## Development Commands

### Basic Operations
```bash
# Install dependencies and run server
uv run main.py

# Run with specific tools only
uv run main.py --tools gmail drive calendar

# Run in HTTP mode for debugging
uv run main.py --transport streamable-http

# Run in single-user mode (simplified auth)
uv run main.py --single-user

# Install via uvx (production)
uvx workspace-mcp

# Install dependencies for development
uv sync
```

### Testing and Quality
```bash
# The project doesn't include specific test runners in pyproject.toml
# Check README.md for any testing instructions if available
python -m pytest  # if tests exist

# Check service health (when running in HTTP mode)
curl http://localhost:8000/health
```

### Installation Helpers
```bash
# Auto-configure Claude Desktop
python install_claude.py

# Docker build and run
docker build -t workspace-mcp .
docker run -p 8000:8000 workspace-mcp --transport streamable-http
```

## Architecture

### Core Structure
- **`main.py`**: Entry point with argument parsing, tool loading, and server startup
- **`core/server.py`**: FastMCP server instance with OAuth callback handling and tool registration
- **`auth/`**: Complete authentication system with decorators, OAuth flows, and credential management
- **`g{service}/`**: Service-specific tool implementations (gmail, gdrive, gcalendar, etc.)

### Key Architectural Patterns

#### Service Decorator System
The `@require_google_service()` decorator in `auth/service_decorator.py` handles:
- Automatic service authentication and injection
- 30-minute service caching with TTL
- Scope resolution from friendly names to Google OAuth URLs
- Token refresh error handling with user-friendly messages

```python
@require_google_service("gmail", "gmail_read")
async def search_messages(service, user_google_email: str, query: str):
    # service is automatically injected and authenticated
    return service.users().messages().list(userId='me', q=query).execute()
```

#### Scope Management
- Centralized scope definitions in `auth/scopes.py`
- Friendly scope group names (e.g., `"gmail_read"` → actual Google scope URL)
- Service-specific scope groupings for easy maintenance

#### Transport Flexibility
- **Stdio mode**: Default MCP transport for Claude Desktop integration
- **HTTP mode**: FastAPI-based server for web interfaces and debugging
- **OAuth callback handling**: Automatic HTTP server for OAuth redirects in both modes

#### Authentication Flow
1. Tools check for cached valid credentials
2. If none found, returns authentication URL via `start_google_auth`
3. User completes OAuth flow in browser
4. Server handles callback and stores credentials
5. Original tool request is retried with fresh credentials

### Environment Configuration
- OAuth credentials via environment variables (recommended) or `client_secret.json`
- Service-specific environment variables for customization
- Development mode support with `OAUTHLIB_INSECURE_TRANSPORT`

### Service Caching
- 30-minute TTL for authenticated service instances
- Cache key includes user email, service type, version, and scopes
- Automatic cache invalidation on token refresh errors
- Thread-safe implementation for concurrent requests

## Important Notes

### OAuth Setup Requirements
1. Google Cloud Project with enabled APIs (Calendar, Drive, Gmail, Docs, Sheets, etc.)
2. OAuth 2.0 Client ID configured with redirect URI: `http://localhost:8000/oauth2callback`
3. Credentials provided via environment variables or `client_secret.json`

### Tool Development Pattern
New tools should follow the established pattern:
1. Use `@require_google_service()` decorator
2. First parameter must be `service` (auto-injected)
3. Include `user_google_email: str` parameter
4. Return native Python objects (automatically serialized)
5. Place in appropriate `g{service}/` directory

### Error Handling
- Token refresh errors trigger reauthentication flow
- Service caching automatically cleared on auth failures
- User-friendly error messages with specific remediation steps

### Multi-User Support
- Session-based credential isolation
- MCP session ID linking for multi-user environments
- Single-user mode available for simplified setups