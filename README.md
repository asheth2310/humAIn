# GrandVillage 🏡

> *Care.com helps you hire a stranger. GrandVillage helps you find family.*

A two-sided marketplace where **vetted retired seniors (60+)** offer childcare, household help, and lessons — and **parents** browse profiles, book by the hour, and pay in-platform. Seniors earn income and purpose; parents get trusted, compassionate help. Built for **Hacks for Humanity 2026** (ASU Project Humanities) by **Team 18 — HumAIn**.

## The Problem

- **Loneliness epidemic** — millions of retired seniors live alone, far from family, with no income beyond fixed benefits and no daily purpose.
- **High cost of living** — professional childcare is unaffordable for young families, especially transplant parents with zero local support network.
- **No competitive work for seniors** — retirees who want to keep contributing have almost no flexible, dignified earning options.

## The Solution

GrandVillage matches both sides: seniors list what they're good at, parents book them by the hour. The platform keeps a fee per booking; seniors keep the rest. Background checks, ID verification, and two-way reviews keep everyone safe. The product isn't really childcare — it's a **chosen-family machine**.

**What makes it different:** on every other platform, seniors are customers who *need* care. On GrandVillage, seniors are the ones *giving* care — and getting paid for it. Listings sites (RentAGrandma) just list profiles; GrandVillage wraps the match in community trust infrastructure: volunteer-assisted vetting, senior-center partnerships, and relationships measured in months, not hours.

## Core Features

| # | Feature | Status |
|---|---------|--------|
| 1 | Senior & parent profiles (photo, bio, services, rates, verification badges) | Planned |
| 2 | Browse / marketplace (filter by service, language, price, availability) | Planned |
| 3 | Matching (compatibility filters: language, culture, parenting style) | Planned |
| 4 | Booking flow (pick slot → request → senior confirms → statuses) | Planned |
| 5 | Payments (Stripe Connect: parent pays, platform fee split, senior payout) | Planned |
| 6 | Two-way reviews & in-app messaging | Planned |
| 7 | Senior onboarding wizard + quality assurance (ID check, background-check consent) | Planned |
| 8 | Trust & Safety page | Planned |

## Tech Stack

- **Frontend:** Next.js 15 (App Router, TypeScript) + Tailwind CSS → Vercel
- **Backend:** Supabase (Postgres, Auth, Storage) — no separate server
- **Payments:** Stripe Connect (Express accounts, test mode) — destination charges with `application_fee_amount` splits
- **Identity:** Stripe Identity (ID verification badge); criminal background checks simulated via mock endpoint (honest demo labeling)

## Architecture

```
┌─────────────┐      ┌──────────────────┐      ┌──────────────┐
│  Next.js    │─────▶│    Supabase      │      │    Stripe    │
│  (Vercel)   │      │  Postgres+Auth   │      │   Connect    │
│  PWA-ready  │◀─────│  Storage (photos)│      │  (test mode) │
└─────────────┘      └──────────────────┘      └──────────────┘
       │                     │                         │
       └─────── API routes (/app/api) ─────────────────┘
```

**Data model:** `profiles` (role: parent/senior) → `availability` → `bookings` → `reviews` (post-completion only) + `messages` (per booking, contact hidden until confirmed). Row-Level Security on all tables.

**Key flows:**
1. *Senior onboarding:* 4-step wizard (about → services → availability → verification) → profile goes live
2. *Booking:* browse → profile → pick slot → request → senior accepts → Stripe Checkout → chat unlocks → complete → mutual reviews
3. *Trust:* "ID Verified" badge (real, Stripe Identity) + background-check status (simulated Checkr pattern, labeled as demo)

## Getting Started

```bash
# 1. Scaffold
npx create-next-app@latest grandvillage --typescript --tailwind --app
cd grandvillage
npm i @supabase/supabase-js @supabase/ssr stripe

# 2. Env (.env.local)
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...
STRIPE_SECRET_KEY=sk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...

# 3. Run
npm run dev
```

See [`kiro-starter-pack.md`](./kiro-starter-pack.md) for the full build prompt: exact DB schema, RLS policies, seed data, test-mode payment recipes, and the phase-by-phase build order for AI coding assistants.

## Senior-Friendly Design Rules

The host side is used by adults 65+: 18px minimum type, ≥4.5:1 contrast, one task per screen, text label on every icon, 48px touch targets, guided stepper onboarding, persistent Help/phone contact. No jargon, no hamburger-only nav.

## Compliance Notes (US)

- **No childcare license needed:** seniors babysit in the *parents'* home (nanny/sitter exemption, all 50 states); the platform is a marketplace, like Care.com.
- **Benefits-safe:** at full retirement age (67) there is **no** Social Security earnings limit; under 67, the 2026 cap is $24,480/yr. The app shows seniors an earnings tracker.
- **COPPA:** never publish children's photos/names publicly — ages only, visible solely to the booked senior.
- **Background checks:** FCRA consent checkbox built into onboarding.

## Team — HumAIn (Team 18)

| Member | Roles |
|--------|-------|
| Aagam Sheth | Coder / Designer / Community Advocate |
| Mohamed Fayaz Buhari | Project Manager / Content Creator / Community Advocate |
| Nirmalraju Kangeyan | Coder / Designer / Researcher / Mentor Liaison / Community Advocate |
| Thanh Tan Nguyen | Entrepreneur / Pitch Specialist / Mentor Liaison / Community Advocate |

*Hacks for Humanity 2026 · ASU Project Humanities · Memorial Union, Tempe · Oct 9–11, 2026*
