# ARENA · Product spec for the v1 build

This is the build brief. `index.html` is the working reference: every rule below is implemented there in plain JavaScript (see the `<script>` at the bottom, class `Component`). When this doc and the prototype disagree, ask before building either.

## 1. The product in one paragraph

ARENA is a hard-paywall iOS app for ambitious people who start too many things. Users pick at most 2 daily focus areas, set tiny daily "floors" they can clear on their worst day, and get nudged at the right moments of their real routine. When they slip, the app brings them back fast instead of guilting them. Target market: US, Canada, UK, Australia. Pricing: $39.99/yr or $7.99/mo, with a 7-day free trial.

## 2. Screens

| Area | Screens | Notes |
|---|---|---|
| Onboarding (9 steps) | Welcome, Identity (max 2), Goals (max 6), Tier sort, Your day (routine times), First floor (if-then), Savings floor + age, Nudge preview + permission, Paywall | Progress bar, back button, one primary button per screen |
| Trial success | "Day 1 starts now" | Goes to Today |
| Today tab | Header (date, idea button, streak, shields), ring + next nudge, floors list, Also today (tasks), Skipped spends, Close the day card | Primary daily screen |
| Plan tab | Week strip, day timeline (routine anchors, nudges, tasks), + add task | Edit a routine time for one day or every day |
| Progress tab | Coach (Pro), identity votes, streak stats, 12-week grids, 66-day Forge (Pro), Automatic, Savings | |
| You tab | Profile, Focus & idea inbox, Daily routine, Future Self (Pro), Partner, Weekly pair (soon), Nudges toggle, Widget, Subscription, Help, Stakes (soon) | Demo rows are prototype only, do not ship |
| Full-screen flows | Close the day (3 steps + done), Celebration, Comeback, Fresh start, Day 66 graduation, Sunday review + share card, Focus & idea inbox | All have an X back to Today |
| Sheets | Skipped spend, Reward, Proof, Paywall, Contacts, Partner, Future Self, Subscription, Help, Tier 2 check-in, Promote/idea decide, Park idea, Edit routine time, Add/edit task, Lunch check, Coming soon, Widget | Bottom sheets, tap outside to close |

## 3. Data model

```
User          id, name, age, identities[≤2], createdAt, timezone, plan, trialEndsAt, notifEnabled
Routine       userId, wake, lunch, clockOut, home, checkin           // minutes after midnight
RoutineOverride  userId, date, anchor, minutes                       // one-day changes from Plan
Goal          id, userId, name, tier (1|2|3), lastCheckIn, checkInResult
Floor         id, userId, goalId|null (null = core floor), anchor, whenText, action,
              identity, startedOn, rewardText, healthKitType|null, level
FloorLog      floorId, date, status (done|missed|shielded), source (tap|healthkit|close), proofText, proofPhotoUrl
Task          id, userId, date, minutes, title, remind, done
Idea          id, userId, name, parkedOn, status (cooling|ready|promoted|released)
SkipSpend     id, userId, date, label, amountCents
Transfer      id, userId, month, amountCents, done
DayClose      userId, date, note, closedAt
WeeklyReview  userId, weekStart, worked, slipped, oneThing
Slip          userId, date, reason (no_time|low_energy|forgot|life)
Partner       userId, contactName, contactPhone, nudgeAfterTwoMisses
Shield        userId, earnedOn, usedOn|null
Letter        userId, text, sealedOn
```

Store money in cents and times as minutes after midnight in the user's timezone.

## 4. Business rules

**Focus and tiers**
- Tier 1 (daily focus) holds max 2 goals. Promoting a third forces the user to pick one to move to Tier 2 ("Something has to move down.").
- Today shows floors whose goal is Tier 1, plus the core floor from onboarding.
- Tier 2 gets a weekly check-in (On track / Slipping / Park it). Park it moves to Tier 3.
- Tier 3 shows only a Promote button.

**Floors and streaks**
- A day counts when every visible floor is cleared. That increments the streak and shows the celebration once per day.
- Shields: earn 1 per week, hold max 1. A missed day uses a shield automatically, so the streak is kept and the grid shows a shield.
- Missed 1 day: on next open, show Comeback (reason chips, optional "make tomorrow's floor smaller", "I'm back").
- Missed 2+ days: show Fresh Start instead, which resets the streak to 0. Only lapsed users ever see it.
- Shrink map example: 30 min gym → 10 min walk. Keep a per-floor "lighter version" field.

**Identity votes**
- Each cleared floor = +1 vote for that floor's identity. Goal to identity mapping: Side business→Builder, Gym→Athlete, Save money→Saver, Guitar/Content→Creator, Reading→Learner. If the user didn't pick that identity, use their first identity. The core floor maps to Saver if picked.

**66-day Forge (Pro)**
- The label shows "Day N of 66" before day 33, then "X days to Automatic".
- At 66: graduation screen, then "Raise the floor?" Raising creates the next level (30 min → 45 min gym) at day 1 and adds the old one to Automatic.

**Routine and nudges** (all local notifications, no server needed)

| Nudge | Time | Content |
|---|---|---|
| Morning brief | wake + 30 min | Floor count and first floor |
| Before lunch | lunch − 60 min | Floors anchored to the morning |
| Lunch check | lunch + 30 min | "Did you eat?" plus the lunch floor. Actions: Yes (clear floor) / Not yet (snooze 30 min) |
| Clock-out floor | clockOut | Floors anchored to clock out |
| Home | home + 15 min | Floors anchored to home |
| Close the day | checkin (default home + 45) | Opens the Close the day flow |
| Task reminder | task time − 15 min | If remind is on |
| Trial ending | day 5 of the trial | Required by the paywall promise |

- A one-day routine change in Plan overrides that date only. "Every day" updates the routine and clears overrides for that anchor.
- Reschedule notifications whenever the routine, an override, tasks or floors change. iOS caps pending local notifications at 64, so schedule a rolling 7 days.

**Close the day**
1. Recap: floors, votes, skipped spends, tasks, plus open floors the user can tick late.
2. "What happened today?" (optional, chips).
3. Plan tomorrow: shows tomorrow's times with a link to change them, and adds tasks with a time.
4. Done, then mark the day closed.

**Idea inbox**
- Parked ideas cool off for 14 days, then become "ready" and trigger a notification.
- Decide: Tier 1 (with the demote rule), Tier 2, or Let it go.

**Money**
- Skipped spends are logged daily (presets plus a custom amount). The monthly transfer is a separate monthly floor.
- Future Self = monthly × ((1.01^n − 1) / 0.01) + saved × 1.01^n, where n = months to target age. Target age = 35 if under 35, else age + 10. Assumes 12%/yr compounded monthly. Show this assumption in the UI.

**Coach v1 (Pro)** is rules, not AI. Find a floor's worst weekday over the last 6 weeks (missed ≥ 4 of 6) and the most common slip reason. Offer 2 fixes: shrink that weekday's floor, or move it. The first insight appears on day 3 at the earliest.

**Partner**
- One contact. They get the Sunday card (shared via the iOS share sheet or SMS) and an optional nudge after 2 missed days. Cheers are one tap.
- A real two-way partner needs the partner to have the app. v1 can be SMS-only.

**Pro gating**
- Trial active or subscribed: everything unlocked.
- Free after trial: floors, streaks, shields, nudges, Plan, close the day and the idea inbox stay free.
- Locked: Coach, Forge, locked rewards, Future Self. Tapping a lock opens the paywall sheet.

## 5. Recommended stack

- **App:** Expo (React Native) + TypeScript, using a dev build (not Expo Go) because HealthKit and widgets need native code.
- **Local data:** SQLite (expo-sqlite) with Drizzle as the source of truth, so it works offline. Add sync later.
- **Backend:** Supabase (auth with Sign in with Apple, Postgres for backup and sync, storage for proof photos). Add it in v1.1 if speed matters more.
- **Notifications:** expo-notifications with local schedules and notification actions (Yes / Not yet on the lunch check).
- **Paywall and trial:** RevenueCat with a hard paywall at the end of onboarding, plus restore purchases and a trial reminder.
- **HealthKit:** react-native-health. Read workouts to auto-clear floors that have healthKitType set.
- **Widget:** WidgetKit extension (expo-apple-targets) showing the ring, streak and next floor, via an App Group shared store.
- **Analytics:** PostHog. Events: onboarding_step, trial_started, floor_cleared, day_completed, close_day_done, nudge_opened, comeback_shown, idea_parked, paywall_viewed, purchase.

## 6. Build order

1. **Week 1–2:** Onboarding, data model, Today (floors, ring, streak), RevenueCat paywall.
2. **Week 3:** Routine + local nudges, lunch check actions, Close the day flow, Plan tab with tasks and one-day overrides.
3. **Week 4:** Progress (grids, votes, Forge), comeback / fresh start / shields, idea inbox, skipped spends.
4. **Week 5:** HealthKit, widget, Sunday review + share card, Coach v1, TestFlight.
5. **Later:** weekly pair (needs a backend and matching), Stakes (needs payments and legal review), AI-written reflections.

## 7. Open decisions for the founders

- Is the account required at signup, or added after the paywall? (Recommendation: Sign in with Apple right after the paywall.)
- Should shields be earned by a perfect week or granted weekly? The prototype grants 1 per week.
- Partner v1: SMS-only, or require the partner to install the app?
- The prototype's copy and sample data (Abbu, Riya, the dollar amounts) are placeholders. Replace them before launch.
