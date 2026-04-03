# Phase Structure & Task-Splitting Rules

## Phase Overview

| # | Phase Title | Included When | Typical Task Count |
|---|-------------|---------------|--------------------|
| 1 | Project Foundation | Always | 4–6 |
| 2 | Data Models & Schemas | schemas non-empty | 2–8 (group ≤5 schemas/task) |
| 3 | Authentication & Security | securitySchemes non-empty | 2–3 |
| 4 | Core API — [Tag] | One per tag group | 1–3 per tag |
| 5 | Middleware & Utilities | Always | 3–5 |
| 6 | Testing | Always | 2–5 |
| 7 | Documentation & Deployment | Always | 2 |

Renumber sequentially if phases are skipped (e.g. no auth → Phase 3 becomes "Phase 3: Core API — Users").

---

## Phase 1: Project Foundation

**Purpose**: Get a working, runnable skeleton that every subsequent step builds on.

**Tasks (one per concern):**
1. **Repository & dependency init** — `git init`, package manifest (package.json / requirements.txt / go.mod / pom.xml), install core framework and runtime dependencies, `.gitignore`
2. **Folder structure** — Create all top-level directories: `/src`, `/routes` (or `/controllers`), `/models`, `/middleware`, `/services`, `/tests`, `/config` — match the framework's convention
3. **Base server setup** — Entry point file, framework instance, health-check route (`GET /health`), server start with port from env
4. **Environment & configuration** — `.env.example` with all required vars (PORT, DB_URL, JWT_SECRET, etc. inferred from swagger servers/security), config loader module
5. **Database connection** (only if swagger schemas suggest a DB) — connection module, connection test, graceful shutdown

**Rule:** Each task is independently executable (no task assumes another is done within Phase 1, except task 3 assumes task 1's deps are installed).

---

## Phase 2: Data Models & Schemas

**Purpose**: Every `components.schemas` entry becomes a database model, ORM definition, or validation schema.

**Task-splitting rules:**
- Group related schemas together: e.g. `User` + `UserProfile` + `UserPreferences` = one task
- Maximum 5 schemas per task
- Keep request/response DTOs with their parent entity (e.g. `CreateUserInput` and `UpdateUserInput` go with the `User` task)
- Order: core entities first, then join/relational models, then pure DTOs

**Each task includes:**
- Full schema property list with types, formats, and required fields (copied verbatim from swagger)
- ORM model definition (Mongoose schema, SQLAlchemy model, GORM struct, Sequelize model, etc.)
- Input validation rules derived from OpenAPI constraints (`minLength`, `pattern`, `minimum`, `enum`, etc.)
- Export/import statement

---

## Phase 3: Authentication & Security

**Purpose**: All auth infrastructure before any protected routes are built.

**Tasks (one per mechanism):**
1. **JWT setup** (if bearerAuth) — JWT sign/verify utility, secret from env, token expiry config
2. **Auth routes** — `/register`, `/login`, `/refresh`, `/logout` endpoints with request/response shapes from swagger
3. **Auth middleware / guards** — Middleware function that validates Bearer token, attaches user to request context, 401/403 responses

**If apiKey scheme:** One task for API key validation middleware (header or query param).
**If OAuth2:** One task for OAuth2 flow setup and callback routes.

---

## Phase 4: Core API — [Tag]

**Purpose**: Implement all endpoints for each tag group, one tag at a time.

**Task-splitting rules:**
- One task per tag group by default
- Split a tag into multiple tasks if it has **more than 8 endpoints**:
  - Task A: CRUD operations (GET list, GET by ID, POST, PUT/PATCH, DELETE)
  - Task B: Specialized operations (search, bulk, nested resources, actions)
- Always specify the full endpoint contract: HTTP method, path, path params, query params, request body schema (with all fields), response schema (with all fields), auth requirement, status codes

**Each task includes:**
- Route file creation (e.g. `routes/users.js` or `controllers/UserController.ts`)
- Service/handler functions for each endpoint
- Input validation wired to Phase 2 schemas
- Auth guard applied where swagger marks the endpoint as secured
- All response shapes from swagger `responses` definitions

---

## Phase 5: Middleware & Utilities

**Purpose**: Cross-cutting concerns applied globally or across many routes.

**Tasks (one per concern):**
1. **Error handling** — Global error handler, error response format (`{ error, message, statusCode }`), 404 handler
2. **Request validation** — Body/query/param validation middleware using Phase 2 schemas or a validation library
3. **Logging** — Request/response logger, error logger, log format
4. **CORS & security headers** — CORS config (origins from env), helmet/security headers if applicable
5. **Rate limiting** (only if swagger or server config hints at it) — Rate limiter middleware, config from env

---

## Phase 6: Testing

**Purpose**: Verify the build is correct before documentation and deployment.

**Tasks:**
1. **Test setup & fixtures** — Test framework config (Jest/pytest/go test), test DB setup, seed data fixtures, helper utilities
2. **Model/schema unit tests** — Validation tests for each schema (required fields, type checks, constraint enforcement)
3. **Auth integration tests** — Register, login, token refresh, protected route access/rejection
4. **API integration tests per tag** — One task per tag group: test each endpoint (happy path + error cases)

---

## Phase 7: Documentation & Deployment

**Purpose**: Make the project runnable and shareable by anyone.

**Tasks:**
1. **Documentation** — Complete README (project name, description, setup steps, env vars table, API overview, example requests), `.env.example` finalized
2. **Deployment config** — `Dockerfile` (multi-stage if applicable), `docker-compose.yml` for local dev with DB, or platform config (Railway/Heroku `Procfile`, Vercel `vercel.json`) — choose based on inferred stack

---

## Priority Mapping

| Phase | Priority |
|-------|----------|
| 1–3 | high |
| 4–5 | medium |
| 6–7 | low |
