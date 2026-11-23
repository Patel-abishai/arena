# ARENA

**Stop starting. Start finishing.**

ARENA is a self-improvement app for ambitious people who start too many things. You pick at most two things that matter, set small daily floors you can clear on your worst day, get nudged around your real routine, and get back on track fast after a slip.

<p>
<img src="screenshots/03-today.png" width="200" alt="Today">
<img src="screenshots/04-plan.png" width="200" alt="Plan">
<img src="screenshots/05-close-the-day.png" width="200" alt="Close the day">
<img src="screenshots/06-progress.png" width="200" alt="Progress">
</p>

## Try the prototype

Open `index.html` in any browser. No install, no internet needed. Everything is clickable.

- Start somewhere specific: `index.html?start=today` (also `plan`, `progress`, `you`, `focus`, `close`, `comeback`, `fresh`, `review`, `graduate`, `paywall`, `trialEnded`).
- In the app, the **You** tab has demo controls to jump between states and to fire notifications (morning brief, lunch check, close the day, Apple Health workout, partner cheer, idea cooled off).
- To share it as a link, turn on GitHub Pages: repo Settings → Pages → Deploy from branch `main` / root. It will be live at `https://patel-abishai.github.io/arena/`.

## What's in this repo

| File | What it is |
|---|---|
| `index.html` | The full clickable prototype as one self-contained page (Preact inlined). This is the reference for the build. |
| `SPEC.md` | Build brief for engineering: screens, data model, every business rule, nudge schedule, Pro gating, recommended stack, build order. |
| `design/ARENA.dc.html` | Source of the prototype in Claude Design format (runs in the ARENA design canvas). |
| `design/canvas.json` | Canvas layout for the design file. |
| `screenshots/` | Key screens. |

## The core loop

1. **Onboarding:** identity (max 2), goals (max 6), tier sort (max 2 in daily focus), your daily routine, first if-then floor, savings floor, nudge preview, 7-day trial paywall ($39.99/yr or $7.99/mo).
2. **Today:** clear floors, each clear is a vote for who you're becoming. Log skipped spends.
3. **Nudges:** timed around your routine: morning brief, before lunch, did-you-eat check, clock-out floor, close the day.
4. **Close the day:** recap, one line on how it went, plan tomorrow.
5. **Plan:** a simple calendar. Change any routine time for one day or every day, add tasks with reminders.
6. **Slip:** a shield protects the streak, and Comeback makes tomorrow's floor smaller. After 2+ missed days, Fresh Start.
7. **Grow:** the 66-day Forge turns floors automatic, then you raise the floor. Coach spots patterns in your misses.
8. **Stay focused:** new ideas wait 14 days in the idea inbox before they can take a spot.

<p>
<img src="screenshots/01-welcome.png" width="160" alt="Welcome">
<img src="screenshots/02-paywall.png" width="160" alt="Paywall">
<img src="screenshots/07-focus-ideas.png" width="160" alt="Focus and idea inbox">
<img src="screenshots/08-comeback.png" width="160" alt="Comeback">
<img src="screenshots/09-lunch-nudge.png" width="160" alt="Lunch nudge">
</p>

## Coming soon

- **Weekly pair:** each week you're matched with one person who also has goals, similar or different, to check in with each other.
- **Stakes:** money on a floor that goes to a charity you pick if you miss.

## Status

Design prototype, v1.1. The real app hasn't been built yet. Start with `SPEC.md`.
