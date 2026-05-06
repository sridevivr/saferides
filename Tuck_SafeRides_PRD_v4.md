# Tuck SafeRides App: Product Requirements Document

**Author:** Sri
**Status:** Draft v4
**Last updated:** May 2026

**Changes from v3:**
- "Tonight's drivers" homepage card confirmed as MVP feature (Section 7.9, new)
- Driver-authored welcome content (capturing the personality of the current email blasts) explicitly deferred to post-MVP

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
| **Passenger** | Request ride, view queue position and driver info, cancel own ride request | Be at pickup location |
| **Driver volunteer** | Sign up for shifts via calendar, view queue during shift, claim and progress ride requests, mark no-shows | Drive safely, respond to co-chair recruitment and reminder emails, contact a co-chair if needing to back out of a signup |
| **Co-chair** | Monitor calendar across 8-week horizon, remove volunteer signups (sole cancellation channel), view list of upcoming-week drivers with contact info, view unfilled upcoming shifts | Recruit volunteers via email, send confirmation when someone signs up, send weekly reminder emails to drivers signed up for the upcoming week, handle volunteer back-out requests, escalate when shifts are unfilled |

**Why volunteers cannot cancel their own signup:** Self-cancellation creates chaos at the operational layer (last-minute drops without communication, no replacement coordinated, passengers showing up to no service). Routing all back-outs through the co-chair guarantees there's a human checkpoint where recruitment of a replacement can begin immediately.

## 6. User stories

### Passenger
- As a passenger, I can sign in via magic link sent to my email.
- As a passenger, I can see who's driving tonight on the homepage before I request a ride.
- As a passenger, I can request a ride by entering my pickup location (auto-detected or manual) and destination.
- As a passenger, I can add optional notes including rider count and special needs at request time.
- As a passenger, I can see my position in the queue.
- As a passenger, I can see who my driver is once assigned, including name, photo, and phone number.
- As a passenger, I can see ride status updates as the driver progresses.
- As a passenger, I can cancel my own ride request before being picked up.
- As a passenger, I can see when SafeRides is closed and know when it reopens.
- As a passenger, I see a clear empty-state message when no drivers are on shift during service hours.

### Driver volunteer
- As a volunteer, I can browse a calendar of upcoming Wed/Thu/Fri/Sat dates over the next 8 weeks.
- As a volunteer, I can sign up for an empty shift (becoming the first of two).
- As a volunteer, I can claim the open partner slot on a date someone else has signed up for.
- As a volunteer, I cannot cancel my own signup. If I need to back out, I contact a co-chair.
- As a volunteer on shift, I can see the FIFO queue of pending requests.
- As a volunteer, I can see passenger name, pickup, dropoff, and any notes (including special needs) before claiming.
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

### 7.1 Authentication and identity

**MVP placeholder:** Magic link sent to any email address. No password. Profile created on first sign-in.

**Future (v2):** Tuck SSO restricted to @tuck.dartmouth.edu and any other domains Tuck IT specifies.

- Profile fields: name, photo, phone number.
- Co-chair role: determined by hardcoded list of emails in app config for MVP. Will move to proper role management when SSO lands.
- All other users: passenger by default. Driver role is granted dynamically per-shift based on calendar signup, not as a persistent user attribute.

### 7.2 Service hours

- Wed 10pm to 12am
- Thu, Fri, Sat 10pm to 2am
- **Submission cutoff:** New ride requests cannot be submitted after service hours close.
- **In-flight requests at close:** Any request submitted before close stays live in the system. Drivers exercise their own judgment on whether to honor it or cancel. There is no automatic timeout.
- Calendar signup is always available, independent of service hours, including last-minute signups for the same night.

### 7.3 Volunteer signup calendar

- Calendar view of upcoming Wed/Thu/Fri/Sat dates over the next 8 weeks (term-aligned horizon).
- Each date displays its fill state:
  - **0/2:** Empty, available for signup.
  - **1/2:** First volunteer's name shown, partner slot open for claim.
  - **2/2:** Greyed out, both volunteers visible.
- Signup mechanic (Option A): solo signup with second slot opening for any other verified user to claim.
- **Volunteers cannot cancel their own signup.** All cancellations are handled by a co-chair on the volunteer's behalf via the admin view.
- **No minimum-horizon floor:** Volunteers can sign up for the same night up until shift start, supporting last-minute coverage.
- Co-chair calendar view: all dates, all names, with delete button per signup. Deleting reopens the slot.
- Co-chair "this week" view: list of upcoming-week signups with name, email, phone (for reminder email composition).

### 7.4 Ride request (passenger)

- Form fields:
  - **Pickup location** (GPS auto-fill with manual override; no service-area validation in MVP).
  - **Dropoff location** (text).
  - **Rider count** (numeric, optional, default 1).
  - **Special needs / notes** (text, optional). Placeholder examples: "wheelchair access," "large bag," "two riders."
- Submit creates a queued request with timestamp.
- Visible states to passenger: queued (with position), claimed (driver info revealed), en route, arrived, in vehicle, completed, cancelled, no-show.

### 7.5 Queue management (driver pair)

- Single FIFO queue per night, visible to both members of the active pair.
- Each queued request displays: passenger name, photo, pickup, dropoff, rider count, special needs notes, time in queue.
- Drivers see full pickup, dropoff, and notes before claiming.
- "Claim next" button assigns the top request. Driver can also claim a specific request out of order if needed.
- Once claimed, the request leaves the queue for both pair members and becomes the active ride.
- Single active ride at a time per pair (they're in one car).

### 7.6 Status states

Seven states, all visible to the passenger:

1. **Queued** (position shown)
2. **Claimed** (driver name, photo, phone revealed)
3. **En route** (driver is on the way)
4. **Arrived** (driver is at pickup)
5. **In vehicle** (ride in progress)
6. **Completed** (terminal)
7. **Cancelled** or **No-show** (terminal)

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
- Delete any volunteer signup (this is the sole cancellation path; no soft-delete for MVP).
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
- Shown during service hours when a volunteer pair is signed up
- Replaced by the empty-state message (Section 7.7) when no pair is signed up
- Hidden outside service hours (or shown with the next-open-night info)

**Future expansion (post-MVP):** Drivers can write a short welcome message that appears on the card. The current SafeRides email blasts have a recognizable cultural voice (humor, themes, inside jokes) worth preserving as the program moves into the app. Implementation details (length limits, moderation, optional vs. required) deferred to a focused design discussion.

## 8. Non-functional requirements

- **Service area:** Hanover, Lebanon NH, Norwich VT, and the broader Upper Valley. No hard validation in MVP; users are trusted to enter sensible locations.
- **Privacy:** Contact details revealed only after a request is matched. No driver-side viewing of passenger phone or photo until claim. Driver names and photos visible on the homepage card by virtue of their volunteer signup are intentional and not considered private (consistent with the current email blast).
- **Reliability:** Must work on cellular coverage typical for the Upper Valley.
- **Performance:** Queue and status updates propagate within 5 seconds.
- **Auditability:** All ride records (request, claim, state transitions, completion or terminal state) timestamped and persisted. Volunteer signup events similarly logged. Co-chair deletions logged with co-chair identity.
- **Graceful degradation:** Phone numbers always exposed once matched, so the app can fall back to call or text if anything in-app fails.

## 9. Out of scope for v1

- Tuck SSO (deferred until IT engagement)
- Live driver location on a map
- Push notifications (use in-app polling; phone numbers as fallback)
- Automated weekly reminder emails (co-chairs handle these manually for MVP)
- Multi-stop route optimization
- Ratings or feedback
- Analytics dashboard
- Native iOS or Android apps (web app is sufficient)
- Service-area address validation
- Volunteer self-cancellation
- Driver-authored welcome content on the "Tonight's drivers" homepage card (deferred to focused design discussion post-MVP)

## 10. Success metrics

- **Adoption:** % of weekly SafeRides requests that come through the app vs. text or call within first month of launch.
- **Speed:** Median time from request submitted to request claimed.
- **Completion:** % of requests that reach the "completed" state vs. cancelled or no-show.
- **Volunteer scheduling friction:** % of weekly shifts that fill via in-app signup without co-chair intervention.
- **Volunteer satisfaction:** Co-chair and driver survey at end of first month.

## 11. Open questions

Resolved in earlier versions: cutoff for 1/2-filled shifts, service hours close, special needs field, volunteer signup window, "Tonight's drivers" homepage card.

One open item carried forward:

1. **Volunteer no-show on shift night.** What happens if a volunteer signs up but doesn't show up the night-of? Driver pair becomes a single driver, or no service at all. Co-chair likely won't know in real time. For v1, recommend: trust the volunteer system, log the no-show against that volunteer's record (co-chair-visible), handle social repercussions out-of-band.

## 12. MVP scope

### In MVP

1. **Magic-link auth.** Email-based, any domain for now. Profile setup (name, photo, phone) on first sign-in.
2. **Co-chair role via hardcoded email list** in app config.
3. **Volunteer self-signup calendar.** 8-week horizon. Solo signup with partner-claim mechanic. No self-cancellation. Last-minute signups allowed. Greyed-out fully-filled dates.
4. **Co-chair calendar oversight.** Read all, delete any signup, view this-week list with contacts, view unfilled upcoming shifts.
5. **Passenger ride request.** Pickup (GPS or manual), dropoff (text), rider count, optional special needs notes.
6. **FIFO shared queue** for the night's pair. Both pair members see the same queue.
7. **Drivers see full pickup, dropoff, and notes** before claiming.
8. **Claim flow** with match reveal: passenger sees driver name, photo, phone. Driver sees passenger name, phone, pickup, dropoff, notes.
9. **Seven status states:** claimed, en route, arrived, in vehicle, completed, cancelled, no-show.
10. **Service hours enforcement** with submission cutoff and in-flight request behavior.
11. **"No drivers tonight" empty state.**
12. **"Tonight's drivers" homepage card** showing names, photos, date, hours.
13. **All records persisted:** rides, signups, state transitions, co-chair deletions.

### Explicitly out of MVP

- Tuck SSO
- Live driver location map
- Push notifications
- Automated reminder emails
- Analytics dashboard
- Service-area validation
- Multi-stop optimization
- Ratings
- Volunteer self-cancellation
- Driver-authored welcome content on the homepage card

## 13. Tech approach

**Stack:** Next.js + Tailwind on the frontend, Supabase for auth (magic link), database, and real-time subscriptions for queue updates. Deploy on Vercel.

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

## 14. Build sequence (rough)

If building solo with AI assistance:

- **Week 1:** Auth, data model, profile setup, basic shell.
- **Week 2:** Volunteer signup calendar (the trickiest UI), co-chair oversight view.
- **Week 3:** Ride request flow, queue, claim, status states, "Tonight's drivers" homepage card.
- **Week 4:** Service hours, empty states, polish, soft launch with one weekend of testing.

This assumes evenings-and-weekends pace. Could compress to 3 weeks if focused full-time.

## 15. Pilot plan

For the soft launch:
- Recruit current SafeRides co-chairs as design partners.
- Run one weekend in parallel with the existing Google Sheet + email + GroupMe workflow.
- Collect feedback from drivers and any passengers who used the app.
- Iterate before formal cutover.
