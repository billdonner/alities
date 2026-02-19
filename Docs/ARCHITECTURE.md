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

The engine is a **CLI-first** tool. The daemon mode runs a background acquisition loop with a lightweight NIO HTTP control server on port 9847 (localhost only). Studio and mobile consume exported game data files — they do not call the engine's HTTP endpoints directly.

## Data Model

### Core Entities

| Entity | Description | Key Fields |
|--------|-------------|------------|
| **GameTemplate** | Reusable game format definition | id, name, format_type, rules_config, created_by |
| **TopicPack** | Collection of trivia questions | id, name, category, version, question_count, created_by |
| **Question** | Single trivia question | id, topic_pack_id, text, correct_answer, wrong_answers[], difficulty, tags[] |
| **GameInstance** | A generated playable game | id, template_id, topic_pack_ids[], seed, status, config |
| **GameSession** | A player's play-through of a game instance | id, game_instance_id, player_id, score, started_at, completed_at |
| **Player** | A user who plays games | id, display_name, email, stats |
| **Creator** | A user who designs templates/packs | id, display_name, email, published_count |

### Relationships

```
Creator ──creates──> GameTemplate
Creator ──creates──> TopicPack ──contains──> Question[]
GameTemplate + TopicPack[] ──generates──> GameInstance
Player ──plays──> GameSession ──of──> GameInstance
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

In daemon mode (`run`), an internal NIO HTTP control server listens on `127.0.0.1:9847` with operational endpoints (`/status`, `/harvest`, `/pause`, `/resume`, `/stop`, `/import`, `/categories`). These are for daemon control only, not for serving game content to clients.

Studio and mobile consume **exported JSON files** (GameData format), not live API calls. Cross-project sync is done by comparing model structs and data formats in source code — there is no OpenAPI spec generation.

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
