# Alities — Game Templates Spec (v1.0)

## Overview

Game templates define reusable game formats. A template specifies the rules, scoring, UI layout hints, and progression logic — but contains no trivia content. Content comes from topic packs at generation time.

## Built-in Format Types

| Format | Description | Players |
|--------|-------------|---------|
| `quiz_show` | Sequential questions with multiple choice, timed | 1+ |
| `flashcard` | Study mode — show question, reveal answer, self-rate | 1 |
| `bracket` | Tournament elimination — questions paired in rounds | 2+ |
| `speed_round` | Maximum questions in fixed time, shorter per-question timer | 1+ |
| `daily_challenge` | One curated game per day, same for all players | 1+ |

## Template Configuration

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

## Difficulty Progression Modes

| Mode | Description |
|------|-------------|
| `flat` | Random difficulty throughout |
| `linear` | Steadily increasing difficulty |
| `wave` | Alternating easy/hard clusters |
| `adaptive` | Adjusts based on player performance mid-game |

## Template Lifecycle

1. **Draft** — creator is editing, not visible to others
2. **Published** — available for game generation
3. **Archived** — no longer available for new games, existing instances still playable
