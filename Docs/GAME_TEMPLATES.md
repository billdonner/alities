# Alities — Game Templates Spec (v1.1)

> **Status: PARTIALLY IMPLEMENTED** — Four game modes (Classic, Speed Round, Streak, Daily Challenge) are implemented in alities-mobile as client-side logic. The full template configuration system (hints, difficulty progression, adaptive mode) is planned for a future version.

## Overview

Game templates define reusable game formats. A template specifies the rules, scoring, UI layout hints, and progression logic — but contains no trivia content. Content comes from topic packs at generation time.

## Implemented Game Modes (v1.1)

These modes are live in the iOS app (alities-mobile):

| Mode | Questions | Timer | Scoring | Persistence |
|------|-----------|-------|---------|-------------|
| Classic | All available, shuffled | None | 1 point per correct | None |
| Speed Round | 10 | 15s countdown per question | Points = seconds remaining | High score (UserDefaults) |
| Streak | Unlimited | None | Consecutive correct count | High score (UserDefaults) |
| Daily Challenge | 5 (deterministic) | None | 0-5 stars | Result + 28-day calendar (UserDefaults) |

### Classic
Standard quiz mode. All categories, shuffled questions, answer at your own pace. Score = correct answer count. Percentage shown at end (green/orange/red).

### Speed Round
10 questions with an animated circular countdown timer (green → orange → red). Points awarded = seconds remaining when answered correctly. Wrong answer or timeout = 0 points. High score persisted via UserDefaults.

### Streak
Endless mode — answer until you get one wrong. Score = consecutive correct answers. Flame icon grows with streak count. Wrong answer ends the game immediately (brief display, then result screen). High score persisted.

### Daily Challenge
5 deterministic questions per day, selected via SplitMix64 PRNG seeded from date hash (djb2). Same questions for all players on the same day. Result: 0-5 stars, Wordle-style emoji grid, ShareLink for sharing. Once-per-day gate enforced by ScoreStore. 28-day calendar grid shows history.

## Implementation Details

- **Daily seeding**: `DailyChallengeService` uses Fisher-Yates shuffle with `SeededRandomNumberGenerator(seed: djb2(dateString))`
- **Score persistence**: `ScoreStore` (@Observable) wraps UserDefaults with keys `alities.streakHighScore`, `alities.speedRoundHighScore`, `alities.dailyResults`
- **Timer**: `Timer.scheduledTimer` at 0.1s interval for smooth ring animation
- **No engine changes required**: All mode logic is client-side in the iOS app

## Planned Format Types (Future)

| Format | Description | Players |
|--------|-------------|---------|
| `quiz_show` | Sequential questions with multiple choice, timed | 1+ |
| `flashcard` | Study mode — show question, reveal answer, self-rate | 1 |
| `bracket` | Tournament elimination — questions paired in rounds | 2+ |

## Template Configuration (Planned)

```json
{
  "name": "Classic Quiz Show",
  "format_type": "quiz_show",
  "rules": {
    "questions_per_game": 20,
    "time_per_question_seconds": 30,
    "points_correct": 100,
    "points_wrong": 0,
    "streak_multiplier": 1.5,
    "max_hints": 3,
    "hint_cost_points": 50,
    "difficulty_progression": "linear",
    "categories_balanced": true
  },
  "ui_hints": {
    "show_timer": true,
    "show_score": true,
    "show_streak": true,
    "animation_style": "slide"
  }
}
```

## Difficulty Progression Modes (Planned)

| Mode | Description |
|------|-------------|
| `flat` | Random difficulty throughout |
| `linear` | Steadily increasing difficulty |
| `wave` | Alternating easy/hard clusters |
| `adaptive` | Dynamically adjusts upcoming question difficulty based on player performance |

## Template Lifecycle (Planned)

1. **Draft** — creator is editing, not visible to others
2. **Published** — available for game generation
3. **Archived** — no longer available for new games, existing instances still playable
