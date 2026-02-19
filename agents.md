# Alities — Agent Reference

Context file for LLMs and AI agents working on the Alities ecosystem.

## What Is Alities?

Alities is a trivia-based game design and development platform. Creators design game templates and topic packs; the engine generates playable game instances; players interact via web or iOS.

## Repositories

| Repo | Path | GitHub | Stack | Purpose |
|------|------|--------|-------|---------|
| alities | `~/alities` | billdonner/alities | Markdown, orchestration | Hub — specs, docs, no runnable code |
| alities-engine | `~/alities-engine` | billdonner/alities-engine | Swift 5.9+, SPM, PostgreSQL, SQLite | Content engine — trivia acquisition, storage, export |
| alities-studio | `~/alities-studio` | billdonner/alities-studio | React 19, TypeScript, Vite | Web app — game designer and player |
| alities-mobile | `~/alities-mobile` | billdonner/alities-mobile | SwiftUI, iOS 17+, SPM | iOS app — game player |

## Domain Concepts

| Concept | Definition |
|---------|------------|
| Game Template | A reusable trivia game format (quiz show, flashcard drill, bracket elimination, etc.) |
| Topic Pack | A curated set of trivia questions for a specific subject area |
| Game Instance | A generated playable game = template + topic pack(s) + configuration |
| Studio | The web-based designer where creators build and preview games |
| Engine | The backend that stores templates, manages topic packs, generates game instances, and scores gameplay |

## Engine Architecture

The engine is the source of truth for all trivia content and APIs.

### CLI Subcommands

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

### Trivia Providers

| Provider | Source | API Key |
|----------|--------|---------|
| OpenTriviaDB | opentdb.com | No |
| TheTriviaAPI | the-trivia-api.com | No |
| jService | the-trivia-api.com (category-focused) | No |
| AI Generator | OpenAI GPT-4o-mini | Yes (`OPENAI_API_KEY`) |
| File Import | Local JSON/CSV | No |

### Dual Database Architecture

- **PostgreSQL** — daemon mode, high-volume online storage
- **SQLite (GRDB)** — profile commands, local data management
- Both read/write the same JSON formats (GameData and Raw)

### Engine Source Layout

```
Sources/AlitiesEngine/
├── AlitiesEngineCLI.swift       # @main entry point
├── Models/
│   ├── TriviaQuestion.swift     # Core question model
│   ├── GameData.swift           # GameDataOutput + Challenge + TopicPicMapping
│   ├── ProfileModels.swift      # RawQuestion, ProfiledQuestion, DataLoader
│   └── Report.swift             # Report generation and rendering
├── Providers/
│   ├── TriviaProvider.swift     # Protocol + errors
│   ├── OpenTriviaDBProvider.swift
│   ├── TheTriviaAPIProvider.swift
│   ├── JServiceProvider.swift
│   ├── AIGeneratorProvider.swift
│   └── FileImportProvider.swift
├── Services/
│   ├── TriviaGenDaemon.swift    # Main daemon actor
│   ├── PostgresService.swift    # PostgreSQL operations
│   ├── GameDataTransformer.swift
│   ├── ControlServer.swift      # NIO HTTP control server
│   └── SimilarityService.swift
├── Profile/
│   ├── TriviaDatabase.swift     # SQLite/GRDB operations
│   └── CategoryMap.swift        # Category normalization + SF Symbols
└── Commands/
    ├── RunCommand.swift
    ├── GenImportCommand.swift
    ├── ProfileImportCommand.swift
    ├── ExportCommand.swift
    ├── ReportCommand.swift
    ├── StatsCommand.swift
    ├── CategoriesCommand.swift
    ├── HarvestCommand.swift
    ├── CtlCommand.swift
    └── MigrateCommand.swift
```

## Server Ports

| Service | Port | Protocol |
|---------|------|----------|
| alities-engine daemon control | 9847 | HTTP (localhost only) |
| alities-studio dev server | 9850 | HTTP (Vite) |
| PostgreSQL | 5432 | PostgreSQL |

Port file written to `/tmp/alities-engine.port` for CLI auto-discovery.

## Environment Variables

| Variable | Default | Required By |
|----------|---------|-------------|
| `OPENAI_API_KEY` | — | Engine (AI provider) |
| `DB_HOST` | localhost | Engine (PostgreSQL) |
| `DB_PORT` | 5432 | Engine (PostgreSQL) |
| `DB_USER` | trivia | Engine (PostgreSQL) |
| `DB_PASSWORD` | trivia | Engine (PostgreSQL) |
| `DB_NAME` | trivia_db | Engine (PostgreSQL) |

## Build & Test Commands

| Repo | Build | Test |
|------|-------|------|
| alities-engine | `swift build -c release` | `swift test` |
| alities-studio | `npm run build` | `npx vitest run` |
| alities-mobile | `swift build` | `swift test` |

Install engine globally: `cp .build/release/AlitiesEngine ~/bin/alities-engine`

## Cross-Project Sync Rules

The engine is the API source of truth. After any API or model change in alities-engine:

1. Regenerate `openapi.json` and copy to alities-studio
2. Regenerate TypeScript client: `cd ~/alities-studio && npm run api:generate`
3. Update alities-studio UI if affected
4. Update alities-mobile models if affected

After any change touching models, API calls, or shared behavior — check all three repos.

## Provenance

The engine was merged from two predecessor repos:
- [trivia-gen-daemon](https://github.com/billdonner/trivia-gen-daemon) — 5 trivia providers, daemon loop, PostgreSQL
- [trivia-profile](https://github.com/billdonner/trivia-profile) — SQLite import/export/report, category normalization
