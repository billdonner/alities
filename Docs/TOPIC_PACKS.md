# Alities — Topic Packs Spec (v1.0)

> **Status: PLANNED** — This spec describes the target design. Topic packs as first-class versioned entities are not yet implemented. The engine currently imports raw trivia questions into SQLite/PostgreSQL without pack-level grouping, versioning, or quality scoring.

## Overview

Topic packs are versioned collections of trivia questions. Creators author or import questions, and the engine uses them as content sources when generating game instances.

## Question Schema

### Current Engine Format (TriviaQuestion)

```json
{
  "text": "What is the capital of France?",
  "choices": [
    {"text": "Paris", "isCorrect": true},
    {"text": "London", "isCorrect": false},
    {"text": "Berlin", "isCorrect": false},
    {"text": "Madrid", "isCorrect": false}
  ],
  "correctChoiceIndex": 0,
  "category": "Geography",
  "difficulty": "easy",
  "source": "opentdb",
  "explanation": "Paris has been the capital of France since the 10th century.",
  "hint": "It's known as the City of Light."
}
```

### Planned Target Format

> The following schema is the target for when Topic Packs become first-class entities. It is not yet implemented.

```json
{
  "text": "What is the capital of France?",
  "correct_answer": "Paris",
  "wrong_answers": ["London", "Berlin", "Madrid"],
  "difficulty": 1,
  "category": "Geography",
  "tags": ["europe", "capitals"],
  "source": "manual",
  "explanation": "Paris has been the capital of France since the 10th century."
}
```

## Difficulty Scale

### Current (Engine)

| Label | Description |
|-------|-------------|
| `easy` | Common knowledge, most people would know |
| `medium` | Requires some subject familiarity |
| `hard` | Specialist knowledge or tricky wording |

### Planned (5-level)

> Not yet implemented. The engine currently uses a 3-level string enum.

| Level | Label | Description |
|-------|-------|-------------|
| 1 | Easy | Common knowledge, most people would know |
| 2 | Medium | Requires some subject familiarity |
| 3 | Hard | Specialist knowledge or tricky wording |
| 4 | Expert | Deep expertise required |
| 5 | Master | Obscure facts, stumps most experts |

## Import Formats

### CSV
```csv
text,correct_answer,wrong_answer_1,wrong_answer_2,wrong_answer_3,difficulty,category
"What is 2+2?","4","3","5","6",1,"Math"
```

### JSON
Array of question objects matching the schema above.

## Quality Scoring

Each question receives an automated quality score (0-100) based on:

| Factor | Weight | Description |
|--------|--------|-------------|
| Uniqueness | 30% | No near-duplicates in the pack |
| Answer validity | 25% | Correct answer is unambiguous |
| Distractor quality | 25% | Wrong answers are plausible but clearly wrong |
| Metadata completeness | 20% | Difficulty, category, tags all present |

Questions scoring below 50 are flagged for review.

## Versioning

- Topic packs are versioned (v1, v2, ...)
- Existing game instances pin to a specific version
- New games always use the latest version
- Creators can publish updates without breaking in-progress games
