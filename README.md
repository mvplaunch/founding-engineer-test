# MVP Launch – Founders Platform (`app.mvplaunch.com`)

This repo contains the codebase for **MVP Launch**, the internal + founder-facing platform that lives at:

- Production: **https://app.mvplaunch.com**
- Staging: **https://staging.mvplaunch.com** (same app, staging database + keys)

The goal is simple:

> In ~6 weeks we want real founders applying, being accepted into the Lobby, paying, and moving into builds through this platform.  
> No throwaway prototypes – this should be production-ready, tested, and something we’re comfortable putting paying clients into.

The **initial product focus** is:

1. **Application** – public application form + review flow  
2. **Lobby (App Shell)** – gated area for accepted founders before build starts  
3. **In-App features** – Dashboard / Build / Talent / Billing / Referral / Profile / Settings / Support  

The engineer should focus on **Application → Lobby** first so we can use that flow within a few weeks, then layer in the rest of the app.

---

## 1. Tech Stack

We’re optimising for **speed, maintainability, and a small but serious user base** (we don’t expect 2,000+ active users any time soon).

### Core

- **Framework:** Next.js 15+ (App Router, React Server Components)
- **Language:** TypeScript (strict mode)
- **UI:** Tailwind CSS + shadcn/ui
- **ORM:** Prisma
- **Database:** PostgreSQL (managed: Neon/Railway/RDS, etc.)
- **Auth:** Clerk (Email + OAuth)

  > Important: use Clerk, but **wrap it in internal abstraction layers** so we’re not locked in. See **Auth Abstraction** below.

- **Payments:** Stripe  
  - v1: one-off + subscription payments from founders  
  - v2: Stripe Connect for talent payouts
- **Notifications:** Knock  
  - Email provider: SendGrid  
  - SMS provider: Twilio
- **Support:** Intercom (in-app widget + basic help centre)
- **Error Monitoring:** Sentry
- **Hosting:** Vercel (staging + production projects)
- **Version Control:** GitHub
- **E2E Tests:** Playwright
- **Unit/Integration Tests:** Vitest or Jest (pick one)
- **Secrets Management:** 1Password shared vault

AI tools (ChatGPT, Copilot, etc.) can be used for boilerplate and scaffolding, but you must **own the implementation**. You will be expected to explain architecture and key decisions.

We are **not** using Supabase for this project. It’s standard **Next.js + Prisma + Postgres**.

---

## 2. High-Level Architecture

This is a multi-tenant SaaS:

- **Tenants:** Founder companies / projects
- **Actors:**
  - Founder (applicant → accepted → active client)
  - Internal team (review applications, manage billing, manage builds)

Top-level flows:

1. **Public Application**
2. **Lobby** (post-acceptance, pre-build)
3. **App** (during build and ongoing relationship)

### Proposed Route Structure (App Router)

You can refine names, but keep the structure and intent:

```txt
app
├─ (public)
│  ├─ page.tsx                         -> marketing/placeholder or redirect
│  ├─ apply/
│  │   └─ page.tsx                     -> Application form
│  └─ apply/success/
│      └─ page.tsx                     -> “Thanks, we’ve got your application”
│
├─ (auth)                              -> Clerk routes
│  ├─ sign-in/[[...index]]/page.tsx
│  └─ sign-up/[[...index]]/page.tsx
│
├─ (app)
│  ├─ lobby/
│  │   └─ page.tsx                     -> Lobby home (checklist, next steps)
│  └─ app/                             -> Authenticated app shell
│      ├─ layout.tsx                   -> Nav, sidebar, header, Intercom, etc.
│      ├─ dashboard/page.tsx
│      ├─ build/page.tsx
│      ├─ talent/page.tsx
│      ├─ billing/page.tsx
│      ├─ referral/page.tsx
│      ├─ profile/page.tsx
│      ├─ settings/page.tsx
│      └─ support/page.tsx
│
└─ (admin)
   ├─ layout.tsx                       -> Internal admin shell
   ├─ applications/page.tsx            -> Review + change statuses
   └─ founders/page.tsx
Frontend order of implementation (mandatory):

/apply (Application)

/lobby (Lobby / App Shell)

/app/... features (Dashboard, Billing, etc.)

We’ll do deep Figma design passes later. For now, use shadcn components + Tailwind and keep things clean and consistent.

3. 6-Week Delivery Plan
We want this shipped in ~6 weeks with no cutting corners. That includes proper testing, error handling, and production-ready quality.

Week 1 – Project & Infra Setup
Next.js 15 + TypeScript configured.

Tailwind + shadcn/ui set up.

Prisma + Postgres configured; initial migrations created.

Clerk wired via internal auth layer (lib/auth).

Stripe SDK installed; basic customer creation stubbed.

Knock, SendGrid, Twilio, Intercom, Sentry SDKs installed with minimal bootstrap.

Vercel projects created:

mvplaunch-app-staging → https://staging.mvplaunch.com

mvplaunch-app-production → https://app.mvplaunch.com

CI (GitHub Actions) set up to run on PR + pushes:

Typecheck

Lint

Unit tests

Playwright E2E (can be a limited subset initially)

Playwright harness created with at least one working test hitting /apply.

Week 2 – Application Flow (Public → Internal Review)
Implement /apply page:

Multi-step or single-page form capturing:

Name, email, timezone

Company / project details

Current revenue / stage

What they want us to build

Client + server validation (zod or equivalent).

Prisma models for application data (see Data Model below).

Submission logic:

Writes to DB.

Creates or links a Clerk user (if they don’t have one yet).

Triggers Knock notification:

Internal: send to internal team channel/email.

Applicant: “We’ve received your application.”

Additional pages:

/apply/success confirmation page.

Admin:

Basic admin Applications view under /admin/applications:

List + filter applications.

Ability to change status: NEW → IN_REVIEW → ACCEPTED → REJECTED.

Playwright tests:

User can submit application.

Admin can log in and see the new application.

Week 3 – Lobby (Accepted Founders)
Define state machine for a founder/project:

APPLIED, ACCEPTED_PENDING_PAYMENT, LOBBY, ACTIVE_BUILD.

When an application is marked ACCEPTED:

Create a Founder + Project record.

Trigger Knock email to founder with “next steps”.

Implement /lobby:

Auth-gated (Clerk, via our abstraction).

Shows the founder their current status and a simple checklist, e.g.:

Sign agreement.

Pay initial fee.

Upload assets (brief, files, etc.).

Book kickoff call.

Stripe integration (v1):

Ability to trigger a payment link / Checkout session for the initial fee from Lobby.

After successful payment:

Stripe webhook updates projectStatus to LOBBY or ACTIVE_BUILD depending on flow.

Knock notification confirming payment.

Playwright tests:

Application accepted → founder can log into Lobby and see correct status.

Stripe test payment → project status transitions correctly.

Week 4 – App Shell + Early App Features
Implement /app shell:

Layout with sidebar navigation and top bar.

Intercom widget + Sentry error boundaries wired in layout.

Implement the first core pages:

/app/dashboard

Basic project overview: current status, next milestone, key dates.

/app/billing

Show Stripe invoices and current plan.

/app/profile

Ability to update name, timezone, company name, etc.

Stripe integration extended:

Store stripeCustomerId on Founder.

Store stripeSubscriptionId on Project if we move to subscriptions.

Playwright tests:

Authenticated navigation between /app/dashboard, /app/billing, /app/profile.

Basic assertions that data is loaded and visible.

Week 5 – Build, Talent, Referral, Settings, Support
Initial versions can be simple, but must be wired end-to-end.

/app/build

List milestones (Prisma Milestone model).

Basic statuses: NOT_STARTED, IN_PROGRESS, REVIEW, DONE.

/app/talent

Show assigned engineer/designer.

Contact info (email, Calendly link, etc.).

/app/referral

Show referral code or link.

Track basic stats (#clicks, #signups, #conversions) in DB.

/app/settings

Notification preferences (email/SMS toggles).

Company/legal info used on invoices.

/app/support

Intercom widget.

Optional backup support form storing tickets in DB.

Knock flows:

“Milestone started/completed.”

“New message from team” (future messaging system).

Week 6 – Hardening, Testing, and Launch Readiness
Finalise Playwright coverage for:

Application submission.

Admin acceptance.

Lobby usage.

Payment and transition into /app.

Core app navigation.

Add proper loading, error, and empty states throughout.

Wire Sentry to capture and tag relevant errors.

Add basic analytics/logging where useful.

Run a full end-to-end manual test using Stripe test mode and real email delivery.

4. Auth Abstraction (Clerk Wrapped)
We are using Clerk, but don’t want Clerk calls scattered everywhere.

Create lib/auth.ts (or similar) and treat it as the only entry point for auth:

ts
Copy code
// lib/auth.ts
import {
  auth as clerkAuth,
  currentUser as clerkCurrentUser,
} from "@clerk/nextjs/server";

export async function getSession() {
  return clerkAuth();
}

export async function requireSession() {
  const session = await clerkAuth();
  if (!session.userId) throw new Error("Unauthenticated");
  return session;
}

export async function getCurrentUser() {
  return clerkCurrentUser();
}

export type AppRole = "FOUNDER" | "ADMIN" | "INTERNAL";

export function getRoleFromSession(claims: any): AppRole {
  // Map Clerk org role / claims -> our internal roles.
  // e.g. map "org:admin" to "ADMIN", etc.
}
Rules:

Components, API routes, and server actions should import from lib/auth, not @clerk/... directly.

All role / org logic should go through helpers so we can swap providers in future if needed.

5. Data Model (Initial Prisma Schema)
This is a starting point. Adjust as needed, but keep the spirit and keep migrations clean.

prisma
Copy code
// prisma/schema.prisma

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id             String   @id @default(cuid())
  clerkUserId    String   @unique
  email          String   @unique
  name           String?
  role           UserRole @default(FOUNDER)
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt

  founders       Founder[]
}

model Founder {
  id               String    @id @default(cuid())
  user             User      @relation(fields: [userId], references: [id])
  userId           String
  companyName      String?
  website          String?
  stripeCustomerId String?
  createdAt        DateTime  @default(now())
  updatedAt        DateTime  @updatedAt

  projects         Project[]
  applications     Application[]
}

model Application {
  id             String            @id @default(cuid())
  founder        Founder?          @relation(fields: [founderId], references: [id])
  founderId      String?
  name           String
  email          String
  companyName    String?
  website        String?
  details        String?
  revenue        String?
  status         ApplicationStatus @default(NEW)
  createdAt      DateTime          @default(now())
  updatedAt      DateTime          @updatedAt

  project        Project?
}

model Project {
  id                  String         @id @default(cuid())
  founder             Founder        @relation(fields: [founderId], references: [id])
  founderId           String
  name                String
  status              ProjectStatus  @default(APPLIED)
  stripeSubscriptionId String?
  createdAt           DateTime       @default(now())
  updatedAt           DateTime       @updatedAt

  milestones          Milestone[]
}

model Milestone {
  id             String          @id @default(cuid())
  project        Project         @relation(fields: [projectId], references: [id])
  projectId      String
  title          String
  status         MilestoneStatus @default(NOT_STARTED)
  dueDate        DateTime?
  createdAt      DateTime        @default(now())
  updatedAt      DateTime        @updatedAt
}

enum UserRole {
  FOUNDER
  ADMIN
  INTERNAL
}

enum ApplicationStatus {
  NEW
  IN_REVIEW
  ACCEPTED
  REJECTED
}

enum ProjectStatus {
  APPLIED
  ACCEPTED_PENDING_PAYMENT
  LOBBY
  ACTIVE_BUILD
  COMPLETED
}

enum MilestoneStatus {
  NOT_STARTED
  IN_PROGRESS
  REVIEW
  DONE
}
Migrate using:

prisma migrate dev in development

prisma migrate deploy in staging/production

6. Environments, Domains & Deployment
Vercel Projects
Two Vercel projects share this repo:

Staging

Name: mvplaunch-app-staging

Branch: staging

Domain: staging.mvplaunch.com

Uses staging Postgres, Stripe test keys, etc.

Production

Name: mvplaunch-app-production

Branch: main

Domain: app.mvplaunch.com

Uses production Postgres, Stripe live keys (when ready).

Vercel Preview Deployments are automatically created for each PR targeting staging or main.

Git Workflow
main – production, protected.

staging – integration branch, auto-deploys to staging.

Feature branches → PR into staging.

Promote from staging → main via PR once stable.

7. Secrets & Environment Variables (1Password)
All secrets live in a shared 1Password vault.

Process
Install 1Password (app + browser extension, CLI if needed).

Get access to the shared vault.

Copy relevant secrets into .env.local for local dev.

Add the same keys to Vercel Environment Variables for staging/production.

Never commit .env* files.

Required Env Vars (v1)
bash
Copy code
DATABASE_URL=postgres://...

NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=...
CLERK_SECRET_KEY=...

STRIPE_SECRET_KEY=...
STRIPE_WEBHOOK_SECRET=...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=...

KNOCK_API_KEY=...
SENDGRID_API_KEY=...
TWILIO_ACCOUNT_SID=...
TWILIO_AUTH_TOKEN=...

SENTRY_DSN=...

INTERCOM_APP_ID=...
INTERCOM_API_KEY=...
Document any new env vars you add in this README.

8. Local Development
Prerequisites
Node.js 20+

pnpm or npm (assume pnpm below)

Local or remote Postgres instance

Access to 1Password vault

Setup Steps
bash
Copy code
pnpm install

# Create env file and fill values from 1Password
cp .env.example .env.local

# Run migrations
pnpm prisma migrate dev

# (Optional) Seed initial data
pnpm prisma db seed

# Start dev server
pnpm dev
App will be available at: http://localhost:3000

9. Testing
Unit & Integration Tests
Use Vitest or Jest.

Place tests either in __tests__/ or as *.test.ts(x) next to modules.

Minimum coverage:

Helpers in lib/ (role mapping, status transitions, etc.).

Business logic around application acceptance and project status changes.

E2E Tests – Playwright
Configure Playwright with:

ts
Copy code
use: {
  baseURL: "http://localhost:3000",
}
Initial scenarios (expand over time):

Founder submits an application at /apply.

Admin signs in, views applications, and marks one as ACCEPTED.

Founder logs in and accesses /lobby with correct checklist.

Founder completes a Stripe test payment from Lobby and is redirected with updated project status.

Founder can access /app/dashboard and see basic project info.

Run Playwright in CI (on PRs and on merges to staging / main).
If runtime becomes an issue, you can split tests into “smoke” vs full suites.

CI – GitHub Actions
Workflow outline:

yaml
Copy code
on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main, staging]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm test
      - run: pnpm playwright test
10. Coding Standards & Non-Negotiables
No cutting corners. This platform will be used by paying clients.

TypeScript strict mode must stay enabled.

API routes and server actions must include:

Input validation (zod or equivalent).

Proper error handling and meaningful logging.

Each major feature should ship with:

At least one unit/integration test.

At least one relevant Playwright test.

Components should be small and composable, following shadcn patterns.

Keep a clean commit history with meaningful messages.

11. Frontend Build Order (Mandatory)
Build the frontend in this order:

Application

/apply + /apply/success

/admin/applications (internal review)

Lobby

/lobby with authentication and milestone checklist.

Stripe integration for initial payment.

App Features

/app shell and shared layout.

/app/dashboard, /app/billing, /app/profile.

Then /app/build, /app/talent, /app/referral, /app/settings, /app/support.

We will refine visuals with Figma later. For now, it just needs to be usable, consistent, and production quality.

12. Decision Log
Create and maintain docs/DECISIONS.md with:

Any deviations from this README (and why).

Trade-offs or shortcuts taken (and when they will be revisited).

“V2+” ideas that are intentionally out of scope for the first 6 weeks.

This keeps us aligned and makes it easy for future engineers to understand how we got here.

Copy code









Extended thinking



ChatGPT can make mistakes. Check important info.
