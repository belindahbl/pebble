# Product Requirements Document — Personal Holistic Health App

**Version:** 1.1 (v1 scope)
**Status:** Draft for build

## Related documents
- `ARCHITECTURE.md` — platform, storage, iCloud sync, offline, responsive.
- `DESIGN_SYSTEM.md` — colors, typography, navigation, components, dark mode.
- `DATA_MODEL.md` — schema for every module.
- `STRENGTH_SCIENCE.md` — training modes, parameters, cycle-aware deload/load logic.
- `CYCLE_AND_TCM.md` — phase prediction and per-phase training/nutrition suggestions.
- `MIGRAINE_LOGIC.md` — migraine logging fields and correlation rules.
- `BUILD_PLAN.md` — phased build sequence.

---

## 1. Purpose

A single-user app to track strength training, other training (e.g. jiu-jitsu), the menstrual cycle, and migraines **together**, so that cross-domain patterns — cycle phase × training load × hydration × migraines — become visible and actionable. No app on the market connects these domains; that connection is the reason this exists.

The app should reduce migraines and period-symptom burden over time by helping the user (a) train appropriately for their current cycle phase and training load, and (b) discover their own migraine triggers from their own data.

## 2. Users

One user. No accounts, no login, no multi-user support, no sharing. Single-user is a deliberate constraint and is a **non-goal** to change in v1.

## 3. Platform

Single web app opened in the browser and added to the home screen, fully offline after first load, running on both phone and laptop from one codebase with a responsive layout. Data lives locally on each device and is kept in sync across the user's phone and laptop via a data file in the user's own iCloud Drive (server-free, local-first). See `ARCHITECTURE.md`.

## 4. Data & privacy

All data is stored locally on the device (IndexedDB) and, for cross-device use, in a file in the user's own iCloud Drive. No backend, no third-party server, no accounts. Data stays in the user's own devices and iCloud account. Privacy is a core product value and takes precedence over convenience. See `ARCHITECTURE.md`.

## 5. Modules (in priority order)

### 5.1 Strength + cycle-aware programming *(Priority 1)*
- Custom exercise library, including unconventional movements not found in mainstream apps. The user defines their own exercises.
- **Workout planning with named splits** (e.g. Split A / B / C): reusable workout templates the user builds, schedulable onto future dates.
- Workout logging matching the user's programming style.
- Science-backed training **modes** (e.g. hypertrophy, strength, contrast/complex methods), each carrying parameters for rep ranges, intensity, proximity to failure, rest, and weekly volume.
- **Cycle-aware and load-aware auto-suggestion:** suggests lifting lighter (deload) or heavier based on current cycle phase and recent jiu-jitsu / other training intensity, applied to the planned session for the day. Suggestions auto-populate, can always be overridden, and show their reasoning.
- Prescription is driven primarily by the user's own logged performance (actual or estimated 1RM per lift), not population formulas.
- Modes, parameters, and deload/load logic are defined in `STRENGTH_SCIENCE.md`. Split/plan/log structure and schema in `DATA_MODEL.md`.

### 5.2 Period tracker + suggestions *(Priority 2)*
- Cycle logging and **phase prediction**: rule-based initially, structured to learn the user's pattern over time.
- Per-phase guidance on **how to train** and **how to eat** (e.g. anti-inflammatory, warming foods), drawing on evidence-based sources and Traditional Chinese Medicine.
- Prediction and suggestion content defined in `CYCLE_AND_TCM.md`. Schema in `DATA_MODEL.md`.

### 5.3 Migraine tracker + trigger discovery *(Priority 3)*
- Fast logging: onset, severity, duration, **symptoms** (including nausea and vomiting), and **medications taken** (name, dose, timing, and whether it helped).
- Context capture linking each migraine to hydration, cycle phase, and training load.
- **Trigger discovery** surfaces correlations between migraines and hydration, cycle phase, training load, and daily wellness inputs.
- Presents **associations, not causes**. Logging fields and correlation rules defined in `MIGRAINE_LOGIC.md`. Schema in `DATA_MODEL.md`.

### 5.4 Jiu-jitsu / other workouts *(Priority 4)*
- Session logging with a training-**intensity** signal that feeds back into the strength deload/load logic (§5.1, `STRENGTH_SCIENCE.md`).

## 6. Cross-cutting features (from day one)

- **Hydration logging** — central to the migraine hypothesis.
- **Daily wellness inputs** — quick 1–5 scales for sleep, stress, energy, and soreness.
- **Dashboard (home)** — the landing screen; a glanceable summary across all modules with a persistent quick-add control. See §7 for navigation and the dashboard card set.
- **Calendar overview** — a month view with per-day markers (cycle phase, workout, migraine, hydration); tapping a day opens and adds entries. Primary way the user moves through time.

## 7. Navigation & information architecture

The app uses a single fixed set of five top-level destinations — a bottom tab bar on phone, a left sidebar on laptop (see `DESIGN_SYSTEM.md`). Tapping a destination swaps the content area only; there is no second layer of tab bars. Depth within a destination is handled by pushed screens (with back) or sheets, never nested tabs.

The five destinations:
1. **Home** — dashboard summary + quick-add; the default screen.
2. **Calendar** — month spine; tap a day to view/add.
3. **Train** — splits (A/B/C), workout planning, exercise library, and logging a session.
4. **Insights** — deeper analysis: PR history, strength growth trends, migraine trigger correlations.
5. **Cycle** — period tracking and phase guidance (training + nutrition).

**Home dashboard cards** (each glanceable, tapping opens the relevant destination):
- **Cycle status** — current phase and cycle day, next period prediction.
- **Today's training** — the planned split for today with the cycle/load suggestion applied, or a prompt to plan/log.
- **Strength highlight** — a recent PR or a short progress trend.
- **Migraine watch** — days since last migraine and any active risk flag from correlations.
- **Hydration** — today's intake with one-tap add.
- **Wellness** — today's sleep/stress/energy/soreness, or a prompt to log.
- **Quick-add** — a persistent control to log any entry (including a migraine) in one tap, preserving log-first speed from the dashboard.

Logging is reachable from Home quick-add and from Calendar day entries; analysis of that data lives in Insights. Logging and analysis are deliberately separated.

## 8. Product principles

- **Log-first design.** Logging is fast and always one tap away via the dashboard quick-add and the calendar, even though the landing screen is a summary dashboard.
- **Fixed, small navigation.** Five destinations maximum; actions are local to where the user is rather than scattered as global buttons. This is the primary defense against UI clutter.
- **Correlation, not causation.** Trigger discovery surfaces associations from a personal dataset and never asserts cause.
- **No medical-diagnosis claims.** The app supports self-understanding; it does not diagnose or replace professional care.

## 9. Non-goals (v1)

No backend or app-operated server; no accounts or authentication; no silent automatic multi-device sync (sync is via user-initiated iCloud file save/load); no social or sharing features; no medical-diagnosis claims; no wearable/device integrations; no notifications or reminders.

## 10. Designed-for but deferred

Out of v1 scope, but the architecture and data model must not preclude them:
- Notifications/reminders (e.g. hydration prompts, log reminders).
- Silent automatic multi-device sync — see `ARCHITECTURE.md`.
- Pattern-learning cycle prediction — see `CYCLE_AND_TCM.md`.
