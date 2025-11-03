# MVP Launch – Founding Engineer 72hr Build Test

Thanks for taking this on.

This test is designed to see if you can stand up a **real, production-style slice** of our platform in 72 hours — not a toy app.

We’re building `app.mvplaunch.com` where founders can:
1. Apply,
2. Get accepted into a lobby,
3. See their build pipeline,
4. Start paying us.

This assignment is a mini version of that.

---

## 📦 What You’ll Build (Scope)

You have **72 hours** from the time we send you this repo to ship:

1. **Public Homepage (unauthenticated)**
   - Simple hero using MVP Launch brand (red/black/white).
   - CTA: “Start Application”.

2. **Application Flow (unauthenticated → authenticated)**
   - Form to collect: name, email, company, current revenue (select), what they want to build (text), country.
   - On submit → create a “founder” record in DB.
   - After submit → ask user to create an account (auth).

3. **Auth**
   - Use **Clerk** or **Auth.js** (your choice) — we just want modern best practice.
   - Email/password or magic link is fine.
   - Auth should protect the dashboard.

4. **Lobby / Dashboard (authenticated)**
   - After login, user should see a dashboard with **status steps**:
     - `Application received`
     - `Placement (7 days)`
     - `MVP build (8 weeks)`
     - `Post-launch`
   - Each step should show:
     - status (pending / in progress / done)
     - short description
     - last updated
   - This can be static data seeded in DB — we want to see your data modeling.

5. **Onboarding Task List**
   - On the dashboard, show a “Next Steps” card:
     - “Complete profile”
     - “Add billing method (mock)”
     - “Upload product brief”
   - Mark tasks as done (simple toggle, persisted).

6. **Admin View (very basic)**
   - Route for us: `/admin`
   - Protected by role (seed an admin user).
   - Shows list of applicants + their status.
   - Ability to change status to “Accepted”.

That’s it. Small but real.

---

## 🧱 Tech Stack (what we prefer)

Use this or something very close:

- **Frontend:** Next.js 14+ (App Router)
- **Styling:** TailwindCSS
- **Auth:** Clerk or Auth.js
- **DB:** Postgres (local or Supabase)
- **ORM:** Prisma (preferred) or Drizzle
- **API:** Next.js route handlers or tRPC
- **Deployment target:** assume **Vercel** for frontend, **Supabase / Railway / Render** for DB
- **AI assist:** you can use Vercel AI SDK / simple OpenAI call to generate UI copy for sections — we want to see you can wire AI in fast.

If you have a better modern setup, explain it in your notes.

---

## 🎨 Branding Notes

- Primary: `#FF3B30` (red)
- Text: near-black / dark gray
- Keep it clean, founder-facing (think Stripe + Deel + Toptal)
- Put our logo placeholder in the navbar: `MVP LAUNCH`

We don’t care about pixel perfection — we care that you can ship a **coherent app shell**.

---

## 📚 Data Model (example)

Use something like this (Prisma example):

```ts
model User {
  id            String   @id @default(cuid())
  email         String   @unique
  name          String?
  role          Role     @default(USER)
  applications  Application[]
  createdAt     DateTime @default(now())
}

model Application {
  id             String   @id @default(cuid())
  userId         String
  user           User     @relation(fields: [userId], references: [id])
  companyName    String?
  revenueRange   String?
  projectSummary String?
  country        String?
  status         AppStatus @default(RECEIVED)
  createdAt      DateTime  @default(now())
  updatedAt      DateTime  @updatedAt
}

enum Role {
  USER
  ADMIN
}

enum AppStatus {
  RECEIVED
  REVIEWING
  ACCEPTED
  REJECTED
}
