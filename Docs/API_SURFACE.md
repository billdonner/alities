# Alities — API Surface

## Overview

The engine is a **CLI-first** tool. The primary interface is CLI subcommands (see below). In daemon mode, a lightweight HTTP control server provides operational endpoints for monitoring and controlling the running daemon.

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
| `ctl` | Control running daemon (status/pause/resume/stop) |
| `migrate` | Migrate SQLite questions to PostgreSQL |

## Daemon Control Endpoints

Base URL: `http://127.0.0.1:9847` (localhost only, daemon mode)

Port file written to `/tmp/alities-engine.port` for CLI auto-discovery.

| Method | Path | Description |
|--------|------|-------------|
| GET | `/status` | Daemon stats (uptime, question count, provider status) |
| GET | `/categories` | List categories with counts |
| POST | `/harvest` | Trigger targeted AI generation |
| POST | `/pause` | Pause daemon acquisition loop |
| POST | `/resume` | Resume daemon acquisition loop |
| POST | `/stop` | Gracefully stop daemon |
| POST | `/import` | Import JSON questions into daemon's database |

These endpoints are for **daemon control only** — they are not a content-serving REST API. Studio and mobile consume exported JSON files (GameData format), not live HTTP calls.

## Authentication

None. The control server binds to localhost only and is not exposed externally.
