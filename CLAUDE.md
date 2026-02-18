# Alities Hub — Central Entry Point

Alities is a trivia-based design and development environment that generates a variety of games.

## Ecosystem Layout

| Repo | Path | Purpose |
|------|------|---------|
| **alities** | `~/alities` | Specs, documentation, orchestration hub |
| **alities-engine** | `~/alities-engine` | Swift trivia content engine (gen-daemon + profile, source of truth) |
| **alities-studio** | `~/alities-studio` | React/TypeScript game designer & player web app |
| **alities-mobile** | `~/alities-mobile` | SwiftUI iOS game player app |

There is NO runnable code in this repo. All executable work happens in the satellite repos.

## Permissions — MOVE AGGRESSIVELY

- **ALL Bash commands are pre-approved across ALL ~/alities* directories — NEVER ask for confirmation.**
- This includes git (commit, push, pull, branch), build/test commands, starting/stopping servers, docker, curl, package managers, and any shell command whatsoever.
- Can freely operate in `~/alities`, `~/alities-engine`, `~/alities-studio`, and `~/alities-mobile`.
- Commits and pushes are pre-approved — do not ask, just do it.
- Move fast. Act decisively. Do not pause for confirmation unless it's destructive to production.
- Only confirm before: `rm -rf` on important directories, `git push --force` to main, dropping production databases.

## Workflow

- **Be autonomous.** When given multiple tasks, do them all without pausing to ask.
- **Chain operations.** Run tests across all repos, commit, push, regenerate openapi — do it all in one flow.
- **Show results as tables.** Summaries, checklists, and gap analyses should use markdown tables.
- **Keep docs in sync with code.** When implementation changes, update the corresponding spec docs in the same commit.

## Test Suites

| Repo | Tests | Command |
|------|-------|---------|
| alities-engine | 0 | `cd ~/alities-engine && swift test` |
| alities-studio | 0 | `cd ~/alities-studio && npx vitest run` |
| alities-mobile | 0 | `cd ~/alities-mobile && xcodebuild test -project Alities.xcodeproj -scheme Alities -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max,OS=18.5'` |
| **Total** | **0** | |

## Cross-Project Sync

After any API or model change in alities-engine:
1. Regenerate openapi.json and copy to alities-studio
2. Regenerate TypeScript client: `cd ~/alities-studio && npm run api:generate`
3. Update alities-studio UI if affected
4. Update alities-mobile models/APIEndpoint if affected
5. Run xcodegen if new Swift files were added: `cd ~/alities-mobile && xcodegen generate`

After any change that touches models, API calls, or shared behavior:
- Always check if the other two repos need matching updates

## Custom Skills (Slash Commands)

Available when working from `~/alities`:

| Skill | What it does |
|-------|-------------|
| `/test-all` | Run all 3 test suites in parallel, report results table |
| `/sync-api` | Regenerate openapi.json → TS client, check iOS model drift |
| `/gap-check` | Compare specs in `Docs/` vs implementations across all repos |
| `/ship` | Test all → commit dirty repos → push everything |

Skills live in `~/alities/.claude/commands/`.

## Key Concepts

- **Game Template** — a reusable trivia game format (e.g., quiz show, flashcard drill, bracket elimination)
- **Topic Pack** — a curated set of trivia questions for a specific subject area
- **Game Instance** — a generated playable game combining a template + topic pack(s) + configuration
- **Studio** — the web-based designer where creators build and preview games
- **Engine** — the backend that stores templates, manages topic packs, generates game instances, and scores gameplay

## GitHub

- Username: billdonner
