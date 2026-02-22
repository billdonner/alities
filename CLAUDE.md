# Alities Hub — Central Entry Point

Alities is a trivia-based design and development environment that generates a variety of games.

## Ecosystem Layout

| Repo | Path | Purpose |
|------|------|---------|
| **alities** | `~/alities` | Specs, documentation, orchestration hub |
| **alities-engine** | `~/alities-engine` | Swift trivia content engine (gen-daemon + profile, source of truth) |
| **alities-studio** | `~/alities-studio` | React/TypeScript game designer & player web app |
| **alities-mobile** | `~/alities-mobile` | SwiftUI iOS game player app |
| **alities-trivwalk** | `~/alities-trivwalk` | Python TrivWalk trivia walking game |
| **adveyes** | `~/adveyes` | Python FastAPI multi-tenant advice backend (Claude) |

There is NO runnable code in this repo. All executable work happens in the satellite repos.

## Cross-Project Sync

The engine is CLI-first with no OpenAPI spec generation. Sync is done by comparing source code directly.

After any model or data format change in alities-engine:
1. Check studio's `src/` for model structs or data format assumptions that may need updating
2. Check mobile's model structs (e.g., `GameModels.swift`) for field drift against engine models in `Sources/AlitiesEngine/Models/`
3. Update alities-studio UI if affected
4. Update alities-mobile models if affected

## Test Suites

| Repo | Tests | Command |
|------|-------|---------|
| alities-engine | 82 | `cd ~/alities-engine && swift test` |
| alities-studio | 4 | `cd ~/alities-studio && npx vitest run` |
| alities-mobile | 36 | `cd ~/alities-mobile && swift test` |
| **Total** | **122** | |

## Custom Skills (Slash Commands)

Available when working from `~/alities`:

| Skill | What it does |
|-------|-------------|
| `/test-all` | Run all 3 test suites in parallel, report results table |
| `/sync-api` | Check API & model drift across all 3 repos |
| `/gap-check` | Compare specs in `Docs/` vs implementations across all repos |
| `/ship` | Test all → commit dirty repos → push everything |

Skills live in `~/alities/.claude/commands/`.

## Key Concepts

- **Game Template** — a reusable trivia game format (e.g., quiz show, flashcard drill, bracket elimination)
- **Topic Pack** — a curated set of trivia questions for a specific subject area
- **Game Instance** — a generated playable game combining a template + topic pack(s) + configuration
- **Studio** — the web-based designer where creators build and preview games
- **Engine** — the backend that stores templates, manages topic packs, generates game instances, and scores gameplay
