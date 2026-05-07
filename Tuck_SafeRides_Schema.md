# Tuck SafeRides: Database Schema (MVP)

**Author:** Sri
**Last updated:** May 2026
**Status:** Locked for MVP build

This document captures the database schema for the SafeRides MVP, the design decisions behind each table, and the key application logic that interacts with the schema. It is the source of truth for setting up Supabase tables and writing the data layer.

---

## Overview

The MVP uses three tables in the `public` schema, plus Supabase's managed `auth.users` for authentication.

| Table | Purpose |
|-------|---------|
| `auth.users` | Supabase-managed. Email, magic link, sessions. Not modified directly. |
| `public.profiles` | User profile data (name, phone, photo). 1:1 with `auth.users`. |
| `public.volunteer_signups` | Records of which user is signed up for which Wed/Thu/Fri/Sat shift. |
| `public.ride_requests` | Ride requests with status, locations, and timestamps for every state transition. |

## Design conventions

- **Naming:** snake_case for tables and columns.
- **Primary keys:** UUID, generated via `gen_random_uuid()`.
- **Timestamps:** `timestamptz` everywhere. Storage in UTC, display in local time.
- **Soft delete:** Not used in MVP. All deletes are hard.
- **Foreign keys:** Explicit `ON DELETE` clause for every FK.
- **Status fields:** Text + CHECK constraint, not Postgres enum types (for migration flexibility).
- **Indexes:** Explicit on query patterns; no over-indexing.

## What we explicitly didn't build

Two tables were considered during design and dropped:

1. **`ride_state_transitions`.** A separate audit log of every ride status change. Collapsed into denormalized `*_at` timestamp columns on `ride_requests`. Reasoning: success metrics computable from timestamps, "who triggered" not in PRD requirements, simpler model wins for MVP.

2. **`co_chair_actions`.** An audit log of co-chair mutating actions (signup deletions). Dropped entirely. Reasoning: 2-4 co-chairs at most, low deletion frequency, disputes resolvable via email faster than via DB query.

Both can be added in v2 with non-breaking migrations.

---

## Table 1: `public.profiles`

User profile information. One row per `auth.users` row.

```sql
create table profiles (
  id           uuid primary key references auth.users(id) on delete cascade,
  first_name   text not null,
  last_name    text not null,
  email        text not null unique,
  phone        text not null,
  photo_url    text,
  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now()
);
```

### Design decisions

- **`first_name` + `last_name` instead of single `name`.** Enables first-name greetings ("Hi Maya"), compact homepage card formatting ("Ben S. & Maya P."), and clean sorting. Easy to collapse later if needed.
- **No `role` column.** Co-chair status comes from a hardcoded email list in app config. Driver status is dynamic per-shift via `volunteer_signups`. Passenger is the default. Adding a column would create two sources of truth.
- **`phone` is required.** The app exposes phone numbers as a fallback channel after a request is matched, so we cannot have nullable phone numbers. This forces profile completion at signup.
- **`photo_url` is nullable.** UI shows initials as fallback.
- **`email` denormalized from `auth.users`.** Slight redundancy, but spares a JOIN for the most common queries (homepage card, co-chair this-week list).
- **`on delete cascade` from `auth.users`.** If auth identity is deleted, profile follows.

### Indexes

Implicit only (primary key, unique email). No additional indexes needed in MVP.

### RLS policies

- `SELECT`: authenticated users can read profiles of users they need to see (drivers and passengers in active matches, calendar volunteers, tonight's drivers card). Specific scoping during build.
- `INSERT` / `UPDATE`: users can manage their own row only.
- `DELETE`: cascade only via `auth.users`.

---

## Table 2: `public.volunteer_signups`

Records of which user is signed up for which shift date. One row per (user, shift_date).

```sql
create table volunteer_signups (
  id           uuid primary key default gen_random_uuid(),
  user_id      uuid not null references profiles(id) on delete cascade,
  shift_date   date not null,
  created_at   timestamptz not null default now(),
  unique (user_id, shift_date),
  check (extract(dow from shift_date) in (3, 4, 5, 6))
);

create index volunteer_signups_shift_date_idx
  on volunteer_signups(shift_date);
```

### Design decisions

- **One row per signup, not one row per shift.** Cleaner audit, easier "who signed up first" queries, trivial uniqueness enforcement. Alternative (one row per shift_date with two user_id columns) was considered and rejected.
- **No separate `shifts` table.** Shifts are deterministic (Wed/Thu/Fri/Sat); calendar dates are generated client-side; signup queries handle the rest.
- **Day-of-week CHECK constraint.** The `extract(dow from shift_date) in (3, 4, 5, 6)` clause enforces that only Wednesday (3), Thursday (4), Friday (5), and Saturday (6) dates can be inserted. Postgres week starts on Sunday=0.
- **Shift hours computed from day-of-week, not stored.** Wed = 10pm-12am, Thu/Fri/Sat = 10pm-2am. App-side helper function derives this.
- **Hard delete on co-chair removal.** No soft-delete column. Audit history was considered (`co_chair_actions` table) and explicitly deferred from MVP.
- **`on delete cascade` from `profiles`.** If a user is deleted, their signups go with them.

### Indexes

- Primary key (implicit)
- Unique (user_id, shift_date) (implicit)
- Explicit B-tree on `shift_date` for calendar queries

### Required application logic

**Race condition protection on partner-claim.** Two volunteers tapping "claim partner slot" simultaneously could both succeed without atomic guarantees. The app calls a Postgres function to handle the claim atomically:

```sql
-- Pseudo-shape; exact implementation during build
create or replace function claim_shift(p_user_id uuid, p_shift_date date)
returns volunteer_signups
language plpgsql
as $$
declare
  current_count int;
  new_signup volunteer_signups;
begin
  -- Lock relevant rows for this shift_date
  perform 1 from volunteer_signups
  where shift_date = p_shift_date for update;

  select count(*) into current_count
  from volunteer_signups
  where shift_date = p_shift_date;

  if current_count >= 2 then
    raise exception 'shift_full';
  end if;

  insert into volunteer_signups (user_id, shift_date)
  values (p_user_id, p_shift_date)
  returning * into new_signup;

  return new_signup;
end;
$$;
```

User-facing behavior on race: first write wins; second user sees an error message and a refresh prompt.

### RLS policies

- `SELECT`: any authenticated user (calendar visibility).
- `INSERT`: authenticated users can insert only their own row (`user_id = auth.uid()`). Calls go through `claim_shift` function in practice.
- `UPDATE`: not allowed.
- `DELETE`: only co-chairs (per config list). This enforces the "volunteers cannot cancel themselves" rule at the database layer, not just the UI.

---

## Table 3: `public.ride_requests`

Ride requests submitted by passengers, with status and timestamps for every state transition.

```sql
create table ride_requests (
  id              uuid primary key default gen_random_uuid(),
  passenger_id    uuid not null references profiles(id) on delete cascade,
  shift_date      date not null,
  pickup_text     text not null,
  pickup_lat      numeric,
  pickup_lng      numeric,
  dropoff_text    text not null,
  rider_count     smallint not null default 1 check (rider_count between 1 and 4),
  notes           text,
  status          text not null default 'queued' check (
                    status in ('queued', 'claimed', 'en_route', 'arrived',
                               'in_vehicle', 'completed', 'cancelled', 'no_show')
                  ),
  requested_at    timestamptz not null default now(),
  claimed_at      timestamptz,
  en_route_at     timestamptz,
  arrived_at      timestamptz,
  in_vehicle_at   timestamptz,
  completed_at    timestamptz,
  cancelled_at    timestamptz,
  no_show_at      timestamptz
);

create index ride_requests_shift_date_status_idx
  on ride_requests(shift_date, status, requested_at);

create index ride_requests_passenger_idx
  on ride_requests(passenger_id);

create unique index ride_requests_one_open_per_passenger
  on ride_requests(passenger_id)
  where status in ('queued', 'claimed', 'en_route', 'arrived', 'in_vehicle');
```

### Design decisions

- **Denormalized timestamps for each state, no separate transitions table.** `claimed_at`, `en_route_at`, `arrived_at`, `in_vehicle_at`, `completed_at`, `cancelled_at`, `no_show_at` all live on the row. Loses "who triggered" info but is much simpler. Success metrics fully computable.
- **One open request per passenger,** enforced via the partial unique index. Cancel or complete the first before requesting another.
- **`shift_date` is stored explicitly,** computed by the app at submission time. This handles the cross-midnight case: a request submitted at 1:30am Saturday belongs to the Friday shift (`shift_date = Friday's date`), not Saturday.
- **Pickup as required text + nullable lat/lng.** Captures coordinates when GPS is available, falls back to user-entered text. Future map features get coordinates for free; MVP doesn't depend on them.
- **`rider_count` capped at 4.** Sanity check; the typical car holds 4 passengers + 2 drivers.
- **`status` as text + CHECK constraint** instead of a Postgres enum type, for ease of adding states later.
- **`on delete cascade` from `profiles`.** When a user is deleted (post-graduation), their ride history goes with them. Privacy-respecting default; pickup addresses could be identifying.

### Indexes

- Primary key (implicit)
- `(shift_date, status, requested_at)` for the queue query: "show me queued requests for tonight's shift in FIFO order."
- `(passenger_id)` for "show me this passenger's request history."
- Partial unique index for the one-open-request-per-passenger constraint.

### Required application logic

**Cross-midnight shift_date assignment** at submission time. App rule:
- If current time is between 10pm and 11:59pm on a service night → `shift_date = today`
- If current time is between 12am and 2am → `shift_date = yesterday` (prior night's shift)
- Otherwise → reject the request (outside service hours)

**Service hours enforcement.** New ride request submission is rejected outside service hours. Existing live requests are unaffected.

**Status transitions.** App writes the new status and the corresponding `*_at` timestamp atomically.

**Stale queued requests.** Queries filter by today's `shift_date`, so unclaimed requests from prior nights don't appear in the active queue. No automatic cleanup; data persists for analytics.

### RLS policies

- `SELECT`: passenger sees own requests; on-shift drivers see today's queue; co-chairs see everything.
- `INSERT`: any authenticated user can create a request for themselves (`passenger_id = auth.uid()`).
- `UPDATE`: passenger can update their own request (cancellation only); on-shift driver can update status of claimed requests; co-chairs can update anything.
- `DELETE`: not allowed.

---

## Key application logic outside the schema

These rules live in the app, not the database:

- **Magic link auth flow:** unified email entry, routes new users to profile setup, returning users to homepage.
- **Profile setup flow on first sign-in:** redirect to profile completion form (name, phone, optional photo); block other features until done.
- **Co-chair role check:** lookup current user's email against the hardcoded list in app config.
- **Calendar generation:** Wed/Thu/Fri/Sat dates for next 8 weeks computed client-side, cross-referenced with `volunteer_signups` for fill state.
- **Cross-midnight shift_date computation** for ride requests.
- **Service hours enforcement** for new request submission.
- **Atomic shift claim** via the `claim_shift` Postgres function described above.

## Future migration paths

The MVP schema is designed to allow non-breaking additions:

- **Adding `co_chair_actions`:** Create the table; existing logic unchanged. Application code starts writing audit rows on signup deletion.
- **Adding `ride_state_transitions`:** Create the table; existing `*_at` columns can stay as denormalized convenience or be dropped in a later migration. Backfill historical transitions from existing timestamp columns.
- **Tuck SSO:** Swap Supabase auth provider; `profiles` structure unchanged. Co-chair role moves from config-list lookup to a proper role column or separate table.
- **Live driver location:** `pickup_lat`/`pickup_lng` already exist; add `driver_lat`/`driver_lng` to track active rides.
- **Service-area validation:** Add CHECK constraint or trigger to validate `pickup_lat`/`pickup_lng` against service-area polygon.

## Open implementation questions for build phase

These are not design decisions but build-time questions to resolve when writing code:

1. Exact RLS policy SQL (drafted above as intent, needs concrete WHERE clauses).
2. Concrete implementation of `claim_shift` function including error code conventions.
3. Whether to use Supabase realtime subscriptions or polling for queue updates.
4. Whether to write a separate `claim_ride` Postgres function for the ride_requests claim flow (recommended, similar reasoning as `claim_shift`).
