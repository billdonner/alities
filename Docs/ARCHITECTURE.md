# Alities — Architecture

## System Overview

```
┌─────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  alities-studio │     │ alities-mobile   │     │  (future CLIs)   │
│  React/TS       │     │ SwiftUI/iOS      │     │                  │
└────────┬────────┘     └────────┬─────────┘     └────────┬─────────┘
         │                       │                         │
         └───────────┬───────────┴─────────────────────────┘
                     │ HTTPS / REST
              ┌──────┴──────┐
              │  alities-   │
              │  engine     │
              │  FastAPI    │
              ├─────────────┤
              │  Game Gen   │
              │  Scoring    │
              │  Content    │
              └──────┬──────┘
                     │
         ┌───────────┼───────────┐
         │           │           │
    ┌────┴────┐ ┌────┴────┐ ┌───┴────┐
    │ Postgres│ │  Redis  │ │ S3/Obj │
    │ (data)  │ │ (cache, │ │ (media)│
    │         │ │  scores)│ │        │
    └─────────┘ └─────────┘ └────────┘
```

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

| Service | Responsibility |
|---------|---------------|
| **Content Service** | CRUD for templates, topic packs, questions; import/export |
| **Generation Service** | Combines template + packs into game instances; question selection algorithm |
| **Play Service** | Game session management, real-time scoring, hint dispensing |
| **Analytics Service** | Player stats, leaderboards, question difficulty calibration |
| **Auth Service** | Registration, login, JWT tokens, roles |

## API Contract

- OpenAPI spec generated from alities-engine (source of truth)
- TypeScript client auto-generated via orval for alities-studio
- iOS models manually synced; drift checked by `/sync-api` skill

## Tech Stack

| Layer | Technology |
|-------|-----------|
| API Server | Python 3.12, FastAPI, SQLAlchemy (async), PostgreSQL |
| Cache/Realtime | Redis |
| Web Client | React, TypeScript, Vite |
| Mobile Client | SwiftUI, Swift 6, iOS 17+ |
| Project Gen (iOS) | xcodegen |
| Package Mgmt | uv (Python), npm (JS), SPM (Swift) |
