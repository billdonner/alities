# Alities — Scoring Spec (v1.1)

> **Status: PARTIALLY IMPLEMENTED** — Per-mode scoring is implemented in alities-mobile. Speed bonus formula, streak multipliers, hint penalties, leaderboards, and player statistics are planned for a future version.

## Overview

Scoring rewards correct answers, speed, and consistency. Each game mode has its own scoring rules.

## Implemented Scoring (v1.1)

These scoring rules are live in the iOS app:

| Mode | Score Formula | Result Display |
|------|--------------|----------------|
| Classic | +1 per correct answer | Score / total, percentage (green/orange/red) |
| Speed Round | +ceil(seconds remaining) per correct, 0 for wrong/timeout | Total points, correct count, high score badge |
| Streak | Consecutive correct count | Streak number, flame icon, high score badge |
| Daily Challenge | Correct out of 5 | 0-5 stars, emoji grid, ShareLink, 28-day calendar |

### Speed Round Scoring
- 15-second countdown per question
- Points = `max(0, ceil(timeRemaining))` on correct answer
- Wrong answer or timeout (0 seconds) = 0 points
- Total score = sum of all question points
- High score persisted in UserDefaults

### Streak Scoring
- Score = consecutive correct answers (increments by 1)
- Wrong answer ends the game immediately
- High score persisted in UserDefaults

### Daily Challenge Scoring
- Stars = number of correct answers (0-5)
- Emoji grid: green square for correct, red square for wrong
- Share text format: `Alities Daily YYYY-MM-DD\n🟩🟥🟩🟩🟥 3/5`
- 28-day calendar grid with color-coded results (green=5, yellow=3, orange=2, red=0-1)

## Planned Scoring (Future)

### Full Score Calculation

```
question_score = base_points x speed_bonus x streak_multiplier - hint_penalty
```

### Components

| Component | Formula | Description |
|-----------|---------|-------------|
| **Base points** | Template `points_correct` (default 100) | Awarded for correct answer |
| **Speed bonus** | `1.0 + (time_remaining / time_limit) x 0.5` | Up to 1.5x for fast answers |
| **Streak multiplier** | `min(max(streak - 1, 0) x template.streak_multiplier + 1.0, 3.0)` | No bonus on 1st correct; grows from 2nd onward, capped at 3x |
| **Hint penalty** | `template.hint_cost_points x hints_used` | Deducted per hint used on this question |
| **Wrong answer** | `template.points_wrong` (default 0) | Usually 0, but templates can set negative |

## Leaderboards (Planned)

| Scope | Description | Retention |
|-------|-------------|-----------|
| **Game** | Top scores for a specific game instance | Permanent |
| **Daily** | Top scores across all daily challenges | 30 days rolling |
| **Global** | All-time top scores across all games | Permanent |
| **Category** | Top scores filtered by topic category | Permanent |

## Player Statistics (Planned)

Tracked per player:

| Stat | Description |
|------|-------------|
| `total_games` | Number of completed game sessions |
| `total_questions` | Questions answered |
| `accuracy` | Correct / total questions |
| `avg_speed` | Average time to answer (seconds) |
| `best_streak` | Longest consecutive correct streak |
| `total_score` | Lifetime accumulated score |
| `favorite_category` | Most-played topic category |
| `games_by_format` | Breakdown by game format type |
