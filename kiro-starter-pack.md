# GrandVillage — Kiro Starter Pack
_Paste the Master Prompt into Kiro, then attach this file as context._

---

## MASTER PROMPT (paste this into Kiro first)

> You are building **GrandVillage**, a two-sided marketplace web app where vetted retired seniors (60+) offer childcare, household help, and lessons, and parents browse profiles, book by the hour, and pay in-platform. Think "Airbnb meets Care.com, but the hosts are grandmas."
>
> **Stack (do not change):** Next.js 15 (App Router, TypeScript) + Tailwind CSS + Supabase (Postgres, Auth, Storage) + Stripe Connect (Express, TEST mode). Deploy target: Vercel.
>
> **Two user roles:** `parent` and `senior` (host). One login, role stored on the profile.
>
> **Build order — follow strictly, one phase at a time. Do not skip ahead:**
> Phase 1: Scaffold + Supabase schema + seed data (4 fake seniors, 2 parents)
> Phase 2: Auth (email + Google via Supabase Auth), role selection on signup
> Phase 3: Parent browse (senior cards) + senior profile page
> Phase 4: Booking flow (pick slot → request → senior accepts → status changes)
> Phase 5: Stripe Connect test-mode payment (destination charge + platform fee split)
> Phase 6: Two-way reviews (only after completed bookings) + in-app messaging
> Phase 7: Senior dashboard (requests, availability, earnings) + parent dashboard (bookings)
> Phase 8: Trust & Safety page + verification badges (Stripe Identity stubbed, background check SIMULATED — see notes)
>
> **Rules:**
> - Senior-facing UI: 18px minimum font, high contrast, one task per screen, every icon has a text label, touch targets 48px+. No hamburger-only navigation.
> - Never publish children's photos or names publicly. Child info = ages only, visible only to the booked senior.
> - Background checks are SIMULATED in this build (mock endpoint returning pending → clear). Label the UI honestly: "Demo simulation."
> - No custom admin panel — Supabase dashboard is the admin tool.
> - All money is Stripe TEST mode. Never touch real payments.
> - Mobile-first. Every page must work at 390px width.
>
> **Start with Phase 1 now.** Scaffold the Next.js app, define the Supabase schema below, write the seed script, and confirm the app runs locally before moving on. Ask me before starting each new phase.

---

## 1. PRODUCT IN ONE PARAGRAPH

Lonely retired seniors need income and purpose; busy transplant parents need trusted, affordable help. GrandVillage matches them: seniors list what they offer (childcare, chores, lessons), parents browse, book, and pay by the hour. The platform keeps a fee per booking; seniors keep the rest. The emotional core: manufacturing "chosen family" for people whose real family is far away.

## 2. DATA MODEL (Supabase Postgres)

```sql
-- profiles: one row per user, role distinguishes parent vs senior
create table profiles (
  id uuid primary key references auth.users(id),
  role text not null check (role in ('parent','senior')),
  display_name text not null,
  bio text,
  photo_url text,
  hourly_rate_cents int,              -- seniors only
  services text[] default '{}',        -- seniors only: e.g. {childcare, cooking, tutoring}
  languages text[] default '{}',
  verification_status text default 'unverified',  -- unverified | id_pending | id_verified | bg_pending | bg_clear
  stripe_account_id text,              -- senior's Stripe Connect Express account
  created_at timestamptz default now()
);

-- availability: senior's weekly slots
create table availability (
  id uuid primary key default gen_random_uuid(),
  senior_id uuid references profiles(id) on delete cascade,
  day_of_week int check (day_of_week between 0 and 6),
  start_time time not null,
  end_time time not null
);

-- bookings: the core transaction
create table bookings (
  id uuid primary key default gen_random_uuid(),
  parent_id uuid references profiles(id),
  senior_id uuid references profiles(id),
  starts_at timestamptz not null,
  ends_at timestamptz not null,
  status text default 'requested'
    check (status in ('requested','confirmed','completed','cancelled')),
  stripe_payment_intent_id text,
  platform_fee_cents int,
  total_cents int,
  created_at timestamptz default now()
);

-- reviews: only after a completed booking, one per side
create table reviews (
  id uuid primary key default gen_random_uuid(),
  booking_id uuid references bookings(id) on delete cascade,
  author_id uuid references profiles(id),
  rating int check (rating between 1 and 5),
  text text,
  created_at timestamptz default now(),
  unique(booking_id, author_id)
);

-- messages: in-app chat per booking; contact info hidden until confirmed
create table messages (
  id uuid primary key default gen_random_uuid(),
  booking_id uuid references bookings(id) on delete cascade,
  sender_id uuid references profiles(id),
  body text not null,
  created_at timestamptz default now()
);
```

**Row-Level Security (enable on all tables):**
- Anyone can read senior profiles where `verification_status` in ('id_verified','bg_clear').
- Users can read/update only their own profile.
- Booking participants (parent_id or senior_id) can read their bookings; parents create bookings.
- Reviews readable by all; writable only by booking participants after `status='completed'`.
- Messages readable/writable only by booking participants.

## 3. KEY FLOWS

**Senior onboarding (4-step wizard, one group per step):**
1. About you → name, photo, bio, languages
2. Services & rates → services[], hourly_rate_cents
3. Availability → availability rows
4. Verification → Stripe Identity (stub: button flips status to id_verified) + background-check consent checkbox (required) → mock endpoint sets bg_pending, then bg_clear after 5s. Show status honestly as demo simulation.

**Booking flow:**
Browse seniors → senior profile → pick date/time → "Request booking" (status=requested) → senior sees request in dashboard → Accept (status=confirmed) → Stripe Checkout (test card 4242…) → payment splits: platform fee via `application_fee_amount`, rest to senior's Connect account → chat unlocks → after end time, "Mark complete" → both sides prompted to review.

**Payments (Stripe Connect, TEST mode only):**
- Senior onboarding: create Express account via API, redirect to Stripe hosted onboarding. Test magic: address line 1 = `address_full_match` passes verification.
- Charge: PaymentIntent with `transfer_data[destination]` = senior's account id, `application_fee_amount` = 15% of total.
- Demo payout: use test card `4000 0000 0000 0077` so funds land in *available* balance and the payout succeeds live.
- Skip webhooks if short on time: poll the Checkout Session status server-side after redirect.

## 4. PAGES (mobile-first, 390px)

**Parent side:** `/` browse (senior cards: photo, name, services, rating, price, distance) · `/seniors/[id]` profile (story, badges, reviews, availability, Book button) · `/bookings` my bookings · `/messages/[bookingId]` chat · `/safety` trust & safety page
**Senior side:** `/dashboard` requests + upcoming · `/dashboard/availability` manage slots · `/dashboard/earnings` balance + payout button · `/dashboard/profile` edit profile
**Shared:** `/login` · `/signup` (role picker) · `/onboarding` (senior wizard)

## 5. SEED DATA (Phase 1)

4 seniors: Rosa M. (68, childcare + cooking, Spanish/English, $18/hr, id_verified + bg_clear), James T. (71, tutoring + yard work, $20/hr), Priya S. (65, childcare + lessons, Hindi/English, $18/hr), Elena R. (70, cooking + companionship, $16/hr, verification pending — shows the pending state). 2 parents with 1 completed booking + reviews between them (so reviews render on day one).

## 6. DO NOT BUILD (hackathon scope cuts)

- Custom admin panel (use Supabase dashboard)
- Real Checkr background checks (simulate UI, label honestly)
- Real bank payouts / 1099s / disputes
- Compatibility/matchmaking algorithm (filters only: service, language, price, availability)
- Support chatbot (static Help page is enough)
- Kids' public profiles (ages only, private to booked senior)

## 7. CONVENTIONS

- TypeScript strict; no `any` without a comment.
- Server Components by default; `"use client"` only for interactive islands.
- All Stripe calls in Next.js route handlers (`/app/api/...`), never client-side, keys in env.
- Tailwind only, no extra UI kits. Warm palette: cream background, deep forest green primary, amber accents.
- Commit after every phase; deploy to Vercel from phase 1 so the demo URL is always live.
