# Development plan: **Guest Link** (MVP — Welcome pages + Automated WhatsApp welcome messages + Basic real-time messaging)

Great — thanks for the answers. I have everything I need and I generated a step-by-step plan you can follow with an “AI as pair programmer” (Vibe coding) workflow: *AI generates new files from spec → you review → commit*. Below you’ll find milestones, tasks, acceptance criteria (with automated test guidance), CI/CD + infra recipe, repository scaffold, Prisma schema pointers, AI prompts & PR templates tuned for SWE-1 / code-supernova style use, a WhatsApp provisioning checklist, and example one-pass scaffolds you can feed to the AI to start coding immediately.

Where useful I reference your uploaded specs and architecture docs so the plan follows your stated requirements. See the Requirements Specification (REQs) and Architecture Design Doc for details.  &#x20;

---

# 1 — Big picture & assumptions (MVP scope)

* **MVP features (confirmed):**

  * Personalized welcome pages (host creates content; guests access via unique URL).&#x20;
  * Automated WhatsApp welcome messages (template includes welcome page link).&#x20;
  * Basic real-time messaging (guest ↔ host via WhatsApp + in-app basic UI to view messages).&#x20;
* **Out of scope for MVP:** staff task queue, guest DB & analytics (saved for Phase 2).&#x20;
* **Tech choices (you chose / I recommend):**

  * Frontend: **Next.js 14+ (App router)** + **TypeScript** + **Tailwind CSS** → host & welcome pages.&#x20;
  * Backend: **Node.js** microservice(s) (Express or Fastify) for WhatsApp webhook & API.
  * DB: **PostgreSQL** (via Prisma). (Prisma schema snippets in Architecture Doc).&#x20;
  * Real-time: **Socket.IO** (self-hosted) or a managed pub/sub like Pusher / Ably (optional). Architecture doc expects WebSocket.&#x20;
  * Hosting: **Vercel** for frontend; **AWS** (ECR + ECS/Fargate or AWS App Runner) for backend; **S3** for media.&#x20;
* **AI pair programmer flow:** “Generate files from spec → human review → commit” (we’ll provide exact prompts for that). Use SWE-1 for succinct project-context edits and code-supernova for very long-context tasks / multi-file refactors.&#x20;

---

# 2 — High-level milestones (sequence + why)

1. **Milestone 0 — Heavy Upfront Setup (reduce later iterations)**
   Goal: prepare repo, infra, conventions, and baseline tests so AI can generate code predictably.
2. **Milestone 1 — Welcome Pages (content + rendering + shareable URL)**
   Goal: hosts can create/edit a page; unique URL is generated; guest can open without login.
3. **Milestone 2 — WhatsApp Integration (sending automated welcome message + webhook handlers)**
   Goal: system sends a templated welcome message (with link) after stay creation (or scheduled).
4. **Milestone 3 — Real-time messaging (basic)**
   Goal: messages delivered/updated in near-real time in host UI (and reflect WhatsApp inbound via webhooks).
5. **Milestone 4 — End-to-end tests, CI/CD, deploy to Vercel (frontend) + AWS (backend)**
   Goal: automated test suite + continuous deploy flows.
6. **Milestone 5 — Launch checklist & monitoring**
   Goal: smoke tests, logs/alerts, basic usage monitoring.

Below I break each milestone down into tasks, acceptance criteria, and AI prompts you can feed the pair-programmer.

---

# 3 — Milestone 0: Heavy Upfront Setup (essential, do first)

Why: You asked for a heavier upfront deliverable to reduce iterations. This milestone sets conventions and infra so the AI’s generated files slot in cleanly.

## Tasks

1. **Create GitHub repo + branch policy**

   * Repo: `guest-link` (main branch protected; PR required).
   * Add CODEOWNERS (you and AI reviewer), .gitignore, LICENSE (MIT or your choice).

2. **Pick & scaffold Next.js template**

   * Use a Vercel Next.js App Router template with TypeScript + Tailwind + PWA support (AI can choose exact template from Vercel). Command for AI to run to scaffold:

   ```bash
   npx create-next-app@latest guest-link --experimental-app --typescript --eslint --src-dir
   ```

   (AI will choose a Vercel starter template that's closest to PWA & Tailwind.)

3. **Create backend microservice skeleton** (`/services/whatsapp`) with Node.js + TypeScript + Express or Fastify. Add Dockerfile.

4. **Prisma + Postgres initial schema** — include core models for `Property`, `Stay`, `Guest` and `WelcomePage` (we’ll adapt from architecture doc). Use `prisma` client. (Prisma schema sample below; based on your Architecture Doc.)&#x20;

5. **Testing infra** — Jest + React Testing Library for frontend; Vitest is also an option. For backend use Jest + Supertest. Configure Playwright for E2E.

6. **CI workflows (basic)** — GitHub Actions that run linters, unit tests, and build jobs on PR. (Detailed YAML later.)

7. **Environment & secrets management** — create `.env.example` keys (DATABASE\_URL, WHATSAPP\_API\_TOKEN, WHATSAPP\_PHONE\_ID, AWS\_\*).

## Deliverables

* Working dev environment: `yarn dev` runs frontend; `yarn start:dev` runs backend; `pnpm` or your preferred package manager is fine.
* Prisma migrations created and run to local dev DB.

## Acceptance criteria + automated checks

* `yarn test` runs and all baseline tests (smoke) pass (example: health endpoint & homepage render test).
* `GET /api/health` returns `200 OK` for backend (unit/integration test).
* Frontend root (`/`) renders expected title (unit test).
* CI runs these tests on PRs.

## AI prompts (first-use)

Use SWE-1 or GPT fallback for short tasks; use code-supernova if you want the AI to examine the full architecture doc and propose the exact template.

**Scaffold repo prompt (for SWE-1):**

```
You are my AI pair programmer. Create a new Next.js 14 TypeScript app skeleton with Tailwind and PWA support. Use the App Router structure, add ESLint, TypeScript, and a /services/whatsapp Node.js TypeScript microservice with Express. Add a Dockerfile for the backend, a basic GitHub Actions workflow that runs lint & tests. Output: list of files created and a short summary of commands I should run locally to bootstrap (install, prisma migrate, start). Keep files minimal and testable.
```

---

# 4 — Milestone 1: Welcome Pages (host UI + publishable shareable URL)

This is the most product-visible part of the MVP.

## Key REQs (from your Requirements Spec)

* Host can create a branded welcome page for each stay; templating; preview; unique secure URL; mobile-optimized; no login required for guest.&#x20;

## Tasks (sequential)

1. **DB & Prisma model** (`WelcomePage`, `Stay`, `Property`) — adapt from architecture doc.&#x20;
   Example Prisma model (adapted):

   ```prisma
   model Property {
     id        String @id @default(uuid())
     name      String
     createdAt DateTime @default(now())
     stays     Stay[]
   }

   model Stay {
     id             String @id @default(uuid())
     propertyId     String
     welcomePageUrl String @unique
     startDate      DateTime
     endDate        DateTime
     createdAt      DateTime @default(now())
     welcomePage    WelcomePage?
   }

   model WelcomePage {
     id        String @id @default(uuid())
     stayId    String @unique
     title     String
     body      String   @db.Text
     heroImage String?
     published Boolean @default(false)
     createdAt DateTime @default(now())
   }
   ```

   (source: adapted from Architecture Doc schema).&#x20;

2. **API endpoints (backend)** — CRUD for welcome pages:

   * `POST /api/stays` (create stay + welcomePage url generation)
   * `GET /api/welcome/:welcomeId` (returns content JSON for rendering & SSR)
   * `PUT /api/welcome/:welcomeId` (edit)
   * `POST /api/welcome/:welcomeId/publish` (publish toggle)

   Add validation, simple rate limiting and signed unique URL token generation.

3. **Frontend pages** (Next.js App Router):

   * Host admin UI: `app/dashboard/welcome/[stayId]/edit/page.tsx` (simple WYSIWYG or markdown + image upload).
   * Public welcome page: `app/welcome/[welcomeId]/page.tsx` — server-rendered, mobile optimized, offline caching enabled (Service Worker).

4. **Preview & offline** — implement a preview route that renders draft content and add simple PWA caching via `next-pwa` or basic Service Worker.

## Acceptance criteria + automated tests

* **API tests** (backend):

  * Unit+integration: creating a stay returns a `welcomePageUrl` (unique) and persisted in DB (Jest + Supertest).
  * Test: POST `/api/stays` → 201 and DB has Stay with `welcomePageUrl`.
* **Frontend tests**:

  * Component unit test: `WelcomePage` renders title, image, and body (react-testing-library).
  * SSR integration E2E: Playwright test visits `/welcome/{welcomeId}` and asserts content visible.
* **Security**: the public URL should be unguessable (UUID or cryptographic token) and test should confirm URL length/randomness.

## AI prompts (example)

**Generate Prisma models & migration (SWE-1):**

```
Using the project Prisma setup, generate Prisma schema models for Property, Stay, and WelcomePage matching the following requirements: host creates a WelcomePage per Stay, the WelcomePage stores title, body (rich text or markdown), optional hero image, published flag, and unique welcomePageUrl. Add relations and indexes. Output: prisma/schema.prisma file content and a suggested migration command.
```

**Generate `GET /api/welcome/:welcomeId` (code-supernova for context):**

```
I have a Next.js frontend and an Express backend using Prisma. Create a backend endpoint GET /api/welcome/:welcomeId that:
- Validates welcomeId
- Loads welcome page and stay info
- Returns JSON with title, body (sanitized), heroImage url, meta tags
- Sets cache TTL headers for 5 minutes
- Proper error handling (404/500)
Include Jest+Supertest unit tests that stub Prisma.
```

---

# 5 — Milestone 2: WhatsApp Integration (send automated welcome messages)

This milestone wires the WhatsApp Business API with your backend.

## Principal REQs

* Host sends automated welcome messages, includes welcome page link, delivery confirmation, scheduled sending.&#x20;

## Tasks

1. **WhatsApp service module (backend)**

   * A small service wrapper to call WhatsApp Business API (or Meta Cloud API). Abstract HTTP client, retries & exponential backoff, logging.
   * Functions: `sendTextMessage(phone, body)`, `sendTemplateMessage(phone, templateName, variables)`

2. **Webhook endpoint** to receive inbound messages & delivery receipts: `POST /api/webhooks/whatsapp`

   * Verify signature, parse events, store events (or publish to socket).

3. **Welcome scheduler / trigger**

   * When a stay is created (or when host hits publish), call `sendTemplateMessage` with the welcome page URL. Provide scheduling via a simple job queue (Bull / Redis) — for MVP a cron or immediate send is fine.

4. **Local dev simulator**

   * Because you may not have Live WhatsApp credentials yet, create a **simulator adapter** to test sending/receiving in dev. The adapter returns success/failure and simulates a delivery receipt.

## Acceptance criteria + automated tests

* **Unit tests** for `sendTextMessage` with HTTP client mocked.
* **Integration tests** for webhook endpoint that verifies receipt parsing and publishes to socket (use Supertest).
* **E2E** (Playwright or Postman): simulate creating a stay → app calls send function (mocked) → verify that an event is queued / recorded as “sent”.

## AI prompts (example)

**Create WhatsApp service wrapper (SWE-1):**

```
Create a TypeScript module `services/whatsapp.ts` exposing sendTextMessage(phone: string, text: string) and sendTemplateMessage(phone: string, templateName: string, variables: Record<string,string>). Implement retry logic (3 tries with exponential backoff) and structured logging. Add unit tests that mock HTTP calls.
```

**Create webhook handler (code-supernova):**

```
Create an Express POST /api/webhooks/whatsapp handler that:
- Verifies signature header
- Parses message events and delivery receipts
- Saves events to DB (or publishes to socket)
- Returns 200 quickly to WhatsApp
Add tests that simulate a delivery receipt and an inbound user message.
```

---

# 6 — Milestone 3: Basic Real-time Messaging (guest ↔ host)

Goal: reflect inbound WhatsApp messages in a simple host UI (no full inbox required).

## Tasks

1. **Socket server** (could be part of backend service) — Socket.IO or WebSocket endpoint that hosts can connect to; emits events on new inbound messages (from webhook).
2. **Host mini UI** on Next.js: `app/dashboard/messages/[stayId]/page.tsx` — list messages in realtime, display send option (for manual responses via WhatsApp API).
3. **Message flow**: inbound webhook → backend validates → store message → emit socket event → host UI receives and shows message.
4. **Send from host UI**: host types → call backend API `/api/whatsapp/send` which calls WhatsApp API so the guest receives the message.

## Acceptance criteria + tests

* **Integration test**: simulate webhook inbound → message stored → socket event emitted (test by connecting to socket in test harness).
* **Frontend unit test**: `MessagesList` component renders recent messages and shows live updates when socket emits.
* **Manual E2E**: With the simulator adapter, host UI shows inbound message within <1 second.

## AI prompts (example)

**Implement socket emission for inbound messages (SWE-1):**

```
Add Socket.IO integration to the WhatsApp backend service: on receiving a message via webhook, save to DB and emit `message:new` to room `stay-{stayId}` with message payload. Add Jest tests that mock socket server and verify emission.
```

---

# 7 — Milestone 4: Tests, CI/CD, Deploy

Make sure everything is reproducible and deployable.

## CI/CD recipe

* **Frontend (Vercel):**

  * Connect GitHub repo to Vercel; configure environment variables in Vercel dashboard (e.g., NEXT\_PUBLIC\_API\_URL).
  * Vercel will auto-deploy on pushes to `main` by default.
* **Backend (AWS):**

  * Build Docker image in GH Actions → push to ECR → deploy to ECS Fargate / App Runner.
  * Use GitHub Actions secrets for AWS credentials.
  * Health checks: `/api/health` endpoint for load balancer.

## Example GitHub Actions workflows

* `ci.yml` (run on PRs): lint, unit tests (frontend & backend), typecheck, build.
* `deploy-backend.yml` (on merge to main): build & push Docker to ECR, update ECS service (or App Runner).

*(I can provide full YAML templates on request; below is a minimal example for CI)*

```yaml
# .github/workflows/ci.yml
name: CI
on: [pull_request]
jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      - name: Install
        run: pnpm install
      - name: Lint
        run: pnpm lint
      - name: Run tests
        run: pnpm test -- --runInBand
```

## Monitoring & alerts

* CloudWatch for backend logs + custom dashboards.
* Sentry for exceptions (optional).
* Basic uptime monitoring (Pingdom/UptimeRobot).

## Acceptance criteria

* CI runs on PRs and blocks merges on failed tests.
* Auto deploys succeed to staging and production with health checks passing.

---

# 8 — Testing strategy (detailed)

You asked each task to include automated tests. Here’s the recommended testing pyramid:

* **Unit tests** (Jest / Vitest) — components, utilities, service wrappers.
* **Integration tests** (Jest + Supertest) — APIs, DB interactions (use a test DB / SQLite in memory or ephemeral Postgres).
* **E2E tests** (Playwright) — user flows: open welcome page, create stay (API), verify WhatsApp send simulated, host UI sees realtime message.

### Example test matrix by feature

* **Welcome page**

  * Unit: component renders title + sanitized HTML.
  * Integration: `GET /api/welcome/:id` returns JSON.
  * E2E: visit `/welcome/:id`, assert header/body present.
* **WhatsApp send**

  * Unit: `sendTextMessage` retries on failure.
  * Integration: creating a stay triggers a job to call `sendTemplateMessage` (mocked).
* **Realtime**

  * Integration: webhook → DB → socket emission.

---

# 9 — Repo scaffold (folder layout + sample files)

```
guest-link/
├─ apps/
│  └─ web/                 # Next.js app (frontend)
│     ├─ app/
│     │  ├─ welcome/[id]/page.tsx
│     │  └─ dashboard/messages/[stayId]/page.tsx
│     ├─ components/
│     └─ tests/
├─ services/
│  └─ whatsapp/            # backend microservice
│     ├─ src/
│     │  ├─ routes/webhooks.ts
│     │  ├─ controllers/whatsapp.ts
│     │  ├─ services/whatsapp.ts
│     │  └─ sockets/
│     └─ Dockerfile
├─ prisma/
│  └─ schema.prisma
├─ .github/workflows/ci.yml
├─ package.json
└─ README.md
```

### Example `package.json` scripts

```json
{
  "scripts": {
    "dev:web": "cd apps/web && pnpm dev",
    "dev:whatsapp": "cd services/whatsapp && ts-node-dev --respawn src/index.ts",
    "test": "pnpm -w -r test",
    "lint": "pnpm -w -r lint"
  }
}
```

### Prisma snippet (adapted from your Architecture Doc)

(Place in `prisma/schema.prisma`) — adapted from the Architecture Design Document.&#x20;

```prisma
model Property {
  id        String   @id @default(uuid())
  name      String
  stays     Stay[]
  createdAt DateTime @default(now())
}

model Stay {
  id             String   @id @default(uuid())
  propertyId     String
  welcomePageUrl String   @unique
  startDate      DateTime
  endDate        DateTime
  welcomePage    WelcomePage?
  createdAt      DateTime @default(now())
}

model WelcomePage {
  id        String   @id @default(uuid())
  stayId    String   @unique
  title     String
  body      String   @db.Text
  heroImage String?
  published Boolean  @default(false)
  createdAt DateTime @default(now())
}
```

---

# 10 — AI pair-programmer artifacts (prompts, PR templates, review checklist)

## A. Prompt templates (copy/paste and reuse)

**1) Scaffolding / create file (SWE-1 short context)**

```
Task: Create file at {path} for project repo 'guest-link'. Context: use Next.js App Router, TypeScript, Prisma, Node/Express backend. Purpose: {one-sentence description}. Requirements: {bullet points}.
Constraints: keep file < 250 lines, include types, add minimal test file. Output: full file contents only, no commentary.
```

**2) Multi-file change / refactor (code-supernova — long context)**

```
You are an AI engineer with the full project context (see architecture doc). Implement the following multi-file change:
- Add API endpoint POST /api/stays which creates a Stay, generates an unguessable welcomePageUrl, creates a WelcomePage draft, and enqueues an immediate 'send welcome message' job to the WhatsApp service.
Files to update: prisma/schema.prisma, services/whatsapp/src/controllers/stays.ts, services/whatsapp/src/services/jobQueue.ts
Requirements: include error handling, logging, tests (Jest + Supertest), and update README with new env variables. Output: a patch containing new/updated files, test files, and the commands to run migrations and tests locally.
```

**3) Generate tests for a feature**

```
Generate unit and integration tests for the endpoint GET /api/welcome/:id. Use Jest and Supertest. The tests should spin up an Express server in test mode, use an in-memory SQLite or mocked Prisma client, and assert 200/404 cases. Provide test file and mock setup.
```

## B. PR description template

```
Title: [FEATURE] Implement Welcome Page creation + publish endpoint

Summary:
- Adds Prisma models: Stay, WelcomePage
- Adds POST /api/stays and GET /api/welcome/:id
- Adds unit & integration tests (Jest + Supertest)
- Adds frontend stub for /welcome/[id]

Why:
Implements MVP welcome page flow: host creates stay -> guest receives link.

Testing:
- `pnpm test` passes
- Manually: create stay via POST /api/stays → visit /welcome/{url}.

Notes:
- Env changes: DATABASE_URL, NEXT_PUBLIC_API_URL
- Reviewers: @you
```

## C. Commit messages (conventional)

* `feat(welcome): add WelcomePage model + /api/welcome/:id endpoint`
* `test(welcome): add GET /api/welcome integration tests`
* `chore(ci): add GH Actions CI workflow`

## D. Code review checklist for PRs (AI + human)

* Does code meet the acceptance criteria?
* Are new functions typed and tested?
* Are secrets only referenced via env variables?
* Are edge cases & input validation covered?
* Are logging & error messages descriptive?
* Are docs/README updated with new env & commands?

---

# 11 — WhatsApp provisioning checklist (practical steps)

You asked to include WhatsApp steps — here’s a short checklist to get a phone number and API access:

1. **Create Meta Business Manager & verify your business** (business verification may be required).
2. **Register a WhatsApp Business Account** and add a phone number you control.
3. **Obtain API credentials** (access token / app id / phone id) via Meta Cloud/WhatsApp Business API.
4. **Add webhook endpoint** in WhatsApp settings (point to `https://<your-backend>/api/webhooks/whatsapp`). Ensure HTTPS with valid cert.
5. **Create message templates** in Meta dashboard for “welcome” templates if required by the API for templated outbound messages.
6. **Test sending via sandbox / simulator first** (or use Twilio’s WhatsApp sandbox if you prefer). Implement simulator adapter in code.
7. **Enable delivery receipts & logging** and ensure you log event IDs for troubleshooting.

> Note: WhatsApp Business API policies and steps can change — check Meta’s docs while provisioning.

---

# 12 — Security & privacy (baseline)

* Use HTTPS everywhere; enforce HSTS for production.
* Store secrets in Vercel & AWS Secrets Manager / Parameter Store; do not commit `.env`.
* Ensure public welcome URLs are unguessable (UUID4 or cryptographic token). Add optional expiry.&#x20;
* Rate limit APIs & webhook endpoints.
* Sanitize user HTML/markdown (use a safe renderer) for welcome pages.
* Add basic auth or access restrictions for host dashboard endpoints (later you can add Auth provider).
* Logging: avoid logging PII; store minimal message metadata.

---

# 13 — Suggested immediate next steps (first 5 action items you can run with AI)

1. **Scaffold repo** (Next.js + Tailwind + backend microservice skeleton). Use the scaffold prompt above.
2. **Add Prisma models & migration** (use the Prisma snippet from Architecture Doc).&#x20;
3. **Implement backend health check + GET /api/welcome/\:id endpoint** (AI generates code + tests). Verify with unit/integration tests.
4. **Implement `POST /api/stays` to generate welcomePageUrl** and persist objects, with tests.
5. **Create simple frontend `app/welcome/[id]/page.tsx` that SSR fetches the API** and displays sample content; add Playwright E2E to visit the page.

Once these are green in CI, proceed with WhatsApp integration & real-time.

---

# 14 — Appendix: Example code snippets (one-pass scaffolds)

### Example Express webhook skeleton (backend)

```ts
// services/whatsapp/src/index.ts
import express from 'express';
import bodyParser from 'body-parser';
import { handleWhatsappWebhook } from './routes/webhooks';
const app = express();
app.use(bodyParser.json());
app.post('/api/webhooks/whatsapp', handleWhatsappWebhook);
app.get('/api/health', (_, res) => res.json({ ok: true }));
app.listen(process.env.PORT || 3001);
```

### Example Next.js welcome page skeleton

```tsx
// apps/web/app/welcome/[id]/page.tsx
import React from 'react';

type Props = { params: { id: string } };

export default async function WelcomePage({ params }: Props) {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/welcome/${params.id}`, { cache: 'no-store' });
  if (!res.ok) return <div>Not found</div>;
  const data = await res.json();
  return (
    <main className="max-w-3xl mx-auto p-4">
      <h1 className="text-3xl font-bold">{data.title}</h1>
      {data.heroImage && <img src={data.heroImage} alt="" />}
      <div dangerouslySetInnerHTML={{ __html: data.body }} />
    </main>
  );
}
```

### Example unit test (Jest + Supertest) for GET /api/welcome/\:id

```ts
// services/whatsapp/tests/welcome.test.ts
import request from 'supertest';
import { app } from '../src/index'; // express app
describe('GET /api/welcome/:id', () => {
  it('returns 200 and JSON for an existing welcome id', async () => {
    const res = await request(app).get('/api/welcome/test-id');
    expect(res.status).toBe(200);
    expect(res.body).toHaveProperty('title');
  });
});
```

---

# 15 — Final notes, timeline & hand-off artifacts

* **Heavier upfront work accomplished**: Milestone 0 ensures consistent scaffolding & tests, which reduces iterations when AI generates feature files.
* **AI prompts & PR templates included** in this plan — reuse them as session scripts for your Vibe coding cycles.
* **Files I referenced from your docs**: Requirements Spec (REQs for welcome pages & WhatsApp), Architecture Doc (Prisma schema and system layers), Project Overview (feature rationale & KPIs).  &#x20;

---

If you want I can **immediately generate the first set of files** (repo scaffold, Prisma schema file, backend health endpoint, and the `app/welcome/[id]/page.tsx`) as code files in the repo structure — ready to run locally — using the prompts above (I’ll produce the exact file contents so you can paste them).
Would you like me to generate that first scaffold now (I’ll output the file contents and the exact commands to initialise and run locally)?
