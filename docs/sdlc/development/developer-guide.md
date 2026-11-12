# Developer Guide

> Maintained by QuorQ AI SDLC. Text between `sdlc:auto` markers is regenerated; anything outside them is yours and is kept.

<!-- sdlc:auto:start status -->
_Up to date._
<!-- sdlc:auto:end status -->

<!-- sdlc:auto:start source -->
Generated from commit `276f26d` · model `claude-sonnet-5`
<!-- sdlc:auto:end source -->

<!-- sdlc:auto:start overview -->
Book Lending is a two-part, AI-SDLC-generated repository: a frontend Vite + React 19 SPA with approved, reviewed screens that currently render seeded sample data, and a FastAPI 0.115 backend scaffold that has no approved architecture yet beyond a health check. The backend uses SQLAlchemy (SQLite by default, swappable via DATABASE_URL) with no models or real routes implemented, while docker-compose provides optional Postgres and Redis services for when a real architecture is built out. The frontend has a lib/api.ts client (apiFetch, bearer-token session handling) ready to wire screens to the backend, but no screen currently calls it.
<!-- sdlc:auto:end overview -->

<!-- sdlc:auto:start prerequisites -->
- Node.js (for Vite 6 / React 19 / Playwright tooling)
- npm
- Python 3.12
- pip
- Docker and Docker Compose (optional, for Postgres and Redis services in docker-compose.yml)
<!-- sdlc:auto:end prerequisites -->

<!-- sdlc:auto:start setup -->
1. Clone the repository
2. Copy .env.example to .env in the repo root and fill in any required values
3. cd frontend && npm install
4. cd backend && pip install -r requirements.txt
5. (Optional) From the repo root, run `docker compose up -d` to start the Postgres (db) and Redis (redis-queue) services if you want something other than the default local SQLite file
6. Start the backend: cd backend && uvicorn app.main:app --reload
7. Start the frontend: cd frontend && npm run dev
<!-- sdlc:auto:end setup -->

<!-- sdlc:auto:start run -->
| Name | Command | Description |
|---|---|---|
| frontend dev | `cd frontend && npm run dev` | Starts the Vite dev server for the React SPA |
| frontend build | `cd frontend && npm run build` | Bundles the app for production (does not typecheck) |
| frontend preview | `cd frontend && npm run preview` | Serves the production build locally |
| backend dev | `cd backend && uvicorn app.main:app --reload` | Runs the FastAPI service with auto-reload; creates SQLite tables on startup from app/models.py |
| docker services | `docker compose up -d` | Starts the Postgres database and Redis queue containers defined in docker-compose.yml |
<!-- sdlc:auto:end run -->

<!-- sdlc:auto:start structure -->
| Path | Purpose |
|---|---|
| `.devcontainer` | Dev container configuration (devcontainer.json) for a consistent editor/runtime environment |
| `.quorq` | Quorq AI SDLC platform metadata (handoff.json), not application code |
| `backend` | FastAPI service: app code, Dockerfile, requirements.txt, lint/test config |
| `backend/app` | Application package: main.py (entrypoint/CORS/lifespan), database.py (engine/session), models.py (SQLAlchemy models, currently empty), schemas.py (Pydantic schemas/StubResponse), services.py, routers/ (per-component routers, currently empty) |
| `backend/tests` | Pytest suite: tests/integration (e.g. health check test) and tests/unit, with shared fixtures in conftest.py |
| `frontend` | Vite + React 19 SPA with the approved, reviewed UI screens |
| `frontend/src` | App source: main.tsx entrypoint, index.css, test-setup.ts, and lib/ (api.ts client, ui.tsx, icons.tsx, navigate.ts) |
| `frontend/e2e` | Playwright end-to-end specs (example.spec.ts) and support helpers (support/admin.ts) |
<!-- sdlc:auto:end structure -->

<!-- sdlc:auto:start conventions -->
- Backend routes are generated as typed stubs: every non-health endpoint returns a StubResponse (schemas.py) with status 'not_implemented' until a handler is written
- One router per architecture component is expected under backend/app/routers/ (currently empty, per the approved-architecture-not-yet-defined state)
- SQLAlchemy models inherit from the shared Base in app/database.py; tables are created via Base.metadata.create_all in the FastAPI lifespan handler, not via migrations
- Backend database connection is configured via the DATABASE_URL env var, defaulting to a local SQLite file (sqlite:///./app.db)
- CORS allowed origins are configured via the ALLOWED_ORIGINS env var (comma-separated); falls back to the Vite dev server origins (localhost:5173 / 127.0.0.1:5173) when unset
- Frontend screens currently render seeded/sample data and are not yet wired to the backend API
- Frontend API access is centralized in src/lib/api.ts: apiFetch() attaches a bearer token (session token from localStorage, falling back to VITE_API_SECRET_KEY) and targets VITE_API_BASE_URL (defaulting to http://localhost:8000)
- Backend linting/formatting enforced via ruff (ruff.toml); frontend via ESLint (eslint.config.js) and Prettier (.prettierrc)
<!-- sdlc:auto:end conventions -->

<!-- sdlc:auto:start testing -->
| Name | Command | Description |
|---|---|---|
| backend unit/integration tests | `cd backend && pytest` | Runs the pytest suite (tests/unit, tests/integration), with pythonpath=. set in pytest.ini so `app` imports correctly |
| backend lint | `cd backend && ruff check .` | Lints the backend Python code |
| backend format check | `cd backend && ruff format .` | Formats the backend Python code with ruff |
| frontend unit tests | `cd frontend && npm test` | Runs Vitest component tests under src/ using jsdom |
| frontend lint | `cd frontend && npm run lint` | Runs ESLint over the frontend app |
| frontend typecheck | `cd frontend && npm run typecheck` | Runs tsc --noEmit; not run by npm run build |
| frontend e2e | `cd frontend && npm run test:e2e` | Runs Playwright acceptance tests from e2e/; without E2E_BASE_URL it boots the backend (from backend/.venv) and builds/serves the frontend locally |
| frontend e2e browser install | `cd frontend && npm run test:e2e:install` | Installs the Chromium browser Playwright needs before running e2e tests |
<!-- sdlc:auto:end testing -->

<!-- sdlc:auto:start troubleshooting -->
| Problem | Solution |
|---|---|
| Frontend calls to the backend fail with a CORS error | Set the ALLOWED_ORIGINS env var on the backend to include the frontend's origin, or run the frontend on the default dev origin (localhost:5173) which is allowed out of the box |
| `pytest` fails with an import error for `app` when run from backend/ | Ensure you run pytest from the backend directory so pytest.ini's `pythonpath = .` applies, enabling `import app` |
| `npm run build` succeeds but deployment/typecheck fails | Run `npm run typecheck` (tsc --noEmit) separately, since `npm run build` does not typecheck |
| Playwright e2e tests fail to start the backend | Create and install backend/.venv (the config expects `backend/.venv/bin/python`) with `pip install -r requirements.txt`, since the e2e webServer command invokes that interpreter directly |
| Backend data is lost between runs or you need Postgres | Set DATABASE_URL to point at the Postgres instance started by `docker compose up -d` (service `db`, default db/user/password 'app') instead of the default SQLite file |
<!-- sdlc:auto:end troubleshooting -->
