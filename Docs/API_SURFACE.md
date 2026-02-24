# Alities — API Surface

## Overview

The engine is a **CLI-first** tool. The primary interface is CLI subcommands (see below). In daemon mode, a lightweight HTTP control server provides operational endpoints for monitoring, controlling the running daemon, and serving game data to clients.

## CLI Subcommands (Primary Interface)

| Command | Description |
|---------|-------------|
| `run` | Start acquisition daemon (dual-write: file + PostgreSQL) |
| `list-providers` | Show available trivia providers |
| `stats` | Quick PostgreSQL database summary (default command) |
| `categories` | List categories with counts from PostgreSQL |
| `report` | Profile trivia data from JSON files (text or JSON output) |
| `gen-import` | Import JSON file directly to PostgreSQL (with dedup) |
| `harvest` | Request targeted AI generation from running daemon |
| `ctl` | Control running daemon (status/pause/resume/stop/import/categories) |
| `import` | Import JSON files to SQLite (planned — requires GRDB) |
| `export` | Export from SQLite as raw or gamedata JSON (planned — requires GRDB) |
| `migrate` | Migrate SQLite questions to PostgreSQL (planned — requires GRDB) |

## Daemon Control Endpoints

Base URL: `http://127.0.0.1:9847` (localhost only, daemon mode)

Port file written to `/tmp/alities-engine.port` for CLI auto-discovery.

### Public Endpoints (no auth)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check (returns `{"ok": true}`) |
| GET | `/status` | Daemon stats (uptime, question count, provider status) |
| GET | `/categories` | List categories with counts and SF Symbol names |
| GET | `/gamedata` | Full GameDataOutput JSON (all challenges for players) |
| GET | `/metrics` | Structured monitoring data for server-monitor (daemon state, memory, per-provider stats, Postgres counts) |

### Protected Endpoints (Bearer token)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/harvest` | Trigger targeted AI generation for specific categories |
| POST | `/pause` | Pause daemon acquisition loop |
| POST | `/resume` | Resume daemon acquisition loop |
| POST | `/stop` | Gracefully stop daemon |
| POST | `/import` | Import JSON questions into daemon's SQLite database |

## Authentication

- POST endpoints require `Authorization: Bearer <token>` header
- Token is set via the `CONTROL_API_KEY` environment variable
- If `CONTROL_API_KEY` is not set, no auth is required (development mode)
- GET endpoints are always public (no auth required)
- Returns `401 Unauthorized` if token is missing or invalid

## Static File Serving

When launched with `--static-dir <path>`, the daemon serves static files from the specified directory. This is used to host the alities-studio web build directly from the engine.

- SPA fallback: requests without file extensions serve `index.html`
- Directory traversal (`..`) is rejected
- Common MIME types: HTML, JS, CSS, SVG, JSON, images, fonts
- Static routes are lowest priority (API endpoints take precedence)

## Client Usage

Both alities-studio and alities-mobile call the daemon's HTTP endpoints directly:

| Client | Endpoints Used |
|--------|---------------|
| alities-studio | `GET /status`, `GET /categories`, `GET /gamedata` |
| alities-mobile | `GET /status`, `GET /categories`, `GET /gamedata` |
| server-monitor | `GET /metrics` |

CORS headers are included on all responses for studio web app compatibility.
