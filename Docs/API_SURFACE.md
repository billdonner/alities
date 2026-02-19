# Alities — API Surface

## Overview

The engine is a **CLI-first** tool. The primary interface is CLI subcommands (see below). In daemon mode, a lightweight HTTP control server provides operational endpoints for monitoring, controlling the running daemon, and serving game data to clients.

## CLI Subcommands (Primary Interface)

| Command | Description |
|---------|-------------|
| `run` | Start acquisition daemon (dual-write: file + PostgreSQL) |
| `list-providers` | Show available trivia providers |
| `status` | Show daemon status |
| `gen-import` | Import file directly to PostgreSQL |
| `import` | Import JSON files to SQLite (with dedup) |
| `export` | Export from SQLite as raw or gamedata JSON |
| `report` | Profile trivia data (from files or SQLite) |
| `stats` | Quick database summary (default command) |
| `categories` | List categories with counts and aliases |
| `harvest` | Request targeted AI generation from running daemon |
| `ctl` | Control running daemon (status/pause/resume/stop/import/categories) |
| `migrate` | Migrate SQLite questions to PostgreSQL |

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

## Client Usage

Both alities-studio and alities-mobile call the daemon's HTTP endpoints directly:

| Client | Endpoints Used |
|--------|---------------|
| alities-studio | `GET /status`, `GET /categories` |
| alities-mobile | `GET /status`, `GET /categories`, `GET /gamedata` |

CORS headers are included on all responses for studio web app compatibility.
