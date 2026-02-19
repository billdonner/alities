# Alities — Glossary

## Implemented Concepts

| Term | Definition |
|------|-----------|
| **TriviaQuestion** | The engine's core question model: text, choices, correctChoiceIndex, category, difficulty, explanation, hint, source |
| **Challenge** | The exported player-facing question format: topic, pic (SF Symbol), question, answers, correct answer, explanation, hint, aisource |
| **GameDataOutput** | A JSON bundle containing an array of Challenge objects, used by studio and mobile clients |
| **Category** | A normalized subject area for grouping questions, with an SF Symbol icon and optional aliases |
| **Provider** | A trivia data source (OpenTriviaDB, TheTriviaAPI, jService, AI Generator, File Import) |
| **Control Server** | The NIO HTTP server on localhost:9847 that exposes daemon status, categories, game data, and control endpoints |
| **Dual Database** | The engine's architecture: PostgreSQL for daemon acquisition, SQLite/GRDB for local profile operations |
| **Category Normalization** | Mapping raw category names from providers to canonical categories via `CategoryMap` aliases |
| **Difficulty** | Question difficulty rating: `easy`, `medium`, or `hard` (3-level string enum) |
| **Streak** | Consecutive correct answers — currently tracked by mobile app as a simple counter |
| **Hint** | Optional text on a question that reveals partial information |
| **Studio** | The web-based app (alities-studio) for browsing engine content and (planned) designing games |
| **Engine** | The Swift CLI/daemon (alities-engine) that acquires, stores, normalizes, and exports trivia content |

## Planned Concepts (not yet implemented)

| Term | Definition |
|------|-----------|
| **Game Template** | A reusable game format definition (quiz show, flashcard drill, bracket elimination, etc.) with configurable rules |
| **Topic Pack** | A versioned collection of trivia questions organized around a subject area |
| **Game Instance** | A generated playable game combining a template with one or more topic packs and configuration |
| **Game Session** | A single player's play-through of a game instance, tracking score and progress |
| **Creator** | A user who designs game templates and/or authors topic packs |
| **Player** | A user who plays generated game instances |
| **Seed** | A deterministic random seed used to generate reproducible game instances |
| **Difficulty Curve** | The algorithm that selects questions of increasing difficulty throughout a game session |
| **Format Type** | The category of game template: `quiz_show`, `flashcard`, `bracket`, `speed_round`, etc. |
| **Quality Score** | An automated rating of a question's suitability based on clarity, uniqueness, and answer validity |
