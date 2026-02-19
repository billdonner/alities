# Alities — Scoring Spec (v1.0)

## Overview

Scoring rewards correct answers, speed, and consistency. Templates configure the base values; the engine computes final scores during gameplay.

## Score Calculation

```
question_score = base_points × speed_bonus × streak_multiplier - hint_penalty
```

### Components

| Component | Formula | Description |
|-----------|---------|-------------|
| **Base points** | Template `points_correct` (default 100) | Awarded for correct answer |
| **Speed bonus** | `1.0 + (time_remaining / time_limit) × 0.5` | Up to 1.5x for fast answers |
| **Streak multiplier** | `min(max(streak - 1, 0) × template.streak_multiplier + 1.0, 3.0)` | No bonus on 1st correct; grows from 2nd onward, capped at 3x |
| **Hint penalty** | `template.hint_cost_points × hints_used` | Deducted per hint used on this question |
| **Wrong answer** | `template.points_wrong` (default 0) | Usually 0, but templates can set negative |

## Streaks

- Streak increments on each consecutive correct answer
- Streak resets to 0 on a wrong answer or skipped question
- Streak multiplier applies from the 2nd consecutive correct answer onward

## Leaderboards

| Scope | Description | Retention |
|-------|-------------|-----------|
| **Game** | Top scores for a specific game instance | Permanent |
| **Daily** | Top scores across all daily challenges | 30 days rolling |
| **Global** | All-time top scores across all games | Permanent |
| **Category** | Top scores filtered by topic category | Permanent |

## Player Statistics

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
