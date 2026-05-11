# Tuck SafeRides: Database Schema (MVP)

**Author:** Sri
**Last updated:** May 2026
**Status:** Locked for MVP build, v2

**Changes from v1:** Added `has_car` boolean field to `ride_requests` (Table 3) to support the dual-vehicle pickup scenario.

This document captures the database schema for the SafeRides MVP, the design decisions behind each table, and the key application logic that interacts with the schema. It is the source of truth for setting up Supabase tables and writing the data layer.

---

## Overview

The MVP uses three tables in the `public` schema, plus Supabase's managed `auth.users` for authentication.

| Table | Purpose |
|-------|---------|
| `auth.users` | Supabase-managed. Email, magic link, sessions. Not modified directly. |
| `public.profiles` | User profile data (name, phone, photo). 1:1 with `auth.users`. |
| `public.volunteer_signups` | Records of which user is signed up for which Wed/Thu/Fri/Sat shift. |
| `public.ride_requests` | Ride requests with status, locations, has-car flag, and timestamps for every state transition. |

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

- **`first_name` + `last_name` instead of single `name`.** Enables first-name greetings, compact homepage card formatting ("Ben S. & Maya P."), and clean sorting.
- **No `role` column.** Co-chair status comes from a hardcoded email list in app config. Driver status is dynamic per-shift via `volunteer_signups`. Passenger is the default.
- **`phone` is required.** The app exposes phone numbers as a fallback channel after a request is matched. Settings UI blocks save if the field is blank.
- **`photo_url` is nullable.** UI shows initials as fallback.
- **`email` denormalized from `auth.users`.** Spares a JOIN for the most common queries.
- **`on delete cascade` from `auth.users`.** If auth identity is deleted, profile follows.

### Indexes

Implicit only (primary key, unique email). No additional indexes needed in MVP.

### RLS policies

- `SELECT`: authenticated users can read profiles of users they need to see. Specific scoping during build.
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

- **One row per signup, not one row per shift.** Cleaner audit, easier "who signed up first" queries, trivial uniqueness enforcement.
- **No separate `shifts` table.** Shifts are deterministic; calendar dates are generated client-side.
- **Day-of-week CHECK constraint.** Enforces only Wed/Thu/Fri/Sat dates can be inserted. Postgres week starts on Sunday=0.
- **Shift hours computed from day-of-week, not stored.**
- **Hard delete on co-chair removal.** No soft-delete column.
- **`on delete cascade` from `profiles`.**

### Indexes

Primary key (implicit), unique (user_id, shift_date) (implicit), explicit B-tree on `shift_date`.

### Required application logic

**Race condition protection on partner-claim** via Postgres function:

```sql
create or replace function claim_shift(p_user_id uuid, p_shift_date date)
returns volunteer_signups
language plpgsql
as $$
declare
  current_count int;
  new_signup volunteer_signups;
begin
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

First write wins; second user sees error and refresh prompt.

### RLS policies

- `SELECT`: any authenticated user (calendar visibility).
- `INSERT`: authenticated users can insert only their own row. Calls go through `claim_shift`.
- `UPDATE`: not allowed.
- `DELETE`: only co-chairs (per config list).

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
  has_car         boolean not null default false,
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

- **Denormalized timestamps for each state, no separate transitions table.** Loses "who triggered" info but is simpler. Success metrics fully computable.
- **One open request per passenger,** enforced via partial unique index.
- **`shift_date` is stored explicitly,** computed by the app at submission time. Handles cross-midnight (1:30am Saturday request belongs to Friday's shift_date).
- **Pickup as required text + nullable lat/lng.** Captures coordinates when GPS is available.
- **`rider_count` capped at 4.** Sanity check.
- **`has_car` boolean.** When true, the passenger has a car at pickup. SafeRides drives both vehicles to the destination (one driver in passenger's car, the other in the SafeRides vehicle). Surfaced to drivers before claim. Defaults to false.
- **`status` as text + CHECK constraint** instead of Postgres enum, for migration flexibility.
- **`on delete cascade` from `profiles`.**

### Indexes

- Primary key (implicit)
- `(shift_date, status, requested_at)` for the queue query
- `(passenger_id)` for passenger history
- Partial unique index for one-open-request-per-passenger

### Required application logic

**Cross-midnight shift_date assignment:**
- 10pm to 11:59pm on a service night → `shift_date = today`
- 12am to 2am → `shift_date = yesterday`
- Otherwise → reject (outside service hours)

**Service hours enforcement.** New ride request submission rejected outside service hours. Existing live requests unaffected.

**Status transitions.** App writes new status and corresponding `*_at` timestamp atomically.

**Stale queued requests.** Queries filter by today's `shift_date`.

### RLS policies

- `SELECT`: passenger sees own requests; on-shift drivers see today's queue; co-chairs see everything.
- `INSERT`: any authenticated user can create a request for themselves.
- `UPDATE`: passenger can cancel their own request; on-shift driver can update status; co-chairs can update anything.
- `DELETE`: not allowed.

---

## Key application logic outside the schema

- **Magic link auth flow:** unified email entry, routes new users to profile setup, returning users to homepage.
- **Profile setup flow on first sign-in:** redirect to form, block features until done.
- **Co-chair role check:** lookup user email against config list.
- **Calendar generation:** Wed/Thu/Fri/Sat dates for next 8 weeks computed client-side, joined to `volunteer_signups`.
- **Cross-midnight shift_date computation.**
- **Service hours enforcement.**
- **Atomic shift claim** via `claim_shift` function.

## Future migration paths

- **Adding `co_chair_actions`:** Create the table; existing logic unchanged.
- **Adding `ride_state_transitions`:** Create the table; existing `*_at` columns can stay or be dropped.
- **Tuck SSO:** Swap auth provider; `profiles` structure unchanged.
- **Email domain restriction:** Application-side check on `signInWithOtp`; no schema change.
- **Live driver location:** `pickup_lat`/`pickup_lng` already exist; add `driver_lat`/`driver_lng`.
- **Service-area validation:** Add CHECK constraint or trigger.

## Open implementation questions for build phase

1. Exact RLS policy SQL (drafted as intent, needs concrete WHERE clauses).
2. Concrete implementation of `claim_shift` including error code conventions.
3. Whether to use Supabase realtime subscriptions or polling for queue updates.
4. Whether to write a separate `claim_ride` function for ride_requests (recommended).
