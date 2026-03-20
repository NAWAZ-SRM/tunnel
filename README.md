# Tunnel — Instant Localhost Sharing

**Status**: In Development — v1.0.0 MVP

Share your `localhost` with anyone via a public HTTPS URL. No client setup required. Auto-expires on Ctrl+C.

## Quick Start

### Server Setup (VPS)

```bash
# Clone and configure
git clone https://github.com/nawaz/tunnel.git /opt/tunnel
cd /opt/tunnel
cp .env.example .env
# Edit .env with your values

# Start services
docker compose up -d

# Initialize database and port pool
docker compose exec api python -m app.scripts.init_port_pool
docker compose exec api alembic upgrade head

# Verify
curl https://api.tunnel.dev/health
```

### CLI Installation

```bash
pip install tunnel-cli
# or
pipx install tunnel-cli

# Verify
tunnel version
```

### Create a Tunnel

```bash
tunnel share 3000
# Output:
# Public URL:    https://abc123xy.tunnel.dev
# Dashboard:     https://dash.tunnel.dev/t/abc123xy
# Waiting for visitors...
```

## Architecture

```
Developer Machine              Tunnel Server (VPS)
    │                                │
    │  1. tunnel share 3000          │
    │     → POST /tunnels            │
    │     ← {slug, port, url}        │
    │                                │
    │  2. ssh -R {port}:localhost:3000
    │                                │
    │  3. WebSocket live logs ◄──────┤
    │                                │
    │                          Viewer Browser
    │                                │
    │     Caddy (TLS) ───────────────┤
    │                                │
    │     FastAPI ── httpx ── SSH tunnel ── localhost:{port}
    │                                │
    │     PostgreSQL (logs)           │
    │     Redis (port pool)           │
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| CLI | Python + Typer |
| Tunnel Mechanism | OpenSSH `-R` |
| Backend | FastAPI |
| Proxy | httpx |
| Database | PostgreSQL |
| Session Store | Redis |
| Frontend | React + Vite + Tailwind |
| TLS/Reverse Proxy | Caddy |

## Project Structure

```
tunnel/
├── server/               # FastAPI backend
│   ├── app/
│   │   ├── api/         # REST endpoints
│   │   ├── models/      # SQLAlchemy models
│   │   ├── schemas/     # Pydantic schemas
│   │   ├── services/    # Business logic
│   │   └── scripts/      # DB/port pool init
│   ├── alembic/          # Migrations
│   └── Dockerfile
│
├── cli/                  # Python CLI
│   ├── localdrop/
│   │   ├── cli.py       # Typer commands
│   │   ├── ssh.py       # SSH manager
│   │   ├── api_client.py
│   │   ├── websocket.py
│   │   ├── display.py   # Rich terminal UI
│   │   └── auth.py      # OAuth flow
│   └── pyproject.toml
│
├── client/               # React dashboard
│   ├── src/
│   │   ├── pages/       # Home, Tunnel, Login
│   │   ├── components/   # Reusable UI
│   │   └── hooks/        # WebSocket, API
│   └── Dockerfile
│
├── infra/               # Server configs
│   ├── Caddyfile        # TLS + routing
│   ├── sshd_config.snippet
│   └── tunnel-ssh-handler
│
├── docker-compose.yml
└── Makefile
```

## Development

```bash
# Install dependencies
cd server && pip install -r requirements.txt
cd client && npm install

# Run server
cd server && uvicorn app.main:app --reload

# Run client
cd client && npm run dev

# Run tests
make test
```

## License

MIT
# tunnel
# tunnel
