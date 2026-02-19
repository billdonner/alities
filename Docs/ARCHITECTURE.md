# Alities — Architecture

## System Overview

```
┌─────────────────┐     ┌──────────────────┐
│  alities-studio │     │ alities-mobile   │
│  React/TS       │     │ SwiftUI/iOS      │
└────────┬────────┘     └────────┬─────────┘
         │                       │
         └───────────┬───────────┘
                     │ (offline / local data)
              ┌──────┴──────┐
              │  alities-   │
              │  engine     │
              │  Swift CLI  │
              ├─────────────┤
              │  Daemon     │
              │  Providers  │
              │  Profile    │
              └──────┬──────┘
                     │
              ┌──────┼──────┐
              │             │
         ┌────┴────┐  ┌────┴────┐
         │ Postgres│  │ SQLite  │
         │ (daemon │  │ (profile│
         │  mode)  │  │  mode)  │
         └─────────┘  └─────────┘
```

The engine is a **CLI-first** tool. The daemon mode runs a background acquisition loop with a lightweight NIO HTTP control server on port 9847 (localhost only, CORS enabled). Studio and mobile call the daemon's HTTP endpoints directly (`/status`, `/categories`, `/gamedata`) and can also load exported JSON files offline.

## Data Model

### Implemented Entities

| Entity | Description | Key Fields |
|--------|-------------|------------|
| **TriviaQuestion** | Core question model (engine) | text, choices[], correctChoiceIndex, category, difficulty (easy/medium/hard), explanation?, hint?, source |
| **Challenge** | Exported question for players (engine/studio/mobile) | id, topic, pic (SF Symbol), question, answers[], correct, explanation, hint, aisource, date |
| **GameDataOutput** | Exported game data bundle (engine/studio/mobile) | id, generated (timestamp), challenges[] |
| **Category** | Category with SF Symbol (engine) | id, name, pic, count |

### Planned Entities (not yet implemented)

> See feature specs in `Docs/` for the target design of these entities.

| Entity | Description |
|--------|-------------|
| **GameTemplate** | Reusable game format definition (quiz show, flashcard, bracket, etc.) |
| **TopicPack** | Versioned collection of trivia questions |
| **GameInstance** | A generated playable game combining template + topic packs |
| **GameSession** | A player's play-through of a game instance |
| **Player** | A user who plays games |
| **Creator** | A user who designs templates/packs |

### Current Data Flow

```
Providers ──fetch──> TriviaQuestion[] ──store──> PostgreSQL / SQLite
SQLite ──export──> GameDataOutput (Challenge[])
Daemon /gamedata ──serve──> Mobile app
Daemon /categories ──serve──> Studio + Mobile
```

## Service Boundaries

| Module | Location | Responsibility |
|--------|----------|---------------|
| **Commands/** | `Sources/AlitiesEngine/Commands/` | CLI subcommands (run, import, export, report, stats, etc.) |
| **Services/** | `Sources/AlitiesEngine/Services/` | Daemon, PostgreSQL ops, control server, game data transformer |
| **Providers/** | `Sources/AlitiesEngine/Providers/` | Trivia acquisition from 5 sources (OpenTriviaDB, TheTriviaAPI, jService, AI Generator, File Import) |
| **Profile/** | `Sources/AlitiesEngine/Profile/` | SQLite/GRDB operations, category normalization |
| **Models/** | `Sources/AlitiesEngine/Models/` | TriviaQuestion, GameData, ProfileModels, Report |

## API Contract

The engine is **CLI-first**. The primary interface is CLI subcommands (`run`, `import`, `export`, `report`, `stats`, `categories`, etc.).

In daemon mode (`run`), an internal NIO HTTP control server listens on `127.0.0.1:9847` with:
- **Public GET endpoints**: `/health`, `/status`, `/categories`, `/gamedata` — used by studio and mobile clients
- **Protected POST endpoints**: `/harvest`, `/pause`, `/resume`, `/stop`, `/import` — for daemon control (Bearer token auth via `CONTROL_API_KEY`)

Cross-project sync is done by comparing model structs and data formats in source code — there is no OpenAPI spec generation. See [API_SURFACE.md](API_SURFACE.md) for the full endpoint inventory.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Engine CLI/Daemon | Swift 5.9+, SPM, ArgumentParser |
| Daemon DB | PostgreSQL (PostgresNIO) |
| Profile DB | SQLite (GRDB) |
| HTTP Control | SwiftNIO |
| Web Client | React 19, TypeScript, Vite |
| Mobile Client | SwiftUI, iOS 17+, SPM |
| Package Mgmt | SPM (Swift), npm (JS) |
