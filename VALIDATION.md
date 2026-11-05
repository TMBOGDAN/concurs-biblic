# Executed validation

Validated locally on Windows using Python 3.13, Node.js 24, PostgreSQL 18, and installed Google Chrome. These checks use actual API, browser, and PostgreSQL data. No quiz fixtures are shipped as mock application data.

| Check | Result |
| --- | --- |
| `pytest -q` against a separate `quiz_test` PostgreSQL database | 31 passed |
| `alembic upgrade head` on fresh PostgreSQL databases | Passed |
| `alembic check` against the application schema | No pending schema changes |
| `npm test` (Vitest + React Testing Library) | 7 passed |
| `npm run build` (strict TypeScript + Vite production build) | Passed |
| `npm audit` | 0 reported vulnerabilities |
| Playwright, admin + three isolated participant contexts + late participant | Passed |
| Playwright, email accounts / USER restrictions / scheduling / pause / resume / early stop | Passed; 2 browser tests total |
| Mobile screenshot inspection, 390px viewport | Waiting room, multiple-choice question and results reviewed; no horizontal overflow |
| Desktop screenshot inspection | Live control and results reviewed |
| Windows `setup.ps1` and `start-local.ps1` | Random config, isolated password-authenticated PostgreSQL, migration, bootstrap, API and Vite startup passed |
| Full local stack stop/start during a LIVE edition | PostgreSQL, backend and frontend restarted; active question, participant/admin sessions and submitted answer preserved |
| Rejoin cutoff after restart, WebSocket reauthentication and final persisted score | Passed |
| Compose YAML static parsing | Three services, persistent volume and all three health checks present |

The browser acceptance test authors five questions using the UI, including multiple-choice questions; joins three named participants; starts the edition; rejects a late entrant; submits answers; advances all questions; verifies submitted answers after refresh; goes offline and reconnects; finishes; and checks scores of 5/5, 3/5 and 3/5 with shared second place. It also checks for JavaScript page errors and mobile horizontal overflow. Temporary test-created competitions are deleted afterward.

The PostgreSQL integration tests include role checks, CSRF, forbidden origins, room authorization, future-question and foreign-option rejection, duplicate answers, normalized duplicate names, immutable LIVE questions, exact-set scoring, concurrent NEXT requests, and a join/START race.

The extension tests verify USER-only public registration, rejection of role/username injection, normalized email uniqueness, password hashing, opaque session storage, CSRF-protected logout and token revocation, persistent sessions, all administrative endpoint restrictions, current-role rechecks, secure operator promotion with session invalidation, recovery only by the same account, scheduling validation, stale pause/resume commands, blocked answering/NEXT during pause, early-stop publication, admin-only live scores, participant exclusion and catalogue WebSocket notifications. Equal correct answers with very different client-supplied response speeds and claimed points still produce exactly one point and the same rank.

The additional browser test registers through the UI, denies the USER access to `/admin` and the administrative API, updates the SCHEDULED/WAITING catalogue without refresh, pauses with a selected unsent answer, verifies disabled controls, resumes the same question, submits, restores participation after logout/login, and checks an early-stop score of 1/2. Mobile and desktop pause/control screenshots were inspected. A production-data upgrade was applied after taking a development database backup; `alembic check` reports no schema drift.

## Verification limits

Docker Engine/Desktop and the Docker CLI are not installed in this environment. Therefore `docker compose up --build`, actual container health checks, the Nginx proxy inside its image, and container restart behavior have **not been executed here**. The Compose configuration, Dockerfiles and Nginx configuration are supplied, but local PostgreSQL/process restart checks are not claimed as container checks.

The backend is designed for one worker/replica. Horizontal scaling and a 50+ participant load test were not performed. Scheduling announces a date/time and intentionally retains manual opening/START. Email verification and password-reset email delivery are outside the implemented account lifecycle. A third-party Starlette test-client deprecation notice appears with the installed httpx adapter; all integration tests pass. Vite reports two harmless source-annotation warnings from Zod while producing the build.

## Reproduce the restart check

With the local stack already running:

```powershell
.\.venv\Scripts\python.exe scripts\verify-local-restart.py
```

This restarts only the development stack recorded by the scripts in `.local/dev-processes.json`. It creates and removes its own temporary competition, and keeps session tokens in memory. Run it only on an idle development instance because other local users will temporarily disconnect.

## Reference documentation

- [FastAPI WebSockets](https://fastapi.tiangolo.com/advanced/websockets/)
- [SQLAlchemy Session API](https://docs.sqlalchemy.org/en/20/orm/session_api.html)
- [Vite setup requirements](https://vite.dev/guide/)
- [Vitest migration requirements](https://main.vitest.dev/guide/migration/)
- [PostgreSQL official image and version 18 volume layout](https://hub.docker.com/_/postgres)
