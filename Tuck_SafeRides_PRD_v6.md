# Tuck SafeRides App: Product Requirements Document

**Author:** Sri
**Status:** Draft v6
**Last updated:** May 2026

**Changes from v5:**
- New ride request field: passenger indicates if they have a car at pickup (Section 7.4)
- Active ride status copy uses "SafeRides" brand voice instead of individual driver attribution (Section 7.6)
- Driver identity on active ride: both drivers shown (Section 7.6)
- Future email domain restriction noted (Section 7.1): post-MVP, restrict to `tuck.dartmouth.edu` and `dartmouth.edu`
- Notifications (push and SMS) confirmed out of MVP scope (Section 9)
- Settings: name and phone fields are mandatory; users cannot save the form with these blank (Section 7.1)

---

## 1. Overview

Tuck SafeRides is a volunteer-driven safe ride program serving Tuck students on Wednesday, Thursday, Friday, and Saturday nights. Today the program runs on a Google Sheet for volunteer signups, an email blast announcing each week's drivers and phone numbers, a GroupMe channel as backup roster, and direct calls or texts to coordinate pickups.

This document proposes a single web app that replaces both the volunteer scheduling layer and the rider-to-driver coordination layer with a structured queue and an in-app calendar, while keeping the volunteer-led nature of the program intact. Co-chairs continue to own recruitment, reminders, and any cancellations; the app handles the operational mechanics.

## 2. Problem statement

The current workflow has four meaningful frictions:

1. **Riders have to hunt for a driver.** A student needing a ride digs through old emails or scrolls GroupMe to find the night's volunteers, then calls or texts each one until someone picks up.
2. **Drivers have no shared view of demand.** No queue, no log of pickups, no record of completed rides.
3. **Volunteer scheduling lives in a Google Sheet.** Co-chairs maintain it manually, paste it into emails weekly, and chase signups by hand.
4. **Coordination data lives nowhere.** Pickup addresses are typed into texts, dropoffs are spoken on the phone. Co-chairs have no visibility into usage patterns, peak hours, or no-show rates.

The goal is not to replace the human element. It's to remove the lookup-and-dial step, give everyone a shared view of what's happening, replace the spreadsheet with self-service signup, and capture clean data for the co-chairs.

## 3. Goals and non-goals

### Goals

- Cut the time from "I need a ride" to "a driver is on the way" to under 60 seconds.
- Give drivers a single place to see, claim, and complete ride requests in FIFO order.
- Replace the Google Sheet volunteer schedule with an in-app self-signup calendar.
- Give co-chairs a clean view of who is signed up across the upcoming term and the ability to make corrections.
- Capture ride and shift data for co-chair visibility and future analytics.
- Keep the service strictly limited to verified Tuck community members (post-MVP, via Tuck SSO).

### Non-goals (v1)

- Replace co-chair recruitment, reminder emails, or cancellation handling (those stay manual and out-of-app or via co-chair admin only).
- Optimize multi-pickup routing.
- Charge for rides or process payments.
- Open the service to non-Tuck users.
- Compete with Uber or Lyft on feature parity.

## 4. Users and personas

### Persona 1: Student passenger ("Maya")
A first-year leaving Murphy's at 11:30pm on a Thursday. She wants a ride home, doesn't want to walk in the cold, and doesn't want to spend five minutes figuring out who tonight's drivers are.

**What she needs:** Open app, see who's driving tonight, enter destination, see roughly when the ride is coming.

### Persona 2: Student driver volunteer ("Ben")
A second-year who signed up via the calendar three weeks ago to drive Friday 10pm to 2am with a partner. He gets a reminder email from the co-chairs the week before. On the night, he's in his car with his partner, taking turns driving.

**What he needs:** See the queue when on shift, claim the next request, see pickup and dropoff before claiming, drive there, mark complete, move to the next.

### Persona 3: SafeRides co-chair ("Co-chair")
Manages the term's volunteer schedule, recruitment, and any cancellations.

**What she needs (v1):** A calendar view of all signups across the term, the ability to remove a signup if a volunteer drops out (this is the only cancellation channel), and a clean list of who is signed up for the upcoming week so she can send reminder emails outside the app.

## 5. Roles and responsibilities

This matrix makes the boundaries explicit so we don't accidentally scope-creep co-chair workflows into the app or vice versa.

| Role | In-app | Out-of-app |
|------|--------|------------|
| **Passenger** | Complete profile on first sign-in, edit profile via settings, request ride (indicating if they have a car), view queue position and driver info, cancel own ride request | Be at pickup location |
| **Driver volunteer** | Complete profile on first sign-in, edit profile via settings, sign up for shifts via calendar, view queue during shift, claim and progress ride requests, mark no-shows | Drive safely, respond to co-chair recruitment and reminder emails, contact a co-chair if needing to back out of a signup |
| **Co-chair** | Monitor calendar across 8-week horizon, remove volunteer signups (sole cancellation channel), view list of upcoming-week drivers with contact info, view unfilled upcoming shifts | Recruit volunteers via email, send confirmation when someone signs up, send weekly reminder emails to drivers signed up for the upcoming week, handle volunteer back-out requests, escalate when shifts are unfilled, manually delete account data 1 year post-graduation |

**Why volunteers cannot cancel their own signup:** Self-cancellation creates chaos at the operational layer (last-minute drops without communication, no replacement coordinated, passengers showing up to no service). Routing all back-outs through the co-chair guarantees there's a human checkpoint where recruitment of a replacement can begin immediately.

## 6. User stories

### Passenger
- As a passenger, I can sign in via magic link sent to my email.
- As a first-time user, I am guided through profile setup before I can use any other features.
- As a returning user, I land directly on the homepage after authentication.
- As a passenger, I can update my profile (name, phone, photo) via a settings page.
- As a passenger, I can see who's driving tonight on the homepage before I request a ride.
- As a passenger, I can request a ride by entering my pickup location (auto-detected or manual) and destination.
- As a passenger, I can indicate that I have a car at my pickup location, so SafeRides knows to drive both vehicles to my destination.
- As a passenger, I can add optional notes including rider count (up to 4) and special needs at request time.
- As a passenger, I cannot have more than one open ride request at a time.
- As a passenger, I can see my position in the queue.
- As a passenger, I can see who my driver pair is once assigned, including names, photos, and phone numbers.
- As a passenger, I can see ride status updates as the driver progresses.
- As a passenger, I can cancel my own ride request before being picked up.
- As a passenger, I can see when SafeRides is closed and know when it reopens.
- As a passenger, I see a clear empty-state message when no drivers are on shift during service hours.

### Driver volunteer
- As a volunteer, I can browse a calendar of upcoming Wed/Thu/Fri/Sat dates over the next 8 weeks.
- As a volunteer, I can sign up for an empty shift (becoming the first of two).
- As a volunteer, I can claim the open partner slot on a date someone else has signed up for.
- As a volunteer, if two of us try to claim the same partner slot at the same time, the first write wins and the second sees a clear error and refresh prompt.
- As a volunteer, I cannot cancel my own signup. If I need to back out, I contact a co-chair.
- As a volunteer on shift, I can see the FIFO queue of pending requests.
- As a volunteer, I can see passenger name, pickup, dropoff, rider count, has-car flag, and any notes before claiming.
- As a volunteer, I can claim the next request in the queue.
- As a volunteer, I can update ride status: en route, arrived, in vehicle, completed.
- As a volunteer, I can mark a passenger as a no-show after I've arrived.
- As a volunteer, I can decide whether to honor or cancel a request that was submitted just before service hours close.

### Co-chair
- As a co-chair, I can view the full 8-week calendar with all volunteer signups visible.
- As a co-chair, I can see a list of upcoming-week shifts with both volunteer names, emails, and phone numbers (so I can send reminder emails).
- As a co-chair, I can see which upcoming dates have unfilled slots (recruitment surface).
- As a co-chair, I can remove a volunteer's signup when they need to back out, which reopens the slot for recruitment.

## 7. Functional requirements

### 7.1 Authentication, profile setup, and settings

#### Authentication (MVP placeholder)

Magic link sent to the user's email. No password.

**Single entry flow:**
1. User enters email on the entry screen.
2. Magic link sent to email.
3. User clicks the link and is authenticated.
4. App routes based on profile state:
   - **No profile exists** → redirect to profile setup.
   - **Profile exists** → redirect to homepage.

There is no separate "sign up" vs "login" page. The same email field handles both.

**Future (v2):** Tuck SSO restricted to `tuck.dartmouth.edu` and any other domains Tuck IT specifies. Also planned for v2: even with the MVP placeholder magic-link auth, restrict accepted email domains to `tuck.dartmouth.edu` and `dartmouth.edu` (currently any email is accepted in MVP to support demo and testing).

**Co-chair role:** determined by a hardcoded list of emails in app config for MVP. Will move to proper role management when SSO lands.

**Default role:** all other users are passengers by default. Driver role is granted dynamically per-shift based on calendar signup, not as a persistent user attribute.

#### Profile setup (first sign-in)

After a first-time user authenticates, they are redirected to a profile completion form. All other features are blocked until the profile is complete.

Required fields: first name, last name, phone number.

Optional fields: photo upload. UI shows initials as fallback if no photo.

Once submitted, user is taken to the homepage.

#### Settings (profile editing)

A settings page accessible from the main navigation lets users update first name, last name, phone number, and photo.

Email is shown as read-only.

Name and phone are mandatory: the save action is blocked if either is blank. Users cannot remove these fields once set.

### 7.2 Service hours

- Wed 10pm to 12am
- Thu, Fri, Sat 10pm to 2am

**Submission cutoff:** New ride requests cannot be submitted after service hours close.

**Cross-midnight shift assignment:** A request submitted between 12am and 2am belongs to the prior night's shift. For example, a request at 1:30am on Saturday is part of the Friday shift, not Saturday. The app computes the correct `shift_date` at submission time.

**In-flight requests at close:** Any request submitted before close stays live in the system. Drivers exercise their own judgment on whether to honor it or cancel. There is no automatic timeout.

Calendar signup is always available, independent of service hours, including last-minute signups for the same night.

### 7.3 Volunteer signup calendar

- Calendar view of upcoming Wed/Thu/Fri/Sat dates over the next 8 weeks (term-aligned horizon).
- Each date displays its fill state:
  - **0/2:** Empty, available for signup.
  - **1/2:** First volunteer's name shown, partner slot open for claim.
  - **2/2:** Greyed out, both volunteers visible.
- Signup mechanic (Option A): solo signup with second slot opening for any other verified user to claim.
- **Volunteers cannot cancel their own signup.** All cancellations are handled by a co-chair on the volunteer's behalf via the admin view. This is enforced at both the UI layer (no cancel button) and the database layer (RLS policy blocks delete by non-co-chairs).
- **No minimum-horizon floor:** Volunteers can sign up for the same night up until shift start, supporting last-minute coverage.
- **Day-of-week constraint:** Only Wed/Thu/Fri/Sat dates are valid. Enforced at the database level via CHECK constraint.
- **Race condition policy on partner-claim:** If two volunteers tap "claim partner slot" simultaneously, the first write wins. The second user sees a clear error message and a refresh prompt to see updated state.
- Co-chair calendar view: all dates, all names, with delete button per signup. Deleting reopens the slot.
- Co-chair "this week" view: list of upcoming-week signups with name, email, phone (for reminder email composition).

### 7.4 Ride request (passenger)

Form fields:
- **Pickup location** (GPS auto-fill with manual override; no service-area validation in MVP).
- **Dropoff location** (text).
- **Rider count** (numeric, 1 to 4, default 1).
- **Has car at pickup** (boolean, default no). When the passenger indicates yes, drivers know to drive both vehicles to the destination: one drives the passenger's car while the other follows in the SafeRides vehicle. The pickup and route logic are unchanged.
- **Special needs / notes** (text, optional). Placeholder examples: "wheelchair access," "large bag," "two riders."

Submit creates a queued request with timestamp and computed shift_date.

**One open request per passenger:** A passenger can only have one active ride request at a time. They must cancel or complete the current request before submitting another. Enforced at the database layer.

Visible states to passenger: queued (with position), claimed (driver info revealed), en route, arrived, in vehicle, completed, cancelled, no-show.

### 7.5 Queue management (driver pair)

- Single FIFO queue per night, visible to both members of the active pair.
- Each queued request displays: passenger name, photo, pickup, dropoff, rider count, has-car flag, special needs notes, time in queue.
- Drivers see full pickup, dropoff, notes, and has-car flag before claiming.
- "Claim next" button assigns the top request. Driver can also claim a specific request out of order if needed.
- Once claimed, the request leaves the queue for both pair members and becomes the active ride.
- Single active ride at a time per pair (they're in one car).
- **Stale queued requests:** Requests submitted before service hours close that go unclaimed past close stay in the system but are filtered out of the queue view starting the next service night. They do not auto-cancel; they simply become invisible. Data persists for analytics.

### 7.6 Status states

Seven states, all visible to the passenger:

1. **Queued** (position shown)
2. **Claimed** (both driver names, photos, and phone numbers revealed)
3. **En route** (drivers are on the way)
4. **Arrived** (drivers are at pickup)
5. **In vehicle** (ride in progress)
6. **Completed** (terminal)
7. **Cancelled** or **No-show** (terminal)

**Brand voice on status copy:** Passenger-facing status messages use "SafeRides" as the actor rather than individual driver names, to position the program as the service. Examples: "SafeRides is on the way," "SafeRides is heading to you," "SafeRides is here. Please head out and meet [Driver 1] and [Driver 2]." The driver names appear on the driver card (which always shows both drivers in the pair), but the status sentence uses the SafeRides brand.

State transitions:
- Driver advances from claimed through completed.
- Passenger can cancel their own request any time before "in vehicle."
- Driver can mark no-show only after "arrived."
- Driver can cancel a request that came in just before service hours close at their discretion.

### 7.7 Empty state: no drivers on shift

- During service hours, if the date has no volunteer pair signed up, passenger ride request is disabled.
- Message: "No drivers on shift tonight. SafeRides will be back [next service night]."

### 7.8 Co-chair admin

- Read access across the full calendar.
- Delete any volunteer signup (this is the sole cancellation path; no soft-delete for MVP, no audit log in MVP).
- View "this week" list with contact info to support reminder emails.
- View "unfilled upcoming shifts" list to support recruitment.
- No analytics dashboard in MVP. Raw ride and signup records exportable to CSV via direct database access.

### 7.9 Tonight's drivers homepage card

The passenger homepage shows a "Tonight's drivers" card whenever a volunteer pair is signed up for the current service night. This mirrors the trust-building function of the current email blasts: passengers know which classmates are driving before they request.

**MVP content:**
- Names of both drivers
- Photos of both drivers (from their profiles)
- Date and service hours for the night

**Card visibility:**
- Shown during service hours when a volunteer pair is signed up.
- Replaced by the empty-state message (Section 7.7) when no pair is signed up.
- Hidden outside service hours (or shown with the next-open-night info).

**Future expansion (post-MVP):** Drivers can write a short welcome message that appears on the card. The current SafeRides email blasts have a recognizable cultural voice (humor, themes, inside jokes) worth preserving as the program moves into the app. Implementation details (length limits, moderation, optional vs. required) deferred to a focused design discussion.

## 8. Non-functional requirements

- **Service area:** Hanover, Lebanon NH, Norwich VT, and the broader Upper Valley. No hard validation in MVP; users are trusted to enter sensible locations.
- **Privacy:** Contact details revealed only after a request is matched. No driver-side viewing of passenger phone or photo until claim. Driver names and photos visible on the homepage card by virtue of their volunteer signup are intentional and not considered private (consistent with the current email blast).
- **Reliability:** Must work on cellular coverage typical for the Upper Valley.
- **Performance:** Queue and status updates propagate within 5 seconds via in-app realtime subscriptions.
- **Auditability:** All ride records (request, claim, state transitions, completion or terminal state) timestamped and persisted. Volunteer signup events similarly logged via the `created_at` column.
- **Graceful degradation:** Phone numbers always exposed once matched, so the app can fall back to call or text if anything in-app fails.
- **Data retention:** User profile and ride data retained for 1 year past the user's MBA graduation. Co-chair manually deletes accounts after that window. Auth identity deletion cascades to profile and all ride records.
- **Security controls:**
  - Volunteer self-deletion of signups blocked at the database layer via Row Level Security (RLS) policy. UI omits the delete control; RLS prevents direct API calls from bypassing it.
  - One-open-ride-request-per-passenger enforced via database partial unique index.
  - Co-chair role determined exclusively by hardcoded email config (no client-side privilege escalation possible).

## 9. Out of scope for v1

- Tuck SSO (deferred until IT engagement)
- Email domain restriction (currently any email accepted; restriction to `tuck.dartmouth.edu` and `dartmouth.edu` planned for v2)
- Live driver location on a map
- **Push notifications (web push) and SMS notifications.** In-app realtime handles updates when the app is open; phone numbers serve as the fallback for closed-app scenarios. Drivers on shift are expected to keep the app open during their 2-4 hour window.
- Automated weekly reminder emails (co-chairs handle these manually for MVP)
- Multi-stop route optimization
- Ratings or feedback
- Analytics dashboard
- Native iOS or Android apps (web app is sufficient)
- Service-area address validation
- Volunteer self-cancellation
- Driver-authored welcome content on the "Tonight's drivers" homepage card (deferred to focused design discussion post-MVP)
- Passenger ride history view (deferred to v2)
- Co-chair audit log of admin actions (`co_chair_actions` table; deferred to v2)
- Account deletion in-app (handled manually by co-chairs per 1-year retention policy)
- Automatic deletion of accounts after the retention window

## 10. Success metrics

- **Adoption:** % of weekly SafeRides requests that come through the app vs. text or call within first month of launch.
- **Speed:** Median time from request submitted to request claimed.
- **Completion:** % of requests that reach the "completed" state vs. cancelled or no-show.
- **Volunteer scheduling friction:** % of weekly shifts that fill via in-app signup without co-chair intervention.
- **Volunteer satisfaction:** Co-chair and driver survey at end of first month.

## 11. Open questions

Resolved in earlier versions: cutoff for 1/2-filled shifts, service hours close, special needs field, volunteer signup window, "Tonight's drivers" homepage card, volunteer self-cancellation, race condition policy, rider count cap, notifications scope, has-car field, driver identity on active ride.

One open item carried forward:

1. **Volunteer no-show on shift night.** What happens if a volunteer signs up but doesn't show up the night-of? Driver pair becomes a single driver, or no service at all. Co-chair likely won't know in real time. For v1, recommend: trust the volunteer system, log the no-show against that volunteer's record (co-chair-visible), handle social repercussions out-of-band.

## 12. MVP scope

### In MVP

1. **Magic-link auth** with unified entry flow. Email-based, any domain for now.
2. **Profile setup** flow on first sign-in (first name, last name, phone, optional photo). Blocks other features until complete.
3. **Settings page** for profile editing. Name and phone mandatory; cannot save blank.
4. **Co-chair role via hardcoded email list** in app config.
5. **Volunteer self-signup calendar.** 8-week horizon. Solo signup with partner-claim mechanic. No self-cancellation. Last-minute signups allowed. Greyed-out fully-filled dates. Race condition handled with first-write-wins.
6. **Co-chair calendar oversight.** Read all, delete any signup, view this-week list with contacts, view unfilled upcoming shifts.
7. **Passenger ride request.** Pickup (GPS or manual), dropoff (text), rider count (1 to 4), has-car flag, optional special needs notes. One open request per passenger.
8. **FIFO shared queue** for the night's pair. Both pair members see the same queue.
9. **Drivers see full pickup, dropoff, notes, and has-car flag** before claiming.
10. **Claim flow** with match reveal: passenger sees both driver names, photos, phones. Drivers see passenger name, phone, pickup, dropoff, notes.
11. **Seven status states:** claimed, en route, arrived, in vehicle, completed, cancelled, no-show. Brand voice copy uses "SafeRides" as actor.
12. **Service hours enforcement** with submission cutoff, cross-midnight shift assignment, and in-flight request behavior.
13. **"No drivers tonight" empty state.**
14. **"Tonight's drivers" homepage card** showing names, photos, date, hours.
15. **Stale queued requests filtered from active queue** automatically the next service night.
16. **All records persisted:** rides, signups, state transitions via timestamp columns.

### Explicitly out of MVP

- Tuck SSO and email domain restriction
- Live driver location map
- Push and SMS notifications
- Automated reminder emails
- Analytics dashboard
- Service-area validation
- Multi-stop optimization
- Ratings
- Volunteer self-cancellation
- Driver-authored welcome content on the homepage card
- Passenger ride history view
- Co-chair audit log
- In-app account deletion
- Automatic account deletion after retention window

## 13. Tech approach

**Stack:** Next.js + Tailwind on the frontend, Supabase for auth (magic link), database, and real-time subscriptions for queue updates. Deploy on Vercel.

**Database schema:** See companion document `Tuck_SafeRides_Schema.md` for the full schema specification.

**Screen specs:** See companion document `Tuck_SafeRides_Screen_Specs.md` for detailed prose specs of every screen.

**Why this stack:**
- Magic link auth is a Supabase primitive, zero custom code.
- Real-time subscriptions handle queue updates without building websocket infra.
- Next.js gives you both the marketing/landing page and the app in one codebase.
- Vercel deploys are free and fast.
- Clean migration path to Tuck SSO later (Supabase supports SAML and OIDC).

**Why web, not Expo:**
- No app store distribution required (text a link, install as PWA).
- Faster iteration cycle.
- Fully responsive on phone, which is where this will mostly be used.

**Tables explicitly not built in MVP** (captured in schema doc):
- `ride_state_transitions` collapsed to denormalized `*_at` timestamp columns on `ride_requests`. Adding it later requires backfill but no breaking changes.
- `co_chair_actions` audit log table. Trivial to add later.

## 14. Build sequence (rough)

Build is now a two-person effort. Recommended ownership split should be decided at tech-spec time.

- **Week 1:** Auth, profile setup, settings page, data model, basic shell.
- **Week 2:** Volunteer signup calendar, co-chair oversight view, `claim_shift` Postgres function.
- **Week 3:** Ride request flow (including has-car field), queue, claim, status states, "Tonight's drivers" homepage card.
- **Week 4:** Service hours, empty states, polish, soft launch with one weekend of testing.

## 15. Pilot plan

For the soft launch:
- Recruit current SafeRides co-chairs as design partners.
- Run one weekend in parallel with the existing Google Sheet + email + GroupMe workflow.
- Collect feedback from drivers and any passengers who used the app.
- Iterate before formal cutover.
