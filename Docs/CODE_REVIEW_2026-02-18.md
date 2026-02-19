# Alities Ecosystem — Comprehensive Code Review

**Date:** 2026-02-18
**Scope:** All 4 repos (hub, engine, studio, mobile) — every source file, test, config, and doc
**Total files reviewed:** 60+ source files, 19 docs, 8 config files

---

## Executive Summary

| Repo | Quality | Build | Tests | Key Risk |
|------|---------|-------|-------|----------|
| alities-engine | 7.5/10 | PASS | 74/74 pass | Credential logging, unbounded caches |
| alities-studio | 2/10 | FAIL | 1/1 pass (weak) | Template boilerplate — no real functionality |
| alities-mobile | 5/10 | PASS | 3/3 pass | Skeleton app — no API integration |
| alities (hub) | 7/10 | n/a | n/a | Some doc inconsistencies remain |

**Overall ecosystem status:** Engine is production-quality with known issues. Studio and mobile are early-stage skeletons. Documentation is mostly aligned after recent fixes but has remaining gaps.

---

## Part 1 — alities-engine (Swift CLI/Daemon)

### 1.1 Project Stats

| Metric | Value |
|--------|-------|
| Source LoC | ~4,060 |
| Test LoC | ~1,086 |
| Tests | 74 (all passing) |
| Dependencies | 6 (postgres-nio, argument-parser, swift-log, async-http-client, GRDB, swift-nio) |
| Swift version | 5.9+ |
| Platform | macOS 14+ |

### 1.2 Architecture Assessment

**Strengths:**
- Clean separation: Models / Providers / Services / Profile / Commands
- Actor-based concurrency (PostgresService, TriviaGenDaemon, SimilarityService) — thread-safe
- Defensive JSON decoding — Challenge model handles null/missing fields with sensible defaults
- Dual-database design: PostgreSQL for daemon mode, SQLite/GRDB for profile commands
- Pluggable provider system (5 trivia sources, easy to extend)
- Dual-write capability (file + database simultaneously)
- Good error types with localized descriptions

**Weaknesses:**
- Mixed error handling: some `try?` silently swallows errors, some thrown, some logged
- Unbounded in-memory caches (SimilarityService signatures)
- No graceful signal handling (SIGTERM/SIGINT)
- Limited API validation on HTTP control endpoints

### 1.3 Critical Issues

| # | Severity | File | Line | Issue |
|---|----------|------|------|-------|
| E1 | **CRITICAL** | RunCommand.swift | ~91 | Database password logged in plain text during startup |
| E2 | **HIGH** | RunCommand.swift | ~183 | Force unwrap of optional `outputFile!` in log message — crash if nil |
| E3 | **HIGH** | SimilarityService.swift | 9 | Signature cache unbounded — memory leak in long-running daemon |
| E4 | **HIGH** | ControlServer.swift | 175 | `Foundation.exit(0)` on /stop — hard exit with no cleanup |
| E5 | **HIGH** | ControlServer.swift | ~151 | /harvest fires background tasks with no tracking — unlimited queue |

### 1.4 Medium Issues

| # | Severity | File | Line | Issue |
|---|----------|------|------|-------|
| E6 | MEDIUM | SimilarityService.swift | 130 | AI check errors silently caught — hard to debug failures |
| E7 | MEDIUM | FileImportProvider.swift | 38 | `try? fileManager.moveItem()` silently ignores move failures |
| E8 | MEDIUM | FileImportProvider.swift | 95 | Pads incorrect answers with "Unknown" — confusing for CSV import |
| E9 | MEDIUM | Report.swift | 298 | Force unwrap `a.correctPositionDistribution[key]!` — safe but not defensive |
| E10 | MEDIUM | TriviaDatabase.swift | 153 | Force unwrap `cat.id!` after existence check — should use guard |
| E11 | MEDIUM | ControlServer.swift | ~194 | /import loads entire file into memory — could OOM on large files |
| E12 | MEDIUM | ControlServer.swift | ~142 | No request validation on /harvest — accepts any categories array |

### 1.5 Low Issues

| # | Severity | File | Issue |
|---|----------|------|-------|
| E13 | LOW | FileImportProvider.swift | CSV parser assumes UTF-8 encoding |
| E14 | LOW | FileImportProvider.swift | No file locking — race condition if file modified during read |
| E15 | LOW | SimilarityService.swift | No rate limiting on AI similarity checks — could overwhelm OpenAI |
| E16 | LOW | TriviaGenDaemon.swift | Hard sleep between cycles — can't interrupt for graceful shutdown |
| E17 | LOW | TriviaGenDaemon.swift | Duplicate detection only checks last N questions — could miss old dupes |
| E18 | LOW | JServiceProvider.swift | Mutable `categoryIndex` — safe in practice (single-threaded loop) but not thread-safe by design |
| E19 | LOW | RunCommand.swift | DB credentials via CLI args — visible in process list, should prefer env vars |
| E20 | LOW | CategoryMap.swift | Aggressive alias mapping loses category granularity (science/nature/animals all → "Science & Nature") |

### 1.6 Test Coverage Gaps

| Module | Tests | Coverage |
|--------|-------|----------|
| TriviaQuestion | 12 | Excellent |
| Challenge | 8 | Excellent |
| GameDataTransformer | 5 | Good |
| ProfileModels | 6 | Good |
| CategoryMap | 8 | Excellent |
| TriviaDatabase | 5+ | Good |
| Report | 14+ | Good |
| **Providers (all 5)** | **0** | **None — major gap** |
| **TriviaGenDaemon** | **0** | **None — major gap** |
| **ControlServer** | **0** | **None — major gap** |
| **PostgresService** | **0** | **None — major gap** |
| **SimilarityService** | **0** | **None — major gap** |

**Recommendation:** Add provider tests with mock HTTP, daemon state machine tests, and control server route tests. These are the most complex modules with zero coverage.

### 1.7 Security Review

| Area | Status | Notes |
|------|--------|-------|
| SQL injection | SAFE | postgres-nio handles parameterization |
| Credential handling | POOR | Password logged plaintext (E1), CLI args visible (E19) |
| API key handling | GOOD | OpenAI key validated, never logged |
| Network exposure | GOOD | Control server localhost-only |
| Input validation | POOR | /harvest accepts unvalidated input (E12) |
| File operations | FAIR | No locking, silent failures on move |

---

## Part 2 — alities-studio (React/TypeScript)

### 2.1 Project Stats

| Metric | Value |
|--------|-------|
| Source LoC | ~160 |
| Test LoC | ~10 |
| Tests | 1 (passing, but weak) |
| Dependencies | react 19, react-dom 19, vite 7, vitest 4, typescript 5.9 |
| Node version | Current |

### 2.2 Current State: Template Boilerplate

The entire app is the **unchanged Vite + React template**. There is no game designer, no player, no API integration, no state management.

```
src/
├── App.tsx           ← Vite counter demo
├── App.test.tsx      ← Smoke test (unused import breaks build)
├── main.tsx          ← Entry point
├── App.css           ← Template styles
├── index.css         ← Template styles
├── assets/react.svg  ← Template logo
└── test/setup.ts     ← Testing library setup
```

### 2.3 Critical Issues

| # | Severity | File | Issue |
|---|----------|------|-------|
| S1 | **CRITICAL** | App.test.tsx | Unused import `screen` — **breaks `npm run build`** (tsconfig strict) |
| S2 | **HIGH** | App.tsx | Template boilerplate — no game designer functionality |
| S3 | **HIGH** | package.json | No `api:generate` script (previously documented as existing) |
| S4 | **HIGH** | — | No API client — cannot communicate with alities-engine |
| S5 | **HIGH** | — | No state management (Redux/Zustand) for complex UI state |

### 2.4 Medium Issues

| # | Severity | File | Issue |
|---|----------|------|-------|
| S6 | MEDIUM | App.tsx | External links missing `rel="noopener noreferrer"` — security risk |
| S7 | MEDIUM | App.test.tsx | Test only checks `document.body` exists — tests nothing real |
| S8 | MEDIUM | — | No environment variable config (VITE_API_BASE_URL etc.) |
| S9 | MEDIUM | — | No error boundaries or Suspense — errors crash entire app |
| S10 | MEDIUM | eslint.config.js | Using `recommended` not `strictTypeChecked` |

### 2.5 Build & Test Status

| Check | Status |
|-------|--------|
| `npm run build` | FAIL (unused import in test file) |
| `npm run lint` | FAIL (same issue) |
| `npx vitest run` | PASS (1 test, weak assertion) |
| `npm run dev` | PASS (Vite dev server starts on 9850) |

### 2.6 What's Missing for MVP

- Game template browser/editor
- Topic pack viewer
- Question editor
- Game preview/player
- API client for engine communication
- State management
- Routing (react-router)
- Authentication (if needed)
- Error handling and loading states

---

## Part 3 — alities-mobile (SwiftUI iOS)

### 3.1 Project Stats

| Metric | Value |
|--------|-------|
| Source LoC | ~137 |
| Test LoC | ~59 |
| Tests | 3 (all passing) |
| Dependencies | None |
| Swift version | 5.9+ |
| Platforms | iOS 17+, macOS 14+ |

### 3.2 Current State: Skeleton App

The app has a splash screen (ContentView), two model structs (TriviaQuestion, GameInstance), and 3 Codable decoding tests. No gameplay, no networking, no navigation.

```
Alities/Sources/
├── App/AlitiesApp.swift       ← Entry point (no @main — SPM workaround)
├── Models/GameModels.swift    ← TriviaQuestion + GameInstance structs
└── Views/ContentView.swift    ← Splash screen only
```

### 3.3 Issues

| # | Severity | File | Issue |
|---|----------|------|-------|
| M1 | **HIGH** | CLAUDE.md | References nonexistent `Alities.xcodeproj` — xcodebuild command won't work |
| M2 | **HIGH** | — | No API client (URLSession) — cannot fetch games from engine |
| M3 | **HIGH** | — | No ViewModel / @StateObject — no state management |
| M4 | MEDIUM | GameModels.swift:19 | `createdAt: Date` — brittle if API returns non-ISO8601 format |
| M5 | MEDIUM | ContentView.swift:8 | Image missing `.accessibilityLabel()` — fails WCAG 2.1 |
| M6 | MEDIUM | GameModels.swift | No validation that `correctAnswer` is in `answers` array |
| M7 | MEDIUM | ContentView.swift:6 | NavigationStack with no destinations — incomplete navigation |
| M8 | LOW | GameModelsTests.swift:16 | Force unwrap `.data(using: .utf8)!` in tests |
| M9 | LOW | GameModels.swift | Implicit Identifiable conformance — explicit is preferred |

### 3.4 Test Coverage

| Component | Tests | Coverage |
|-----------|-------|----------|
| TriviaQuestion decoding | 2 | 50% (with/without hint) |
| GameInstance decoding | 1 | 25% (basic only) |
| **ContentView** | **0** | **None** |
| **AlitiesApp** | **0** | **None** |
| Error cases | 0 | None (no invalid JSON tests) |
| Edge cases | 0 | None (empty arrays, wrong types) |

### 3.5 What's Missing for MVP

- API client / networking layer
- ViewModel architecture
- Game selection list view
- Gameplay view (question display, answer selection)
- Score tracking and results
- Error handling and retry logic
- Navigation flow
- Accessibility labels throughout

---

## Part 4 — alities (Hub Docs)

### 4.1 Documentation Inventory

| Category | Files | Status |
|----------|-------|--------|
| Core docs | CLAUDE.md, README.md, agents.md | Mostly aligned |
| Feature specs | 7 docs in Docs/ | Internally consistent |
| Automation skills | 5 in .claude/commands/ | Recently fixed |
| Reference | GLOSSARY.md, CATALOG.md | Complete |
| Reviews | CODE_REVIEW_2026-02-19.md | Present |

### 4.2 Remaining Doc Issues

| # | Severity | File | Issue |
|---|----------|------|-------|
| D1 | **HIGH** | .claude/settings.local.json | Pre-authorizes `uv run pytest` and `python3` commands that don't apply to Swift engine |
| D2 | MEDIUM | Docs/REQUIREMENTS.md | Features documented but no detailed spec: multiplayer, session persistence, content moderation |
| D3 | MEDIUM | Docs/GLOSSARY.md | Missing terms: Provider, Control Server, Dual Database, Category Normalization |
| D4 | MEDIUM | Docs/GAME_GENERATION.md | Determinism claim requires history snapshot to be preserved — not explicitly stated |
| D5 | MEDIUM | Docs/GAME_TEMPLATES.md | Adaptive mode partially specified — unclear if runtime adaptation is client-side or server-side |
| D6 | LOW | Docs/TOPIC_PACKS.md | Version scheme not defined (semver? timestamps? auto-increment?) |
| D7 | LOW | Docs/CATALOG.md | Self-referential categorization (catalog lists itself in "Core Docs") |
| D8 | LOW | agents.md | "Source of truth" statement unclear — engine has no OpenAPI, so how do clients sync? (answered later but confusing on first read) |

### 4.3 Cross-Document Consistency

| Check | Status | Notes |
|-------|--------|-------|
| Engine stack (Swift/SPM) | CONSISTENT | All docs agree |
| Daemon port (9847) | CONSISTENT | All docs agree |
| Test counts (74/1/3=78) | CONSISTENT | All docs agree (but may become stale) |
| Repo paths (~/alities*) | CONSISTENT | All docs agree |
| CLI subcommands (12) | CONSISTENT | agents.md and API_SURFACE.md match |
| Cross-project sync | CONSISTENT | No more openapi.json references (outside code review doc) |

---

## Part 5 — Cross-Cutting Concerns

### 5.1 Model Drift Risk

The three repos define trivia question models independently:

| Field | Engine (TriviaQuestion) | Mobile (TriviaQuestion) | Studio |
|-------|------------------------|------------------------|--------|
| id | String (computed) | String | n/a |
| question/text | String | String | n/a |
| answers/choices | [TriviaChoice] | [String] | n/a |
| correctAnswer | computed property | String | n/a |
| category | String | String | n/a |
| difficulty | Difficulty enum | String | n/a |
| hint | String? | String? | n/a |
| explanation | String? | not present | n/a |
| source | String? | not present | n/a |

**Risk:** Engine uses a richer model (enum difficulty, explanation, source) that mobile doesn't mirror. When mobile eventually consumes engine data, these differences will cause issues.

### 5.2 Data Flow Architecture

```
Engine CLI → exports JSON files (GameData format)
                    ↓
            Manual file transfer
                    ↓
    Studio (reads JSON) / Mobile (reads JSON)
```

There is no live API integration. Studio and mobile would consume exported files. This is fine for the current stage but will need a real API layer for production.

### 5.3 Security Posture

| Area | Engine | Studio | Mobile |
|------|--------|--------|--------|
| Credentials | POOR (plaintext logging) | n/a | n/a |
| Input validation | POOR (unvalidated endpoints) | n/a | n/a |
| Network security | GOOD (localhost only) | n/a | n/a |
| XSS/injection | SAFE | MINOR (missing rel attrs) | SAFE |
| Authentication | None (not needed for localhost) | None | None |

### 5.4 Test Coverage Summary

| Repo | Tests | Source LoC | Test LoC | Ratio | Grade |
|------|-------|-----------|----------|-------|-------|
| alities-engine | 74 | 4,060 | 1,086 | 0.27 | B (good for models, none for services) |
| alities-studio | 1 | 160 | 10 | 0.06 | F (placeholder test) |
| alities-mobile | 3 | 137 | 59 | 0.43 | C (models only, no views) |
| **Total** | **78** | **4,357** | **1,155** | **0.27** | |

---

## Prioritized Action Items

### Tier 1 — Fix Now (Blocking / Security)

| # | Repo | Action |
|---|------|--------|
| 1 | engine | Stop logging database password in plaintext (RunCommand.swift:~91) |
| 2 | engine | Fix force unwrap of `outputFile!` (RunCommand.swift:~183) |
| 3 | studio | Remove unused `screen` import from App.test.tsx — unblocks `npm run build` |
| 4 | mobile | Remove stale xcodebuild command from CLAUDE.md |
| 5 | hub | Clean stale Python allowlists from .claude/settings.local.json |

### Tier 2 — Fix Soon (Stability / Correctness)

| # | Repo | Action |
|---|------|--------|
| 6 | engine | Bound SimilarityService signature cache (LRU, max 10k entries) |
| 7 | engine | Replace `Foundation.exit(0)` in /stop with graceful shutdown |
| 8 | engine | Add logging to all `try?` error suppressions |
| 9 | engine | Add request validation on /harvest endpoint |
| 10 | engine | Add SIGTERM/SIGINT signal handling for graceful daemon shutdown |

### Tier 3 — Add Tests (Coverage)

| # | Repo | Action |
|---|------|--------|
| 11 | engine | Add provider tests with mock HTTP responses |
| 12 | engine | Add TriviaGenDaemon state machine tests |
| 13 | engine | Add ControlServer route tests |
| 14 | studio | Write real test that asserts component renders |
| 15 | mobile | Add error case tests (invalid JSON, missing fields) |
| 16 | mobile | Add ContentView snapshot/accessibility tests |

### Tier 4 — Architecture (When Building Features)

| # | Repo | Action |
|---|------|--------|
| 17 | studio | Build actual game designer UI (replace boilerplate) |
| 18 | studio | Add API client, state management, routing |
| 19 | mobile | Build API client (URLSession), ViewModel layer |
| 20 | mobile | Build game selection and gameplay views |
| 21 | mobile | Add accessibility labels to all views |
| 22 | all | Align question model fields across repos |

### Tier 5 — Documentation

| # | Repo | Action |
|---|------|--------|
| 23 | hub | Add specs for multiplayer, session persistence, content moderation |
| 24 | hub | Expand GLOSSARY.md with implementation terms |
| 25 | hub | Clarify adaptive difficulty: client-side or server-side runtime behavior? |
| 26 | hub | Define topic pack versioning scheme |

---

## Issue Counts by Severity

| Severity | Engine | Studio | Mobile | Hub | Total |
|----------|--------|--------|--------|-----|-------|
| CRITICAL | 1 | 1 | 0 | 0 | **2** |
| HIGH | 4 | 4 | 2 | 1 | **11** |
| MEDIUM | 7 | 5 | 3 | 5 | **20** |
| LOW | 8 | 0 | 2 | 3 | **13** |
| **Total** | **20** | **10** | **7** | **9** | **46** |

---

*Review conducted by Claude Opus 4.6. All source files in all 4 repos were read and analyzed.*
