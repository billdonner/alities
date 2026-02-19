# Alities Comprehensive Review (2026-02-19)

## Scope

- Reviewed repo: `~/alities` (docs + automation command specs)
- Goal: consistency defects and bad/fragile command patterns
- Validation: checked key referenced paths in `~/alities-engine`, `~/alities-studio`, and `~/alities-mobile`

## Findings (Ordered by Severity)

### Critical

1. Automation commands target a Python engine layout that does not exist.
- Evidence:
  - `.claude/commands/test-all.md:6` runs `uv run pytest` in engine
  - `.claude/commands/ship.md:12` runs `uv run pytest` in engine
  - `.claude/commands/sync-api.md:7` imports `alities.server.main` via Python
  - `.claude/commands/gap-check.md:8` says engine code is in `~/alities-engine/src/`
  - `agents.md:14` and `agents.md:129` describe the engine as Swift + `swift test`
- Validation:
  - `missing /Users/billdonner/alities-engine/src`
  - `missing /Users/billdonner/alities-engine/pyproject.toml`
  - `exists /Users/billdonner/alities-engine/Package.swift`
  - `exists /Users/billdonner/alities-engine/Sources/AlitiesEngine`
- Impact: `/test-all`, `/ship`, `/sync-api`, `/gap-check` are misaligned with actual repo structure/toolchain.

2. Mobile test command is broken as written.
- Evidence:
  - `.claude/commands/test-all.md:8`
  - `.claude/commands/ship.md:14`
- Validation:
  - `missing /Users/billdonner/alities-mobile/Alities.xcodeproj`
  - Runtime check: `xcodebuild ... -project Alities.xcodeproj ...` returns `xcodebuild: error: 'Alities.xcodeproj' does not exist.`
- Impact: automated cross-repo test workflow fails deterministically.

3. `/sync-api` workflow references artifacts and paths that do not exist.
- Evidence:
  - `.claude/commands/sync-api.md:7` through `.claude/commands/sync-api.md:12`
  - `.claude/commands/sync-api.md:17`
  - `.claude/commands/sync-api.md:26` and `.claude/commands/sync-api.md:27`
- Validation:
  - `missing /Users/billdonner/alities-engine/docs`
  - `missing /Users/billdonner/alities-engine/docs/openapi.json`
  - `missing /Users/billdonner/alities-studio/openapi.json`
  - `missing /Users/billdonner/alities-mobile/Alities/Services/APIEndpoint.swift`
  - `missing /Users/billdonner/alities-mobile/Alities/Models`
- Impact: sync instructions cannot complete and will generate false expectations.

### High

1. Core architecture is inconsistent across top-level docs.
- Evidence:
  - `README.md:14` says engine stack is `Python / FastAPI`
  - `Docs/ARCHITECTURE.md:16` and `Docs/ARCHITECTURE.md:75` describe FastAPI/Python API server
  - `agents.md:14` and `agents.md:32` describe a Swift CLI/daemon engine
- Impact: contributors can choose the wrong runtime and wrong implementation target.

2. API contract docs do not match the current engine operating model.
- Evidence:
  - `Docs/API_SURFACE.md:5` sets base URL `http://127.0.0.1:8001/api/v1`
  - `Docs/API_SURFACE.md:24` through `Docs/API_SURFACE.md:84` define full auth/content/gameplay/admin REST APIs
  - `agents.md:36` through `agents.md:47` define CLI-first engine commands
  - `agents.md:108` documents daemon control port `9847`
- Impact: client generation and integration work risks targeting non-current endpoints.

3. Internal permission guidance conflicts with actual execution model and safe practice.
- Evidence:
  - `CLAUDE.md:18` through `CLAUDE.md:22` says all commands are pre-approved and to never ask
  - `.claude/settings.local.json:4` through `.claude/settings.local.json:31` includes very broad wildcard command allowlists
- Impact: unsafe automation assumptions and portability issues across agent runtimes.

### Medium

1. Scoring formula contradicts streak behavior text.
- Evidence:
  - `Docs/SCORING.md:19` streak multiplier formula is `min(streak × template.streak_multiplier, 3.0)`
  - `Docs/SCORING.md:27` says streak multiplier starts on the 2nd correct answer
- Impact: off-by-one scoring drift across platforms.

2. Generation input schema omits player context required by algorithm step.
- Evidence:
  - `Docs/GAME_GENERATION.md:10` through `Docs/GAME_GENERATION.md:17` input object has no player/session identifier
  - `Docs/GAME_GENERATION.md:23` filters out recently seen questions
- Impact: implementations cannot apply filter behavior consistently.

3. Determinism claim conflicts with history-based filtering.
- Evidence:
  - `Docs/GAME_GENERATION.md:23` uses mutable “recently seen” history
  - `Docs/GAME_GENERATION.md:31` says same inputs always produce same game
- Impact: reproducibility guarantee is ambiguous unless history snapshot is part of input.

4. `adaptive` progression conflicts with precomputed ordering model.
- Evidence:
  - `Docs/GAME_TEMPLATES.md:50` says adaptive adjusts mid-game from performance
  - `Docs/GAME_GENERATION.md:25` through `Docs/GAME_GENERATION.md:27` preorders and shuffles during generation
- Impact: unclear whether adaptation is runtime or pre-generation.

5. “Paginated/filterable” endpoints are underspecified.
- Evidence:
  - `Docs/API_SURFACE.md:33`, `Docs/API_SURFACE.md:44`, `Docs/API_SURFACE.md:58`
- Impact: likely API/client drift (query params, defaults, ordering, response envelope).

6. Dashboard command is brittle and hard to maintain.
- Evidence:
  - `.claude/commands/dashboard.md:8` is a single long shell chain with many inline `find | xargs | awk` segments
- Impact: small path/layout changes will silently break one or more fields; difficult to debug.

### Low

1. Catalog release/review section is stale.
- Evidence:
  - `Docs/CATALOG.md:24` through `Docs/CATALOG.md:28` says no release/review docs
  - `Docs/CODE_REVIEW_2026-02-19.md` exists
- Impact: discoverability gap for governance/review artifacts.

2. Requirements mention export compliance but omit retention/deletion controls.
- Evidence:
  - `Docs/REQUIREMENTS.md:48`
- Impact: compliance requirements are incomplete for implementation planning.

## Recommended Priority Fix Order

1. Correct `.claude/commands/*.md` so automation matches current Swift/SPM repo layouts.
2. Align `README.md`, `Docs/ARCHITECTURE.md`, and `Docs/API_SURFACE.md` to one canonical “current state” architecture.
3. Resolve scoring and generation contradictions with explicit formulas, inputs, and edge-case rules.
4. Expand API contract details for pagination/filtering and error behavior.
5. Update `Docs/CATALOG.md` release/review index.
