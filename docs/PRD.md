# Product Requirements Document: Claude-code PWA

## Overview
This document outlines different approaches for implementing a Progressive Web App (PWA) that integrates the Claude-code backend. The app offers a chat-based UI for users to interact with Claude-code, similar to the Codex experience, where a GitHub repository acts as the environment for conversation about features and code.

## Goals
- Provide a browser-based chat interface to interact with Claude-code.
- Let users connect to a GitHub repository and discuss code changes via chat.
- Host the Claude-code backend in a Docker image with all required tools pre-installed.

## Assumptions
- The hosted Docker backend exposes an HTTP API for message passing and file operations.
- The PWA acts as a thin client, focusing on sending user messages to the backend and displaying replies.

## Approach 1: Lightweight PWA Client
1. **Frontend**
   - Built using HTML, CSS, and vanilla JavaScript or a minimal framework like Svelte.
   - Implements a chat window with message history and a text input field.
   - Utilizes service workers for offline caching and PWA capabilities.
2. **Backend**
   - A hosted Docker container running Claude-code with all the necessary tools.
   - The PWA sends JSON requests (e.g., {"message": "..."}) to the backend, and the backend returns responses.
3. **Repository Interaction**
   - The frontend passes GitHub repository details (owner/repo, branch) to the backend.
   - The backend handles `git clone`, conversation state, and commit operations.
4. **Pros**
   - Simple architecture with minimal dependencies on the client side.
   - Easy to deploy and maintain.
5. **Cons**
   - Limited UI features compared to richer frameworks.
   - More responsibility placed on the backend for repository state and complex interactions.

## Approach 2: Framework-based PWA (React or Vue)
1. **Frontend**
   - Uses React or Vue for a richer component-based UI.
   - Includes chat history, file browser, and commit view.
   - Service worker for offline support and push notifications.
2. **Backend**
   - Same hosted Docker container running Claude-code.
   - API endpoints for chat messages, repository checkout, and commit actions.
3. **Repository Interaction**
   - The frontend may provide a file tree view using GitHub API data.
   - More interactivity, such as diff view or commit messages, on the client side.
4. **Pros**
   - Better user experience with dynamic components.
   - Clear separation of concerns between UI and backend logic.
5. **Cons**
   - Slightly heavier client footprint.
   - Requires build tooling (Webpack/Vite) for bundling.

## Approach 3: Server-driven UI
1. **Frontend**
   - Minimal HTML and JavaScript; relies on server-rendered templates.
   - Chat messages are delivered as server-sent events or websockets.
2. **Backend**
   - Hosted Docker image also handles template rendering.
   - Claude-code integration generates both the message text and any UI hints.
3. **Repository Interaction**
   - More logic is server-side; the client becomes a dumb terminal.
4. **Pros**
   - Very thin client with low maintenance.
   - All logic centralized in the backend.
5. **Cons**
   - Less flexibility in UI/UX.
   - Server must manage sessions and rendering, increasing complexity there.

## Recommendations
- **Start with Approach 1**: A lightweight client will get the product running quickly and keep the backend-focused architecture straightforward.
- Consider moving to a framework-based approach (Approach 2) if richer UI features like file browsing become necessary.
- The server-driven approach (Approach 3) could be used for a rapid prototype but may limit future enhancements.

## Decision
We have selected **Approach&nbsp;1: Lightweight PWA Client** with the following technology stack:

- **UI**: React (with Vite) + TypeScript  
- **API**: Python 3.12 + FastAPI  

**Rationale**

1. React's mature ecosystem and our team's familiarity accelerate delivery while Vite's lightning-fast HMR improves DX.  
2. FastAPI on Python 3.12 provides first-class async support, automatic OpenAPI docs, and strong type-hints that pair well with React's TypeScript.  
3. Both stacks scale horizontally, have thriving communities, and fit our containerised Docker deployment model.

*Archived alternatives (kept for historical reference only)*: Svelte + Vite, Lit + Web Components, Node 20 + NestJS, Go 1.22 + Echo/Fiber, and the server-driven UI approach.

## Considered Alternatives
- **Server-driven UI** – centralized rendering, minimal client logic (**archived**).  
- **Svelte + Vite (UI)** – tiny bundles but lower team familiarity (**archived**).  
- **Lit + Web Components (UI)** – standards-based but verbose (**archived**).  
- **Node 20 + NestJS (API)** – structured TS backend but heavier (**archived**).  
- **Go 1.22 + Echo/Fiber (API)** – high-performance binary but smaller community (**archived**).

## Feature Roadmap for Approach&nbsp;1
The following tables outline the features to implement for the UI and the API. They are sorted by priority from proof of concept (POC) to a potential future "nice to have" set.

### UI Features
**POC**
[ ] Basic chat window and message input field.
[ ] Send user messages to the backend and display replies.
[ ] Field to specify GitHub repository URL and branch.
[ ] Minimal styling and layout.
**MVP**
[ ] Simple authentication with GitHub OAuth.
[ ] Conversation history persisted locally.
[ ] Option to set commit messages when the backend proposes changes.
[ ] Basic error notifications and loading indicators.
[ ] Service worker to cache static assets.
**Public v1** 
[ ] Repository file browser with read-only view.
[ ] Syntax-highlighted diff viewer.
[ ] Support for multiple conversations and repositories.
[ ] Responsive design for mobile.
[ ] Offline access to previous chats.

**Nice to have**
[ ] Push notifications for long-running operations.
[ ] Dark mode and theming support.
[ ] Drag-and-drop file attachments.
[ ] Integration to create GitHub pull requests.
[ ] User settings panel for preferences.


### API Features
**POC**       
[ ] Endpoint to post chat messages and return replies.
[ ] Clone repository from provided URL and branch.
[ ] Maintain simple conversation state per session.

**MVP**       
[ ] Endpoint to switch branches and commit changes.
[ ] Generate diffs between workspace and repository.
[ ] Basic token-based authentication.
[ ] Handle multiple concurrent sessions.

**Public v1** 
[ ] Multi-user support with role-based permissions.
[ ] Webhook integration for GitHub events.
[ ] Rate limiting and request logging.
[ ] File download and upload endpoints.
[ ] Metrics collection for usage monitoring.

**Nice to have**
[ ] Websocket streaming for real-time updates.
[ ] Support for multiple repositories per user.
[ ] Integration with other git hosts.
[ ] Administrative dashboard APIs.
[ ] Analytics endpoints for product insights.


## Next Steps
1. Confirm the set of features to deliver in the POC.
2. Define the API contract between frontend and backend.
3. Outline the architecture for the Docker-based Claude-code backend.
4. Build a minimal prototype to validate chat interactions and repository integration.

## Best Practices

### UI (PWA Frontend)
- **Responsive Design**: Mobile-first layout using CSS Flexbox/Grid. Support common break-points (≥320 px, ≥768 px, ≥1024 px) and fluid typography.
- **Offline Support**: Use a Service Worker (Network-First for API requests, Cache-First for static assets). Provide graceful degradation when offline (queue outbound messages, show cached history).
- **Accessibility**: Conform to WCAG 2.1 AA—semantic HTML, ARIA roles for chat log, focus management, sufficient color contrast.
- **Performance**: Code-split bundles, lazy-load non-critical components, compress assets (brotli/gzip), use HTTP/2.
- **PWA Compliance**: Web App Manifest with icons/splash, `display: standalone`, secure origin (HTTPS), update prompts via `beforeinstallprompt`.
- **UX Patterns**: Conversational UI conventions (typing indicators, time stamps), optimistic UI for quick feedback, skeleton loaders for slow network.
- **State Management**: Keep UI state in client-side store (localStorage/IndexedDB) with schema versioning for forward compatibility.

### API (Claude-code Backend)
- **RESTful Resources**: Plural nouns (`/repos`, `/sessions`, `/messages`). Use standard HTTP verbs.
- **Versioning**: Prefix all routes with `/api/v1/`; introduce `/v2/` when breaking changes occur.
- **Error Handling**: Consistent problem+json envelope `{ "type", "title", "status", "detail", "traceId" }`.
- **Security**: Enforce TLS 1.2+, OAuth2 (GitHub) / JWT bearer tokens, rate-limit per IP & token, input validation (OWASP-ESAPI).
- **Health & Metrics**: `/healthz` (liveness/readiness), `/metrics` (Prometheus). Log structured JSON to stdout.
- **Idempotency**: Support `Idempotency-Key` header for mutation endpoints (e.g., commits).
- **Documentation**: Generate OpenAPI 3 spec; publish Swagger UI `/docs`.
- **Pagination & Filtering**: For list endpoints support `limit`, `cursor`, `filter` query params.
- **Observability**: Trace requests with OpenTelemetry; propagate trace-context to Claude runtime.

---

## Detailed Feature Definitions & Acceptance Criteria
The tables below refine each item in the roadmap. *PoC items must be delivered first; later phases build upon the same contracts.*

### UI Features

#### Proof of Concept (PoC)

| Feature | Functional Description | Definition of Done | Tests | External Deps |
|---------|-----------------------|--------------------|-------|---------------|
| Basic chat window & input | Full-height panel with scrollable message list and text box (Enter = send, Shift+Enter = newline). | Messages render in order, input auto-clears, works at 320 px width. | Jest render test; Cypress send/receive flow; axe-core accessibility scan. | – |
| Send messages to backend & display replies | POST to `/api/v1/chat`; render Claude response. | Round-trip < 2 s on 3G; error toast on failure. | Mock API integration test; offline queue test. | Axios/fetch |
| Repo URL & branch field | Text field persisted per session; validated on blur. | Reject invalid URLs; stored in localStorage. | Unit validation tests. | – |
| Minimal styling & layout | Base theme using CSS variables. | Matches Figma spec; passes Lighthouse perf > 90. | Visual regression snapshots. | – |

#### Minimum Viable Product (MVP)
| Feature | Description | DoD | Tests | Deps |
|---|---|---|---|---|
| GitHub OAuth login | OAuth flow to get user token; show avatar. | Token stored securely; logout button. | Cypress social-login stub; unit token refresh. | GitHub OAuth app |
| Local conversation history | Persist messages in IndexedDB keyed by repo & branch. | History restored on reload, size capped 10 MB. | IndexedDB unit tests. | idb-keyval |
| Custom commit messages | Modal to edit default commit message before sending `/commit`. | Message length ≤72 chars. | Unit form validation; integration commit happy-path. | – |
| Error notifications & loaders | Global snackbar & spinner overlay component. | All async ops show loader; errors localized. | E2E forced error test. | – |
| Service Worker caching | Precache static assets; runtime cache GitHub avatars. | Passes `chrome://inspect` offline mode. | Workbox unit tests. | Workbox |

#### Public v1
| Feature | Description | DoD | Tests | Deps |
|---|---|---|---|---|
| Repo file browser | Tree view fetched via GitHub REST; read-only content pane. | Lazy-load folders; code syntax highlighted. | Cypress navigate tree; snapshot diff. | Prism.js |
| Diff viewer | Side-by-side or inline diff for backend patches. | Color-coded additions/deletions; copy to clipboard. | Jest diff render; accessibility contrast test. | diff2html |
| Multiple conversations/repos | Select menu listing stored sessions. | CRUD on conversations. | Unit context reducer tests. | – |
| Responsive mobile design | Touch-optimized UI, gestures to collapse sidebars. | Lighthouse PWA mobile score ≥ 95. | Browserstack device matrix. | – |
| Offline access to chats | Cache last 20 conversations in IndexedDB. | Open app offline and read chats. | Offline Cypress tests. | – |

#### Nice-to-Have
| Feature | Description | DoD | Tests | Deps |
|---|---|---|---|---|
| Push notifications | Web Push when backend sends update > 3 s. | User can opt-in/out; follow W3C spec. | Service Worker push test. | VAPID keys |
| Dark mode & themes | Toggle saving user preference. | Auto-detect prefers-color-scheme. | Visual regression night mode. | – |
| Drag-and-drop attachments | Drop file to upload via backend. | Shows upload progress; max 5 MB. | E2E upload test. | FilePond |
| GitHub PR integration | Button to open PR with commit. | PR URL returned & shown. | Mock GitHub API test. | GitHub REST |
| User settings panel | Modal with preferences (theme, font). | Settings persisted sync across devices. | Unit settings store test. | Cloud Firestore (speculative) |

### API Features

#### Proof of Concept (PoC)
| Endpoint | Method | Input | Output | DoD | Tests | Deps |
|----------|--------|-------|--------|-----|-------|------|
| /api/v1/chat | POST | `{ message, repo, branch }` | `{ reply, conversationId }` | 200 in <1 s w/ mock; 429 on rate-limit. | pytest + httpx; contract test w/ Postman. | Claude-code core |
| /api/v1/repos | POST | `{ repoUrl, branch }` | `{ repoId, status }` | Clones repo; status=cloned. | Integration test with private repo. | GitPython |
| /api/v1/sessions/:id | GET | – | `{ conversation, repo }` | Returns state or 404. | Unit model test. | Redis |

#### Minimum Viable Product (MVP)
| Endpoint | Method | Description | DoD | Tests | Deps |
|----------|--------|-------------|-----|-------|------|
| /api/v1/commit | POST | Switch branch & commit patch. Body: `{ repoId, branch, message }` | Returns commit SHA. | Idempotent; 409 on merge conflict. | Integration Git sandbox. | libgit2 |
| /api/v1/diff | GET | Query: `repoId`, `base`, `head` | Unified diff string. | 304 if unchanged. | Unit diff test. | difflib |
| /api/v1/auth/token | POST | `{ code }` GitHub OAuth exchange | `{ jwt }` | JWT signed, expires 1 h. | Security unit tests. | PyJWT |
| /api/v1/sessions | GET | List active sessions. | Pagination `limit/cursor`. | Contract test. | Redis |

#### Public v1
| Endpoint | Method | Description | DoD | Tests | Deps |
|----------|--------|-------------|-----|-------|------|
| /api/v1/users | GET | RBAC-protected list of users. | 403 for non-admin. | RBAC policy test. | Casbin |
| /api/v1/webhooks/github | POST | Handle GitHub event payloads. | Respond 202 fast. | Signature validation test. | hmac |
| /api/v1/metrics | GET | Prometheus format metrics. | Exposes default & custom metrics. | Prometheus scrape test. | OpenTelemetry |
| /api/v1/files/:id | GET | Download raw file. | Supports Range header. | Integration download test. | – |
| /api/v1/ratelimit | GET | Return limits & usage. | Matches config. | Unit test. | Redis |

#### Nice-to-Have
| Endpoint | Method | Description | DoD | Tests | Deps |
|----------|--------|-------------|-----|-------|------|
| /api/v1/ws | WS | Real-time stream channel. | <100 ms latency in LAN. | Websocket integration test. | FastAPI-WebSocket |
| /api/v1/repos/:id | PATCH | Add additional remotes. | Returns updated repo info. | Unit SCM test. | GitPython |
| /api/v2/repos | … | Multi-host git provider abstraction. | SDK for GitLab/Bitbucket. | Integration multi-host test. | python-dulwich |
| /admin/api | HTTP | Admin dashboard APIs. | Auth via RBAC token. | Security audit. | – |
| /api/v1/analytics | GET | Usage aggregation. | Cached 5 min. | Metrics unit test. | ClickHouse |

---

## Technical Approach Options

### UI Stack Options
1. **Svelte + Vite (Recommended for PoC/MVP)**
   - *Pros*: Tiny bundle (<10 kB polyfilled), component reactivity, simple store, first-class TypeScript.
   - *Cons*: Smaller ecosystem vs React, fewer enterprise-level UI libs.
   - *Scale Fit*: Good up to mid-size; can migrate to SvelteKit for SSR later.
   - *Constraints*: Require team ramp-up; Vite config for service worker.

2. **React (Vite) + TypeScript**
   - *Pros*: Huge ecosystem, mature tooling (Testing Library, MSW), familiar to most devs.
   - *Cons*: Larger runtime, more boilerplate for state/UI.
   - *Scale Fit*: Excellent long-term; CRA alternative no longer maintained—use Vite/Next.
   - *Constraints*: Need extra libs for animation (Framer), theming (MUI) etc.

3. **Lit + Vanilla Web Components**
   - *Pros*: Standards-based, framework-agnostic, minimal runtime.
   - *Cons*: Learning curve with Shadow DOM, limited SSR story.
   - *Scale Fit*: Suits PoC; might be verbose for large apps.
   - *Constraints*: Testing setup w/ @web/test-runner.

### API Stack Options
1. **Python 3.12 + FastAPI (Recommended)**
   - *Pros*: Type hints generate OpenAPI, async IO, rich ecosystem (pydantic-v2), easy concurrency with Uvicorn.
   - *Cons*: GIL limits CPU-bound tasks—offload heavy git ops to thread pool.
   - *Scale Fit*: Handles thousands of RPS; can containerize with slim image.
   - *Constraints*: Requires async-friendly git libraries.

2. **Node.js 20 + NestJS**
   - *Pros*: Structured architecture (controllers, services), TypeScript first, decorators generate Swagger.
   - *Cons*: Heavier dependency tree, requires DI learning.
   - *Scale Fit*: Proven at scale (millions RPS behind LB).
   - *Constraints*: Need worker thread pool for git operations.

3. **Go 1.22 + Echo/Fiber**
   - *Pros*: Native concurrency, single static binary, low memory.
   - *Cons*: Generics still maturing, slower iteration for quick prototypes.
   - *Scale Fit*: Best for long-term high-throughput services.
   - *Constraints*: Smaller community for git libs with full feature parity.

---

> These guidelines and definitions extend the original roadmap to provide actionable acceptance criteria and architectural guard-rails for all contributors.

## Implementation Plan

### PoC
- [ ] **Project scaffolding & Tooling (Fullstack)**
    - [ ] Initialise mono-repo with PNPM workspaces: `packages/web` (React) and `packages/api` (FastAPI).
    - [ ] UI: `npm create vite@latest web -- --template react-ts`; install React 18 & React-Router 6.
    - [ ] API: `poetry new api && poetry add fastapi uvicorn[standard] pydantic redis gitpython`.
    - [ ] Shared tooling: ESLint + Prettier (TS), `ruff` + `black` (Python), Husky pre-commit hooks.
    - [ ] CI: GitHub Action running `npm run test`, `pytest`, and linters on push & PR.
    - [ ] 🧪 Verify CI passes green on empty scaffold.
- [ ] **Basic chat flow (Fullstack)**
    - [ ] API: implement `POST /api/v1/chat` returning mocked reply.
    - [ ] UI: chat window component, message list, input with Enter / Shift-Enter behaviour.
    - [ ] Wire UI ↔ API via Axios, optimistic append pending message.
    - [ ] Error toast on non-200 responses.
    - [ ] 🧪 Jest render test; Cypress e2e send/receive happy path.
- [ ] **Repository cloning (API)**
    - [ ] Implement `POST /api/v1/repos` using GitPython to clone into `/tmp/repos/{uuid}`.
    - [ ] Validate input URL & branch; return `{ repoId, status }`.
    - [ ] Run clone inside thread executor to keep event-loop free.
    - [ ] 🧪 pytest integration cloning public & private repo mocks.
- [ ] **Repo selector UI (UI)**
    - [ ] Form fields for repo URL & branch; client-side regex validation.
    - [ ] Persist last value in `localStorage`; React context provider.
    - [ ] Disable chat until repo cloned successfully.
    - [ ] 🧪 RTL unit tests for validation.
- [ ] **Dev environment orchestration (Fullstack)**
    - [ ] `docker-compose.yml` with hot-reload containers (`web`, `api`, `redis`).
    - [ ] Makefile shortcuts: `make dev`, `make lint`, `make test`.
    - [ ] Manual smoke test: `make dev` then open `http://localhost:5173`.
- [ ] **Baseline container images & deployment (Fullstack)**
    - [ ] Multi-stage Dockerfiles: node 20-alpine for `web`; python:3.12-slim for `api`.
    - [ ] GH Actions "push-main" workflow building & pushing to GHCR.
    - [ ] 🧪 Deploy preview to Fly.io (backend) & GitHub Pages (frontend); manual QA.

### MVP
- [ ] **GitHub OAuth (Fullstack)**
    - [ ] API: `/api/v1/auth/token` exchanging GitHub code for JWT (PyJWT).
    - [ ] UI: pop-up OAuth flow, store JWT in `Authorization` header, handle refresh.
    - [ ] 🧪 Cypress login stub; pytest token expiry test.
- [ ] **Conversation persistence (UI)**
    - [ ] Use `idb-keyval` to store messages keyed `repo|branch|conversationId`.
    - [ ] Add schema versioning & migration path.
    - [ ] 🧪 IndexedDB unit test ensures history restores after reload.
- [ ] **Commit workflow (Fullstack)**
    - [ ] API `/api/v1/commit` switching branch, applying patch, committing with user-supplied message.
    - [ ] UI modal to edit default commit message (limit 72 chars, validator).
    - [ ] Progress bar & success toast with commit SHA.
    - [ ] 🧪 Integration test using temp git repo; Cypress happy path.
- [ ] **Global error & loading states (UI)**
    - [ ] Axios interceptor sets `LoadingContext`; render `SpinnerOverlay`.
    - [ ] Material UI `Snackbar` for errors; i18n messages.
    - [ ] 🧪 axe-core accessibility scan of overlay.
- [ ] **Service worker caching (UI)**
    - [ ] Add Workbox plugin to Vite build.
    - [ ] Pre-cache static assets; network-first strategy for API requests.
    - [ ] 🧪 Cypress offline mode: page loads & chat queue when offline.
- [ ] **Session store & rate limiting (API)**
    - [ ] Use Redis for per-session conversation state; enforce 30 req/min per token.
    - [ ] Expose `/api/v1/ratelimit` endpoint.
    - [ ] 🧪 pytest rate-limit exhaust test.

### Public v1
- [ ] **Repository browser & file viewer (Fullstack)**
    - [ ] API: `/api/v1/files/:id` streaming raw file; directory listing endpoint.
    - [ ] UI: tree view with lazy loading (`@tanstack/react-virtual`), code viewer via Prism.js.
    - [ ] 🧪 Cypress tree navigation; snapshot diff test.
- [ ] **Diff viewer (Fullstack)**
    - [ ] API `/api/v1/diff?repoId&base&head` returns unified diff string.
    - [ ] UI diff component based on `react-diff-viewer` with copy-to-clipboard.
    - [ ] 🧪 Jest diff render; color-contrast accessibility test.
- [ ] **Responsive mobile UX (UI)**
    - [ ] Implement drawer navigation; touch gestures to toggle sidebars.
    - [ ] CSS custom properties for dynamic sizing.
    - [ ] 🧪 Browserstack iPhone & Android smoke tests.
- [ ] **Offline conversations (Fullstack)**
    - [ ] Store last 20 conversations in IndexedDB; register background-sync.
    - [ ] Queue outbound messages when offline; replay on reconnect.
    - [ ] 🧪 Cypress offline conversation view test.
- [ ] **Observability & metrics (API)**
    - [ ] Integrate `prometheus-fastapi-instrumentator`; expose `/metrics`.
    - [ ] Propagate `traceparent` header to Claude runtime.
    - [ ] 🧪 Integration metrics scrape test.

### Nice to have
- [ ] **Push notifications (Fullstack)**
    - [ ] Backend web-push with VAPID; UI permission prompt.
    - [ ] 🧪 Service worker push unit test.
- [ ] **Dark mode & theming (UI)**
    - [ ] Theming via CSS variables & `prefers-color-scheme` detection.
    - [ ] 🧪 Visual regression dark mode.
- [ ] **Drag-and-drop attachments (Fullstack)**
    - [ ] UI integration with FilePond; API `/api/v1/files` upload route.
    - [ ] 🧪 E2E upload progress test.
- [ ] **GitHub PR integration (Fullstack)**
    - [ ] API to open PR via GitHub GraphQL API; return PR URL.
    - [ ] UI button & toast linking to PR.
    - [ ] 🧪 Mock GitHub API integration test.
- [ ] **User settings panel (UI)**
    - [ ] Modal with toggles for theme, font size & experimental flags.
    - [ ] Sync preferences to GitHub Gist for cross-device backup.
    - [ ] 🧪 RTL unit test saving settings.
