# Alities — API Surface

## Base URL

- Dev: `http://127.0.0.1:8001/api/v1`
- Prod: TBD

## Authentication

All endpoints require JWT bearer token unless marked (public).

## Endpoints

### Version

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/version` | public | Server version, API version, min client version |

### Auth

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/auth/register` | public | Create account |
| POST | `/auth/login` | public | Get JWT token |
| POST | `/auth/refresh` | token | Refresh JWT |
| GET | `/auth/me` | token | Current user profile |

### Game Templates

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/templates` | token | List templates (paginated) |
| POST | `/templates` | creator | Create template |
| GET | `/templates/{id}` | token | Get template detail |
| PUT | `/templates/{id}` | creator | Update template |
| DELETE | `/templates/{id}` | creator | Delete template |
| POST | `/templates/{id}/preview` | creator | Generate preview instance |

### Topic Packs

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/packs` | token | List topic packs (paginated, filterable) |
| POST | `/packs` | creator | Create topic pack |
| GET | `/packs/{id}` | token | Get pack detail with questions |
| PUT | `/packs/{id}` | creator | Update pack metadata |
| DELETE | `/packs/{id}` | creator | Delete pack |
| POST | `/packs/{id}/questions` | creator | Add questions to pack |
| POST | `/packs/import` | creator | Bulk import questions (CSV/JSON) |

### Game Instances

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/games/generate` | token | Generate a game instance from template + packs |
| GET | `/games/{id}` | token | Get game instance detail |
| GET | `/games` | token | List available game instances |

### Gameplay

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/play/{game_id}/start` | token | Start a game session |
| POST | `/play/{session_id}/answer` | token | Submit an answer |
| POST | `/play/{session_id}/hint` | token | Request a hint |
| GET | `/play/{session_id}` | token | Get current session state |
| POST | `/play/{session_id}/complete` | token | End session, finalize score |

### Analytics & Leaderboards

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/stats/me` | token | Current player's statistics |
| GET | `/leaderboards/{game_id}` | token | Game leaderboard |
| GET | `/leaderboards/global` | public | Global leaderboard |

### Admin

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/admin/content/review` | admin | Content moderation queue |
| POST | `/admin/content/{id}/approve` | admin | Approve submitted content |
| POST | `/admin/content/{id}/reject` | admin | Reject submitted content |
