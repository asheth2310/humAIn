# GrandVillage 🏡

> *Other platforms help you hire a stranger. GrandVillage helps you find family.*

A two-sided marketplace where **vetted retired seniors (60+)** offer childcare, household help, and lessons — and **parents** browse profiles, book by the hour, and pay in-platform. Seniors earn income and purpose; parents get trusted, compassionate help.

Built for **Hacks for Humanity 2026** · ASU Project Humanities · **Team 18 — HumAIn**

---

## 💔 The Problem

- **Loneliness epidemic** — millions of retired seniors live alone, far from family, with fixed incomes and no daily purpose.
- **Childcare is unaffordable** — young families, especially transplant parents with zero local support, can't afford professional care.
- **Seniors have no flexible work** — retirees who want to keep contributing have almost no dignified, flexible earning options.

## 💡 The Solution

GrandVillage matches both sides of the crisis: seniors list what they're good at, parents book them by the hour. The platform keeps a fee per booking; seniors keep the rest. Background checks, ID verification, and two-way reviews keep everyone safe.

The product isn't really childcare — it's a **chosen-family machine** for people whose real family is far away.

### What makes it different

| Everyone else | GrandVillage |
|---|---|
| Seniors are customers who *need* care | Seniors are providers who *give* care — and get paid |
| Listings sites (RentAGrandma) just list profiles | Community trust infrastructure: volunteer vetting, senior-center partnerships |
| Transactions measured in bookings | Relationships measured in months ("length of relationship" is a core metric) |
| Only one side benefits | Both sides heal: parents get help, seniors get income **and** purpose |

---

## ✨ Core Features

1. **Profiles** — senior & parent profiles: photo, story, services, languages, rates, verification badges, reviews
2. **Browsing / Marketplace** — filter by service, language, price, availability
3. **Matching** — compatibility filters (language, culture, parenting style); not just who's available
4. **Booking flow** — pick a slot → request → senior confirms → statuses (requested / confirmed / completed)
5. **Payments** — in-platform via Stripe Connect: parent pays, platform keeps a fee, senior gets paid out
6. **Two-way feedback & messaging** — reviews only after completed bookings; chat unlocks on confirmation
7. **Senior onboarding & quality assurance** — 4-step wizard, ID verification, background-check consent
8. **Trust & Safety** — public safety page naming every check component

---

## 🏗️ Architecture

### System overview

```
┌─────────────────────┐         ┌──────────────────────┐         ┌─────────────────┐
│  CLIENT             │         │  BACKEND (Supabase)  │         │  PAYMENTS       │
│  Next.js 15         │────────▶│  Postgres            │         │  Stripe Connect │
│  TypeScript         │         │  Auth (email/Google) │         │  Express (test) │
│  Tailwind · Vercel  │◀────────│  Storage (photos)    │         │  Destination    │
└─────────────────────┘         │  Row-Level Security  │         │  charges + fee  │
                                └──────────────────────┘         └─────────────────┘
           │                              │                                │
           └───────── API routes (/app/api/*) ─────────────────────────────┘
```

No separate backend server — Next.js route handlers + Supabase cover all server logic.

### Data model

```sql
profiles      -- id, role (parent|senior), name, bio, photo, rate, services[], languages[],
               -- verification_status, stripe_account_id
availability  -- senior_id, day_of_week, start_time, end_time
bookings      -- parent_id, senior_id, starts_at, ends_at, status,
               -- stripe_payment_intent_id, platform_fee_cents, total_cents
reviews       -- booking_id, author_id, rating, text  (post-completion only, one per side)
messages      -- booking_id, sender_id, body  (contact hidden until booking confirmed)
```

Row-Level Security: anyone reads verified senior profiles · users manage only their own data · booking participants see their bookings/messages · reviews writable only after completion.

### Key flows

**I. Senior onboarding** — 4-step wizard, one group per screen:
`About you → Services & rates → Availability → Verification`
→ Stripe Identity check → background-check consent → profile goes live

**II. Booking**
`Browse seniors → profile → pick slot → request → senior accepts → Stripe Checkout → payment splits automatically → chat unlocks → mark complete → mutual reviews`

**III. Payments (Stripe Connect, test mode)**
Parent charged via PaymentIntent → `application_fee_amount` (15%) stays with platform → remainder transferred to senior's Express account → senior sees earnings + payout button in dashboard.

**IV. Trust**
`ID Verified` badge = real (Stripe Identity) · background-check panel = simulated Checkr-pattern endpoint, honestly labeled as demo · reviews tied to completed bookings only · report/block on every profile.

### API routes (Next.js route handlers)

| Route | Purpose |
|---|---|
| `POST /api/stripe/onboard` | Create senior's Express account + hosted onboarding link |
| `POST /api/stripe/checkout` | Create PaymentIntent with platform-fee split |
| `POST /api/identity/verify` | Start Stripe Identity verification session |
| `POST /api/webhooks/stripe` | Payment + verification events (or poll session status if short on time) |

---

## 🔨 Build Workflow

Follow in order — each phase unlocks the next. Demo-critical path first.

| Phase | What | Done when |
|---|---|---|
| **0. Setup** | Scaffold Next.js + Tailwind, Supabase project, Vercel deploy, env vars | App runs locally + deploys |
| **1. Data** | Schema (5 tables above) + RLS policies + seed script (4 seniors, 2 parents, 1 completed booking) | Seed loads, dashboards show data |
| **2. Auth** | Supabase Auth (email + Google), role picker on signup | Login as parent *and* senior works |
| **3. Browse** | Senior cards (photo, services, rating, price) + filters + senior profile page | Can view every seeded senior |
| **4. Booking** | Slot picker → request → senior accept/decline → status changes | Full request cycle works end-to-end |
| **5. Payments** | Stripe Connect test mode: onboarding link, Checkout, fee split, earnings page | Test card completes a split payment |
| **6. Trust** | Stripe Identity badge, simulated background-check UI, Trust & Safety page | Badges render on profiles |
| **7. Social** | Two-way reviews (post-completion), in-app messaging | Review + chat work on a completed booking |
| **8. Dashboards** | Senior: requests, availability, earnings · Parent: bookings, receipts | Both dashboards functional |
| **9. Polish** | Senior-friendly pass (18px type, contrast, big buttons), landing page, demo script rehearsal | Demo runs clean twice in a row |

**Cut list (if time runs short):** custom admin panel → use Supabase dashboard · real background checks → simulate honestly · support chatbot → static Help page · matchmaking algorithm → filters only.

---

## 🛠️ Tech Stack

| Layer | Choice |
|---|---|
| Frontend | Next.js 15 (App Router, TypeScript), Tailwind CSS |
| Backend | Supabase — Postgres, Auth, Storage, RLS |
| Payments | Stripe Connect (Express, test mode) |
| Identity | Stripe Identity |
| Hosting | Vercel |

## 🚀 Getting Started

```bash
# 1. Scaffold
npx create-next-app@latest grandvillage --typescript --tailwind --app
cd grandvillage
npm i @supabase/supabase-js @supabase/ssr stripe

# 2. Environment (.env.local)
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...
STRIPE_SECRET_KEY=sk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...

# 3. Database
# Run the schema in /supabase/schema.sql (see Architecture → Data model), enable RLS, then:
npm run db:seed

# 4. Run
npm run dev
```

**Stripe test-mode recipes:** senior onboarding → set address line 1 to `address_full_match` (passes verification) · parent payment → card `4242 4242 4242 4242` · demo payout → card `4000 0000 0000 0077` (funds land in *available* balance so the payout succeeds live).

---

## 👵 Senior-Friendly Design Rules

The host side is used by adults 65+. Non-negotiable:

- 18px minimum body type, ≥4.5:1 contrast (aim 7:1 on key text)
- One task per screen · guided stepper flows · progress shown
- Every icon paired with a text label · 48px minimum touch targets
- Persistent Help / phone contact · plain language, no jargon
- Must survive 200% browser zoom · mobile-first at 390px

## ⚖️ Compliance Notes (US)

- **No childcare license needed** — seniors babysit in the *parents'* home (nanny/sitter exemptions exist in many states, but rules vary by state); the platform is a marketplace, like other care platforms. Never present as licensed childcare; keep services non-medical.
- **Benefits-safe** — at full retirement age (67) there is **no** Social Security earnings limit; under 67, the 2026 cap is $24,480/yr. The app shows seniors an earnings tracker.
- **COPPA** — never publish children's photos or names publicly. Ages only, visible solely to the booked senior.
- **Background checks** — FCRA consent checkbox built into senior onboarding before any check runs.

---

## 👥 Team — HumAIn (Team 18)

| Member | Roles |
|---|---|
| **Aagam Sheth** | Coder / Designer / Community Advocate |
| **Mohamed Fayaz Buhari** | Project Manager / Content Creator / Community Advocate |
| **Nirmalraju Kangeyan** | Coder / Designer / Researcher / Mentor Liaison / Community Advocate |
| **Thanh Tan Nguyen** | Entrepreneur / Pitch Specialist / Mentor Liaison / Community Advocate |

*Hacks for Humanity 2026 · ASU Project Humanities · Memorial Union, Tempe · Oct 9–11, 2026*

---

## 📜 License

MIT — built with compassion at Hacks for Humanity 2026. 💛
