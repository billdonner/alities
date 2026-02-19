# Alities — Game Generation Spec (v1.0)

## Overview

The generation service combines a game template with one or more topic packs to produce a playable game instance. The algorithm selects and orders questions based on the template's rules.

## Generation Input

```json
{
  "template_id": "uuid",
  "topic_pack_ids": ["uuid", "uuid"],
  "player_id": "uuid (optional)",
  "config_overrides": {
    "questions_per_game": 15
  },
  "seed": 42
}
```

- `player_id` is optional. When provided, the "recently seen" filter in step 2 uses this player's history. When omitted, the history filter step is skipped.

## Question Selection Algorithm

1. **Pool** — Gather all questions from the specified topic packs
2. **Filter** — Remove questions the player has seen recently (if player context provided)
3. **Balance** — Ensure category distribution matches template config
4. **Curve** — Order by difficulty according to the template's `difficulty_progression` mode
5. **Trim** — Select exactly `questions_per_game` questions
6. **Shuffle within bands** — Randomize order within same-difficulty groups (seeded)

## Deterministic Seeding

- If a `seed` is provided, the same inputs (including the player's history snapshot at generation time, if `player_id` is given) always produce the same game
- Used for daily challenges (seed = date) and competitive modes (all players get same questions)
- If no seed is provided, a random one is generated and stored with the instance

## Game Instance Output

```json
{
  "id": "uuid",
  "template_id": "uuid",
  "topic_pack_ids": ["uuid"],
  "seed": 42,
  "questions": [
    {
      "question_id": "uuid",
      "position": 1,
      "time_limit_seconds": 30
    }
  ],
  "status": "ready",
  "created_at": "2026-02-18T00:00:00Z"
}
```

## Instance Lifecycle

| Status | Description |
|--------|-------------|
| `ready` | Generated, waiting for players |
| `active` | At least one session in progress |
| `completed` | All sessions finished |
| `expired` | Past time-to-live, no longer playable |
