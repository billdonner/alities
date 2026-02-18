# Alities — Glossary

| Term | Definition |
|------|-----------|
| **Game Template** | A reusable game format definition (quiz show, flashcard drill, bracket elimination, etc.) with configurable rules |
| **Topic Pack** | A versioned collection of trivia questions organized around a subject area |
| **Question** | A single trivia item: prompt, correct answer, wrong answers, difficulty rating, tags |
| **Game Instance** | A generated playable game combining a template with one or more topic packs and configuration |
| **Game Session** | A single player's play-through of a game instance, tracking score and progress |
| **Creator** | A user who designs game templates and/or authors topic packs |
| **Player** | A user who plays generated game instances |
| **Studio** | The web-based designer (alities-studio) where creators build templates and manage topic packs |
| **Engine** | The backend service (alities-engine) that stores content, generates games, and manages gameplay |
| **Seed** | A deterministic random seed used to generate reproducible game instances |
| **Difficulty Curve** | The algorithm that selects questions of increasing difficulty throughout a game session |
| **Streak** | Consecutive correct answers, used for bonus scoring |
| **Hint** | An in-game assist that reveals partial information about a question (costs points) |
| **Format Type** | The category of game template: `quiz_show`, `flashcard`, `bracket`, `speed_round`, etc. |
| **Quality Score** | An automated rating of a question's suitability based on clarity, uniqueness, and answer validity |
