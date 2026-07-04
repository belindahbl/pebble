# Migraine Logic — Personal Holistic Health App

**Version:** 1.0 (v1 scope)

## Related documents
- `PRD.md` — module 5.3 (migraine tracker + trigger discovery), the "correlation not causation" and "no medical-diagnosis" principles this file is bound by.
- `DATA_MODEL.md` — the `MigraineEntry` entity (§10.2) this file expands, and the shared `context` shape (§6.5) that makes cross-domain correlation possible.
- `DESIGN_SYSTEM.md` — the single-hue severity scale (`--severity-1..3`) and the chip/tag component the log uses.
- `CYCLE_AND_TCM.md` — the source of `cyclePhase` captured in each migraine's context.
- `STRENGTH_SCIENCE.md` — the source of `precedingTrainingLoad` (the sRPE-based load figure) captured in context.
- `ARCHITECTURE.md` — offline, local-first; all logging and all correlation runs on-device with no network.

---

## 1. Purpose and scope

This file owns two things:

1. **Migraine logging** — the fields captured per migraine, and how logging stays fast (log-first, `PRD.md` §8).
2. **Trigger discovery** — the correlation rules that surface associations between migraines and hydration, cycle phase, training load, and daily wellness inputs.

**What this file is bound by.** The app presents **associations, never causes** (`PRD.md` §8). It makes **no medical-diagnosis claims** and does not replace professional care. Trigger discovery is a personal-pattern tool over the user's own dataset — it can say "migraines tend to cluster on low-hydration days," never "dehydration causes your migraines." Every correlation output is framed on the safe side of that line (§5).

---

## 2. Logging philosophy — fast, because migraines are logged while unwell

The app is often opened mid-migraine, when the user feels awful and screen time hurts (`DESIGN_SYSTEM.md` §1). So migraine logging is the most speed-critical flow in the app:

- **One entry, minimal taps.** Onset + severity is enough to save a usable record; everything else is optional and can be added later (e.g. duration once it ends, whether a med helped once it's had time to work).
- **Sensible defaults everywhere.** Onset defaults to now; medications are pre-loaded with the user's actual meds and doses (§4).
- **Editable after the fact.** A migraine is often logged in two passes — a quick one at onset, a finishing one once it resolves. `durationMinutes`, `endTime`, and medication `helped` are commonly filled on the second pass.

---

## 3. `MigraineEntry` — full field list

Expands the stub in `DATA_MODEL.md` §10.2. One IndexedDB object store; standard `id` / `createdAt` / `updatedAt` / `deletedAt` conventions apply.

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key (UUID). |
| `date` | date | The day of the migraine (`YYYY-MM-DD`). Derived from `onsetTime` but stored for fast day-bucketing. |
| `onsetTime` | timestamp | When it started. Defaults to now; editable. The one required field beyond severity. |
| `endTime` | timestamp \| null | When it resolved. Null while ongoing / if never recorded. |
| `durationMinutes` | int \| null | Duration. Auto-computed from `onsetTime`→`endTime` if both present; can be entered directly. |
| `severity` | int | 1–3 on the single-hue scale (`DESIGN_SYSTEM.md`): 1 mild, 2 moderate, 3 severe. Required. |
| `symptoms` | string[] | From the checklist (§3.1). Includes nausea and vomiting. |
| `medications` | object[] | Each `{ name, dose, unit, count, timing, helped }` (§4). |
| `context` | object | Auto-captured cross-domain context (§3.2). The hook for trigger discovery. |
| `note` | string \| null | Optional free text (e.g. "started after skipped lunch"). |

### 3.1 Symptom checklist

Multi-select chips. Seeded list, user-extendable (mirrors how the exercise library is extendable). Starting set:

`nausea`, `vomiting`, `light sensitivity`, `sound sensitivity`, `throbbing`, `one-sided`, `neck pain`, `dizziness`, `fatigue`.

- Nausea and vomiting are first-class here because they directly justify the anti-nausea medication and are part of the user's typical presentation.
- Stored as a `string[]` of selected chips; the user can add custom symptoms, which persist to the seeded list for reuse.

### 3.2 `context` (inline object — auto-captured, not typed)

Mirrors `LoggedSession.context` (`DATA_MODEL.md` §6.5) so migraines and training sessions share one shape and correlation needs no awkward joins.

| Field | Type | Notes |
|---|---|---|
| `cyclePhase` | enum \| null | `menstrual` / `follicular` / `ovulation` / `luteal`, resolved from `CyclePrediction` for this date (`CYCLE_AND_TCM.md`). |
| `cycleDay` | int \| null | Day within the cycle. |
| `hydrationMlToday` | number \| null | Sum of the day's `HydrationEntry` at onset time. |
| `precedingTrainingLoad` | number \| null | The rolling sRPE-based load figure (`STRENGTH_SCIENCE.md` §2–3) leading into this day. |
| `wellness` | object \| null | The day's `WellnessEntry` snapshot: `{ sleep, stress, energy, soreness }` (1–5 each). |

Context is captured **automatically** at log time so the user adds nothing — this is what makes correlation possible without extra burden. Snapshotting (rather than joining live later) means a migraine's context is preserved even if underlying data is edited.

---

## 4. Medications — pre-loaded quick picker

The user's actual medications are seeded as defaults so logging meds is one or two taps, not free text. The picker shows these two first; the user can add others, which persist.

| Field | Type | Notes |
|---|---|---|
| `name` | string | Medication name. |
| `dose` | number | Amount per tablet/unit. |
| `unit` | string | E.g. `mg`. |
| `count` | number | Number of tablets/units taken. Editable per log. |
| `timing` | timestamp \| null | When taken. Defaults to now; editable. |
| `helped` | boolean \| null | Whether it helped. Null until assessed (often filled on the second pass). |

### 4.1 Seeded defaults (this user)

| Medication | Default dose | Default count | Notes |
|---|---|---|---|
| Naproxen | 275 mg | 2 | Stored as 275 mg per tablet (naproxen sodium). Storing the mg — not just "2 tablets" — keeps history comparable if brand/tablet size ever changes. |
| Metoclopramide | 10 mg | 1 | Anti-nausea. 10 mg is the standard adult tablet strength (commonly up to three times daily). |

- Tapping a medication adds it with dose and count pre-filled; the user adjusts `count` only when it differs and sets `helped` once known.
- The `name` + `dose` are stored explicitly (not "2 tablets") so the `helped` signal stays meaningful across any future change of drug or strength.

---

## 5. Trigger discovery — associations, never causes

Surfaces patterns between migraines and the context factors, shown in Insights (`PRD.md` §7 — analysis lives in Insights, separate from logging). It **never asserts causation** (`PRD.md` §8).

### 5.1 What it correlates

Each migraine carries its `context` (§3.2). Trigger discovery looks for associations between migraine occurrence/severity and:

- **Hydration** — day's total vs. migraine days (central to the migraine hypothesis, `PRD.md` §6).
- **Cycle phase** — migraine rate per phase (e.g. clustering in menstrual/luteal).
- **Training load** — `precedingTrainingLoad` on migraine days vs. baseline.
- **Wellness inputs** — sleep / stress / energy / soreness on/around migraine days.

### 5.2 How it's computed (v1 — simple and honest)

- **Rate comparisons and simple associations only** in v1 — e.g. "migraines occur on X% of low-hydration days vs. Y% of adequate-hydration days," or "N of your last M migraines fell in the luteal phase." No causal modelling, no ML.
- A minimum-data guard: a pattern is only surfaced once there are **enough migraines to be non-trivial** (a threshold constant, tuned so it doesn't fire on two data points). Below that, the app says it's still gathering data.
- Associations are **directional hints, not scores presented as precision.** Language stays soft: "tends to," "more often," "associated with."

### 5.3 Framing rules (always applied)

- **Never** the words "cause," "trigger causes," or any phrasing implying proof. The feature is named around discovery of *associations*. ("Trigger" in the UI means "possible pattern to watch," and the copy says so.)
- Always attributed to the user's **own data** ("in your logs…"), never general medical claims.
- Correlations invite the user's judgment, they don't instruct: "worth watching," not "avoid X."
- A standing line, as on the cycle card: "Patterns from your own data. Associations, not causes."

### 5.4 The "risk flag" on the dashboard

`PRD.md` §7 lists a Migraine watch card showing "days since last migraine and any active risk flag from correlations." Defined here:

- The flag is **soft and informational** — e.g. "You're in a phase + low-hydration combo that's coincided with past migraines." It is a heads-up, not a prediction.
- It only appears once §5.2's data threshold is met, and it uses the same associations-not-causes language.
- It never diagnoses and never tells the user to medicate.

---

## 6. Data-model reconciliation (`DATA_MODEL.md` §10.2)

Confirms and expands the stub; changes to fold back into `DATA_MODEL.md`:

- `MigraineEntry` gains `endTime`, the `medications[].unit` / `count` fields, and the `context.wellness` snapshot. The rest of the stub stands.
- `medications[]` is `{ name, dose, unit, count, timing, helped }` (the stub had `{ name, dose, timing, helped }` — `unit` and `count` added so "2 × 275 mg" is representable without free-text).
- `context` matches `LoggedSession.context` field-for-field where they overlap (`cyclePhase`, `cycleDay`, `hydrationMlToday`, `precedingTrainingLoad`) and adds the `wellness` snapshot; this alignment is deliberate (`DATA_MODEL.md` §10.2 note).
- Symptom and medication seed lists live in the app's seed data, user-extendable, following the exercise-library pattern.

---

## 7. Deliberately deferred

- **Predictive migraine forecasting** — anything that forecasts a migraine before it happens. Out of scope; the risk flag (§5.4) is a soft retrospective association, not a forecast.
- **Causal inference / weighted multi-factor models** — v1 is simple rate comparisons. Richer correlation (controlling for multiple factors at once) waits for accumulated data and the broader Insights correlation engine.
- **Medication-efficacy analytics** — e.g. "naproxen helped in X% of severe migraines." The `helped` field is captured from day one so this is possible later, but it is not surfaced as its own analysis in v1.
- **Notifications/reminders** (e.g. hydration prompts when a risk flag is active) — deferred app-wide (`PRD.md` §9).

---

## 8. A note on medical safety

This module tracks and surfaces patterns; it does not advise on medication or diagnose. The seeded doses reflect what the user reports taking — the app stores them for logging convenience, it does not recommend them. If logged data ever suggests something worth clinical attention (e.g. rising frequency, escalating medication use), the app's role is to make that visible in the user's own history for them to raise with a doctor — not to interpret it. This is consistent with `PRD.md` §8's no-diagnosis principle.
