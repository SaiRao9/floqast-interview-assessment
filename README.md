# Fintech SDET Assessment

TypeScript + Playwright automation for a hypothetical fintech platform (User Service, Transaction Service, Notification Service, API Gateway).

There is no real backend in this repository. A **local mock API + mock UI** simulate the supplied contract so the suite is fully runnable after `npm install`.

## 1. Project overview

This project is an assessment-ready test framework that demonstrates:

- API automation through a reusable client (not raw `fetch` in every spec)
- UI automation with Page Object Model
- Authentication handling, schema assertions, and test data factories
- HTML / JSON / JUnit reporting
- Environment-based configuration so the mock can later be swapped for real services

## 2. Architecture

```
Playwright tests
  ├── API specs  →  ApiClient  →  API_BASE_URL  →  mock Express API (in-memory)
  └── UI specs   →  Page Objects → UI_BASE_URL  →  mock HTML  →  same API
```

The mock server (Express, in-memory maps) is **not** MongoDB, Redis, or Docker. It exists so tests can run offline. Point `API_BASE_URL` at a real gateway when those services exist; the specs and `ApiClient` stay the same.

## 3. Technology choices

| Choice | Why |
| --- | --- |
| TypeScript (strict) | Safer refactors, clearer models |
| Playwright Test | One runner for API + UI, first-class HTML report |
| Express mock | Lightweight, no external infrastructure |
| dotenv | Local/CI configuration without hardcoded URLs |
| Factories + fixtures | Isolated tests, unique emails, no shared mutable fixtures |

## 4. Folder structure

```
config/                 Environment helpers and named profiles
docs/                   Test strategy and k6 sample
factories/              User and transaction builders
fixtures/               Playwright fixtures (apiClient, createdUser, pages)
mock/                   Express API + static UI
  handlers.ts           Validation and HTTP status behaviour
  public/               Register and transaction pages
pages/                  Page Object Model
tests/api/              API specs
tests/ui/               UI specs
types/                  Shared request/response models
utils/                  ApiClient, logger, custom assertions
playwright.config.ts    Projects, reporters, webServer
.github/workflows/      CI pipeline
```

## 5. How to install

```bash
npm install
npx playwright install chromium
cp .env.example .env   # optional; defaults match the local mock
```

Node.js 18+ is required (Node 22/24 recommended).

## 6. How to run tests

The mock server is started automatically by Playwright `webServer`.

```bash
npm test
```

Equivalent: `npx playwright test`.

To start the mock yourself (UI available at http://localhost:3000):

```bash
npm run start:mock
```

## 7. How to run API tests only

```bash
npm run test:api
```

## 8. How to run UI tests only

```bash
npm run test:ui
```

## 9. How to run headed mode

```bash
npm run test:headed
```

UI tests open Chromium. API tests still run headless (no browser).

## 10. How to view the HTML report

```bash
npm test
npm run report
```

Reports are written to `playwright-report/` even when the run is in CI (`open: 'never'`).

JSON: `test-results/results.json`  
JUnit: `test-results/junit.xml`  
Screenshots / traces / video: `test-results/` on failure (trace on first retry).

## 11. Environment configuration

| Variable | Local default | Purpose |
| --- | --- | --- |
| `ENVIRONMENT` | `local` | Profile name (`local` / `test` are runnable) |
| `BASE_URL` | `http://localhost:3000` | Playwright base URL |
| `UI_BASE_URL` | `http://localhost:3000` | Mock frontend |
| `API_BASE_URL` | `http://localhost:3000` | Mock API origin (no `/api` suffix) |
| `AUTH_TOKEN` | `valid-test-token` | Default bearer token |
| `PORT` | `3000` | Mock server port |

`config/environments.ts` includes **placeholders** for development and staging. Those URLs are not live; do not expect tests to pass against them.

## 12. Test coverage

### API (Playwright)

- **Users**: create success, required fields, invalid email, invalid accountType, duplicate email, missing body, get by id, not found, invalid id
- **Transactions**: create success, missing userId, invalid/zero/negative amount, invalid type, missing recipientId, list, empty list, unknown user
- **Auth**: missing token, invalid token, unauthorized GET user / POST transaction / GET transactions, user-role allowed, admin-only reset forbidden

### UI (Playwright + POM)

- Registration success, required fields, invalid email
- Transaction success, invalid amount, required fields
- API errors rendered: duplicate email, unknown user

### CRUD gap

The product contract supplied for this assessment is **Create + Read only**:

- `POST /api/users`, `GET /api/users/:id`
- `POST /api/transactions`, `GET /api/transactions/:userId`

There are **no** production PUT, PATCH, or DELETE routes. The framework does not invent them. `ApiClient` still implements `put` / `patch` / `delete` so Update/Delete specs can be added when the APIs exist. See `docs/test-strategy.md`.

## 13. Authentication strategy

Every `/api/*` call requires:

```
Authorization: Bearer valid-test-token
```

Missing or unknown tokens return **401**. Optional roles:

- `user-token` — product APIs
- `admin-token` / `valid-test-token` — product APIs + `POST /api/test/reset`

The client attaches the token by default. Tests pass `{ token: null }` or `{ token: tokens.invalid }` for negative cases. No credentials are hardcoded in specs.

## 14. Test data strategy

`buildUser()` always generates a unique email (`qa.user.<timestamp>.<rand>@example.com`). Tests create the records they need. The `createdUser` fixture is used when a spec needs an existing user. Isolation does not depend on test order.

## 15. Reporting strategy

Priority:

1. Playwright HTML report (required)
2. Screenshots on UI failure (required)
3. API request/response logging on stdout (required)
4. JSON report
5. JUnit report (CI)

## 16. Assumptions

- Account types are `basic`, `standard`, `premium`
- Transaction types are `transfer`, `deposit`, `withdrawal`
- Amounts are JSON numbers strictly greater than 0
- User and transaction IDs are UUIDs
- Sender and recipient must already exist to create a transaction
- Successful mock transactions are stored with `status: completed`
- The Notification service is out of scope (no API was provided)

## 17. Limitations

- Mock storage is in-memory and single-process
- Auth is a static bearer token, not JWT/OAuth
- The UI is a thin stand-in, not a production frontend
- No Update/Delete product APIs
- Performance testing is documented (k6 sample) but not wired into `npm test`
- Parallel workers share one mock process; uniqueness prevents most collisions

## 18. Future improvements

- Point the same specs at a deployed gateway
- Add Update/Delete tests when those routes ship
- Contract tests (OpenAPI / JSON Schema) generated from the real spec
- Wire mock notifications (or a real Notification service)
- k6 in a separate CI job against staging
- Visual and accessibility checks on the real UI

## Scripts

| Script | Command |
| --- | --- |
| `npm test` | All Playwright projects |
| `npm run test:api` | API project |
| `npm run test:ui` | UI project |
| `npm run test:headed` | Headed run |
| `npm run test:debug` | Playwright inspector |
| `npm run report` | Open HTML report |
| `npm run lint` | `tsc --noEmit` |
| `npm run start:mock` | Serve mock API + UI |

## Type checking

```bash
npm run lint
```
