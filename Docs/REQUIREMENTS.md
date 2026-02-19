# Alities — Requirements

> **Status: PARTIAL** — The engine's trivia acquisition, storage, and export pipeline is implemented. Game templates, game generation, scoring, leaderboards, multiplayer, and content moderation are planned but not yet built.

## Vision

Alities is a trivia-based design and development environment that generates a variety of games. Creators design game templates and curate topic packs; the engine combines them into playable game instances.

## User Roles

| Role | Description |
|------|-------------|
| **Creator** | Designs game templates, authors/imports topic packs, configures game parameters |
| **Player** | Plays generated game instances, earns scores, competes on leaderboards |
| **Admin** | Manages platform content, reviews submissions, monitors quality |

## Core Features

### Game Templates
- Define reusable game formats (quiz show, flashcard drill, bracket elimination, speed round, etc.)
- Configure rules: time limits, scoring multipliers, hint availability, difficulty progression
- Preview templates in the Studio before publishing

### Topic Packs
- Author trivia questions with metadata: category, difficulty, tags, source
- Import questions from structured formats (CSV, JSON)
- Quality scoring: flag duplicates, validate answers, rate difficulty
- Versioned — updates don't break existing game instances

### Game Generation
- Combine one template + one or more topic packs → playable game instance
- Engine selects questions based on difficulty curve, category balance, and player history
- Deterministic seeding for reproducible games (useful for competitions)

### Gameplay
- Real-time scoring with streak bonuses
- Hints system (costs points, reveals partial info)
- Multiplayer support: head-to-head and group modes
- Session persistence — resume interrupted games

### Analytics & Leaderboards
- Per-player statistics: accuracy, speed, streaks, favorite categories
- Per-topic-pack analytics: question difficulty calibration
- Global and per-game leaderboards

## Non-Functional Requirements

- API response time < 200ms (p95)
- Support 1000 concurrent players
- All data exportable (GDPR/CCPA)
- Content moderation pipeline for user-submitted questions
- Data retention and deletion controls for player data (GDPR right to erasure)
