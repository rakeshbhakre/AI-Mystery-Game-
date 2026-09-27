# THE TRUTH ENGINE — MVP README

## Overview
The Truth Engine is an AI-powered emergent social mystery game. There is one objective truth, but every NPC can have a different belief. The player changes beliefs through natural-language actions, and the world responds.

## Features (MVP)
- 1 mystery (Artifact Theft at Aurelia Museum)
- 8 unique NPCs with structured personalities, goals, fears, secrets, and relationships
- Deterministic truth engine (LLM never owns truth)
- Memory engine (direct observation, second-hand, inference, rumor — tracked separately)
- Belief engine (deterministic updates with bounded confidence 0.0–1.0)
- Relationship engine (updates only by actual events)
- Evidence system (authentic and fabricated evidence, deterministic properties)
- Rumor propagation (original claim preserved, transformations tracked)
- Event log (immutable timeline)
- NPC autonomy (decision engine evaluates goals, beliefs, relationships, evidence, location)
- Natural-language interaction (LLM intent parser with validation)
- Dynamic ending calculation (derived from world state, not hardcoded)
- Developer/debug mode (belief graph, relationship graph, event timeline, rumor propagation)

## Prerequisites
- Node.js v18+
- SQLite3 (embedded, included)
- Optional: OpenRouter API key (`OPENROUTER_API_KEY`) for LLM integration

## Installation (Clean Setup)
```bash
git clone <repo>
cd truth-engine
npm install # server dependencies installed in src/server/
cp .env.example .env # add your OPENROUTER_API_KEY if using LLM
node src/server/db.js init # initialize SQLite database
```

## Running Locally
Start the backend:
```bash
node src/server/server.js
```
The server listens on `http://localhost:3001`.

Open `src/client/index.html` in a browser, or navigate to `http://localhost:3001/` once the server is running.

## Testing
```bash
node --test src/server/tests/
```
For full automated tests:
```bash
npm run test # from src/server/
```

## API Endpoints
- `POST /api/game` — Create new game
- `GET /api/game/:id` — Get safe game state
- `POST /api/game/:id/action` — Player action
- `GET /api/game/:id/debug` — Developer/debug data (beliefs, relationships, events, rumors)
- `GET /api/health` — Health check

## Environment Variables
- `PORT` — Server port (default 3001)
- `NODE_ENV` — development / production
- `OPENROUTER_API_KEY` — OpenRouter API key for LLM calls
- `OPENROUTER_API_URL` — OpenRouter endpoint URL
- `DEBUG` — Enable debug mode

## Security
- `.env` is excluded from repository (`.gitignore`).
- `.env.example` provides template only.
- API keys never exposed in frontend.
- Server-side AI calls only.
- Input validation enforced before any state mutation.
- Request size limit (`express.json({ limit: '10kb' })`).
- Safe logging: no secrets in logs.

## Demo Sequence (Hackathon Mode)
1. Start new game (`POST /api/game`)
2. Player talks to Maya (`action`: talk)
3. Player accuses Rahul (`action`: accuse target `npc_001`)
4. Maya partially believes the player (`belief confidence updates`)
5. Maya tells Arjun (`rumor propagation event`)
6. Arjun becomes suspicious (`belief update` via rumor evaluation)
7. Player discovers CCTV evidence (`evidence_001`)
8. Player shows evidence to Arjun (`action`: show_evidence)
9. NPC beliefs change (`belief engine updates`)
10. NPC independently takes action (`decision engine` evaluates)
11. World state changes (`events` updated, `relationships` updated)
12. Player sees consequences (`event timeline`, `belief graph`, `relationship graph`)

## Architecture Docs
- `docs/ARCHITECTURE.md` — Component diagram, rules, phase plan
- `docs/GAME_DESIGN.md` — Setting, mystery, mechanics, endings
- `docs/AI_SYSTEM.md` — LLM design, prompts, safety, anti-hallucination
- `docs/DATABASE.md` — SQLite schema, constraints, integrity
- `docs/API.md` — Endpoint specification, validation rules, response format

## Known Limitations (MVP)
- LLM integration requires `OPENROUTER_API_KEY` for full natural-language parsing; without it, the deterministic fallback handles basic structured actions.
- Belief updates are wired for accusation events; full memory retrieval and rumor propagation updates are implemented as modules but may need additional wiring for every interaction type.
- The React frontend has dependency issues (`react-scripts` build errors with `ajv-keywords`); a polished vanilla JS frontend (`src/client/index.html`) is provided as the working interface.
- Autonomous NPC actions run via the decision engine module but are not triggered by a background timer in this MVP; they are evaluated upon player interaction or can be triggered manually via the debug interface.

## Next Steps
- Wire full memory creation for all action types
- Implement automatic rumor propagation timer
- Add automated tests for belief arithmetic, rumor transformation, and LLM retry behavior
- Expand to additional villages and mysteries
