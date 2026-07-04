# Data Model — Personal Holistic Health App

**Version:** 1.0 (v1 scope)

## Related documents
- `PRD.md` — modules and priorities this schema serves.
- `ARCHITECTURE.md` — the storage layer (§4–5), file/versioning rules (§8), and the single storage module that owns all reads/writes.
- `STRENGTH_SCIENCE.md` — training modes and the cycle/load deload logic that consume this schema (not yet written).
- `CYCLE_AND_TCM.md`, `MIGRAINE_LOGIC.md` — the cycle and migraine modules (not yet written); their entities are stubbed here and expanded there.

---

## 1. Purpose and conventions

This document defines the entities stored in IndexedDB (`ARCHITECTURE.md` §4), their fields, and their relationships. It is the single source of truth for the schema; every feature reads and writes through the one storage module (`ARCHITECTURE.md` §5).

**Conventions used throughout:**
- Each entity is one IndexedDB object store. Field types are given as: `string`, `number`, `int`, `boolean`, `timestamp` (ISO 8601 string), `date` (`YYYY-MM-DD` string, no time), `enum`, `ref` (id of another entity), `ref[]` (list of ids), or an inline object.
- Every entity has a string `id` (a generated UUID) as its primary key unless stated otherwise.
- `createdAt` and `updatedAt` (`timestamp`) are present on every entity and omitted from the field tables below to reduce noise.
- Deletions are soft where history matters: a `deletedAt` (`timestamp | null`) marks a record hidden from pickers and summaries but retained so past logs that reference it still resolve. Hard deletion happens only on explicit user data-clear.
- All exercise-name and label text is stored in **Title Case** (e.g. `Barbell High Bar Back Squat`), matching the user's own programming. The import normalizer (§11) enforces this.

**Schema version.** The database carries an integer `schemaVersion`. It is written into every exported/synced file (`ARCHITECTURE.md` §7–8) so an older file can be migrated forward on import. The current value is `1`.

---

## 2. Entity overview

The v1 stores, grouped by module:

**Strength / Train (Priority 1 — fully specified here)**
- `Exercise` — the exercise library (§3).
- `Program` — a training block, e.g. a 4-week run (§4).
- `SessionTemplate` — a reusable named session within a program: A1, B1, C1… (§5).
- `TemplateSlot` — one exercise position inside a template, with its targets (§5).
- `LoggedSession` — a dated performance of a template (§6).
- `LoggedExercise` — one exercise as actually performed (or skipped) in a logged session (§6).
- `LoggedSet` — one set: weight, reps, RPE (§6).
- `SessionSummary` — the computed roll-up stored on finishing a session (§7).
- `PersonalRecord` — detected PRs, including tested-max entries (§8).

**Cross-cutting (specified here — small and stable)**
- `HydrationEntry` (§9.1)
- `WellnessEntry` (§9.2)
- `OtherWorkout` — jiu-jitsu and other training, carrying the intensity signal (§9.3)
- `BodyMetric` — periodic bodyweight/height (§9.4)

**Cycle and Migraine (Priority 2–3 — stubbed here, expanded in their own docs)**
- `CycleEntry`, `CyclePrediction` (§10.1)
- `MigraineEntry` (§10.2)

The relationships among the Train entities:

```
Program 1───* SessionTemplate 1───* TemplateSlot *───1 Exercise
                    │
                    └──── (a template is performed as) ────┐
                                                           ▼
LoggedSession *───1 SessionTemplate     LoggedSession 1───* LoggedExercise *───1 Exercise
LoggedSession 1───1 SessionSummary      LoggedExercise 1───* LoggedSet
```

---

## 3. `Exercise`

The exercise library. Seeded from the user's own catalogue (163 movements) and extendable by the user. Every logged and planned exercise references one `Exercise`. The taxonomy fields drive filtering in the picker, and `primaryMuscle` drives the volume analytics — the ranked sets-per-muscle list in Insights (§7, prime-mover-only attribution).

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `name` | string | Title Case, unique. E.g. `Barbell Conventional Deadlift`. |
| `exerciseType` | enum | Broad category. E.g. `Squat`, `Hinge`, `Vertical Push`, `Horizontal Pull`, `Accessory`, `Rehab`, `Carry`, `Conditioning`, `Power`, `Strength`, `Core`. Values may be comma-combined (e.g. `Accessory, Rehab`) — stored as a `string[]` if more than one applies. |
| `movementPattern` | string | E.g. `Hip Hinge`, `Knee Dominant`, `Vertical Pull`, `Anti-Extension`, `Ankle Plantarflexion`. Free-form but normalized on import. |
| `bodyRegion` | enum | E.g. `Lower Body`, `Upper Body`, `Full Body`, `Core`, `Ankle`, `Knee`. |
| `primaryMuscle` | string | The prime mover. **This is the sole muscle credited in weekly-volume attribution** (§7). E.g. `Quadriceps`, `Hamstrings`, `Gluteus Maximus`, `Back`. |
| `secondaryMuscle` | string \| null | Reference/education only; not counted in volume. |
| `tertiaryMuscle` | string \| null | Reference/education only; not counted in volume. |
| `planeOfMotion` | enum[] | One or more of `Sagittal`, `Frontal`, `Transverse`. Import normalizes variants (`Sagittal Plane` → `Sagittal`; typo `Tranverse` → `Transverse`). |
| `regressionOf` | ref \| null | The easier variant (points to another `Exercise`), for the swap/skip flow. |
| `progressionOf` | ref \| null | The harder variant. |
| `isMainLift` | boolean | `true` only for the three lifts that get estimated-1RM tracking: Squat, Deadlift, Bench Press (§8). Default `false`. |
| `loadType` | enum | How this exercise is loaded, which decides what the log captures and whether e1RM applies. One of: `weighted` (kg per set), `bodyweight` (reps only), `banded` (band label in note, reps only), `duration` (seconds, e.g. Copenhagen Plank), `weighted_duration` (weight + seconds). |
| `isCustom` | boolean | `true` if user-created rather than seeded. |
| `defaultNote` | string \| null | Optional carried note, e.g. `Purple band`, `Smith`, `SL`. Pre-fills the exercise note when added (§5, §6). |
| `deletedAt` | timestamp \| null | Soft-delete. |

**Notes.**
- The e1RM estimate (§8) is only ever computed for `Exercise` records where `isMainLift === true` and `loadType === 'weighted'`. All other exercises progress by raw best (heaviest weight, most reps at a load, or longest duration) — no formula is applied.
- `secondaryMuscle`/`tertiaryMuscle` are retained from the source catalogue for exercise detail and education, but the finalized volume rule is **prime-mover-only** (`PRD.md` §5.1, confirmed): one performed set credits exactly one set to `primaryMuscle`.

---

## 4. `Program`

A training block the user runs for a fixed number of weeks, then rotates. Default length is 4 weeks. A program owns its session templates; finishing a run can clone-and-progress into the next program.

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `name` | string | E.g. `Program 4`, or a user label. Title Case. |
| `lengthWeeks` | int | Default `4`. The intended run length. |
| `startDate` | date \| null | When the run began. Null until first session is logged. |
| `endDate` | date \| null | Set when the user ends/rotates the program. |
| `status` | enum | `active`, `completed`, `draft`. Exactly one program is `active` at a time in v1. |
| `clonedFrom` | ref \| null | The `Program` this was cloned from, for progression lineage. |
| `deletedAt` | timestamp \| null | Soft-delete. |

**Notes.**
- Insights scopes its analytics to a selected program (the picker at the top of the Insights screen). Weekly windows are computed from `startDate`.
- "End of week 4 → prompt to clone-and-progress" is app logic, not schema: it reads `lengthWeeks` and the count of elapsed weeks since `startDate`.

---

## 5. `SessionTemplate` and `TemplateSlot`

A **`SessionTemplate`** is a reusable, named session (A1, B1, C1, A2, …) the user builds once and reuses every run. It holds the plan, not any performance. A **`TemplateSlot`** is one exercise position within it.

This is the "save each workout and each set for a program, reuse it, and edit only the exercises I want" requirement: the template is the saved plan; editing a slot changes the plan going forward; logging clones the plan's targets as starting values without mutating the template.

### 5.1 `SessionTemplate`

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `programId` | ref | The owning `Program`. |
| `label` | string | The session's rotation label, e.g. `A1`, `B1`, `C1`. Title/upper case. |
| `name` | string \| null | Optional descriptive name, e.g. `Squat + Pull`. |
| `orderIndex` | int | Position of this session within the program's rotation. |
| `deletedAt` | timestamp \| null | Soft-delete. |

### 5.2 `TemplateSlot`

One exercise per slot. The slot's `stationLabel` is the user's own position code (`A1`, `B1`, `C1`, `D1`, `E1`…). There is exactly **one exercise per station** — stations are ordered positions, not supersets.

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `templateId` | ref | The owning `SessionTemplate`. |
| `exerciseId` | ref | The chosen `Exercise`. |
| `stationLabel` | string | E.g. `A1`, `B1`. Determines order within the session. |
| `orderIndex` | int | Numeric sort key derived from `stationLabel` for stable ordering. |
| `targetSets` | int | Planned number of working sets. |
| `targetReps` | int \| string | Planned reps. String allows ranges or durations (`8`, `8-10`, `45s`). |
| `targetRpe` | number \| null | Planned RPE (0–10), optional. |
| `targetWeight` | number \| null | Optional planned load (kg). Often null — load is decided at log time via the suggestion engine. |
| `restTargetSeconds` | int \| null | Planned rest between sets, shown by the per-exercise rest timer at log time. Optional. |
| `note` | string \| null | Exercise-level planning note, pre-filled from `Exercise.defaultNote` (e.g. `Purple band`, `Smith`, `SL`). Editable. |
| `deletedAt` | timestamp \| null | Soft-delete. |

**Notes.**
- Editing a `TemplateSlot` (swap the exercise, change target reps, add a note) affects future logged sessions only; already-logged sessions keep their own copied values (§6).
- The picker's "Skip/Replace" swap can use `Exercise.regressionOf` / `progressionOf` to offer sensible alternatives.

---

## 6. `LoggedSession`, `LoggedExercise`, `LoggedSet`

The performance records. A **`LoggedSession`** is a dated instance created from a template; it copies the template's slots into `LoggedExercise` rows so the log is a faithful record even if the template later changes. Each `LoggedExercise` owns its `LoggedSet` rows or is marked skipped.

### 6.1 `LoggedSession`

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `programId` | ref | The program this session belongs to. |
| `templateId` | ref \| null | The `SessionTemplate` it was created from. Null if an ad-hoc session. |
| `date` | date | The day it was performed. |
| `startedAt` | timestamp \| null | Set when the user manually **starts** the session timer (not on screen open). Null until started. |
| `finishedAt` | timestamp \| null | Set on "Finish session"; triggers summary computation (§7). |
| `elapsedSeconds` | int | Accumulated session duration. Because the timer is manually start/stop (and can be paused/resumed), duration is tracked as accumulated elapsed time rather than a simple `finishedAt − startedAt`. |
| `status` | enum | `in_progress`, `finished`. |
| `suggestionApplied` | object \| null | Snapshot of the cycle/load suggestion shown for this session (§6.4). |
| `context` | object | Cross-domain context captured at session time (§6.5) — the hook that makes correlations computable. |
| `note` | string \| null | Optional session-level note. |
| `deletedAt` | timestamp \| null | Soft-delete. |

### 6.2 `LoggedExercise`

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `sessionId` | ref | The owning `LoggedSession`. |
| `exerciseId` | ref | The `Exercise` performed. |
| `stationLabel` | string | Copied from the slot (`A1`, `B1`…). |
| `orderIndex` | int | Copied ordering. |
| `note` | string \| null | Exercise-level note as performed (starts from the slot note, editable). |
| `skipped` | boolean | `true` if the user skipped it. Default `false`. |
| `skipReason` | string \| null | Optional free-text reason. |

**Skip semantics.** A skipped `LoggedExercise` is **retained and visible** in the log (shown struck-through/tagged), but contributes **zero** sets to volume and nothing to tonnage, RPE, or e1RM. It is never silently removed.

### 6.3 `LoggedSet`

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `loggedExerciseId` | ref | The owning `LoggedExercise`. |
| `setIndex` | int | 1-based order of the set. |
| `weight` | number \| null | kg. Null for bodyweight/banded/duration loads. |
| `reps` | int \| null | Null for pure-duration exercises. |
| `durationSeconds` | int \| null | For `duration` / `weighted_duration` loads (e.g. plank). |
| `rpe` | number \| null | 0–10. Optional per set. |
| `isWarmup` | boolean | `true` for warm-up sets (the `W` row). Default `false`. Warm-up sets are **fully excluded** from working-set count, tonnage, `setsPerMuscle`, and e1RM (§7, §8). They are still stored and shown in the log. |
| `note` | string \| null | Per-set note (e.g. `Felt heavy, knees caved`). |
| `isCompleted` | boolean | Whether the set was actually done (vs. a pre-filled placeholder). |

### 6.4 `suggestionApplied` (inline object on `LoggedSession`)

A snapshot, so the log preserves what was suggested even after the logic evolves. The **numbers and phase logic** live in `STRENGTH_SCIENCE.md`; this schema only fixes the shape.

| Field | Type | Notes |
|---|---|---|
| `direction` | enum | `deload`, `load`, `maintain`. |
| `perLift` | object[] | Each: `{ exerciseId, adjustmentPct, resolvedTargetWeight }`. `adjustmentPct` e.g. `-10` for a 10% cut. |
| `reason` | string | Human-readable reasoning shown inline, e.g. `Luteal + high BJJ load → −10% on squat`. |
| `factors` | object | The inputs that produced it: `{ cyclePhase, recentTrainingLoad, recentRpeTrend, e1rmTrend }`. |
| `overridden` | boolean | `true` if the user changed the suggested loads. |

### 6.5 `context` (inline object on `LoggedSession`)

Captured automatically at session time so every strength session timestamps its cross-domain context — the basis for migraine/cycle correlation later.

| Field | Type | Notes |
|---|---|---|
| `cyclePhase` | enum \| null | `menstrual`, `follicular`, `ovulation`, `luteal` — resolved from the cycle module for this date. |
| `cycleDay` | int \| null | Day within the cycle. |
| `precedingTrainingLoad` | number \| null | A rolling load figure from recent `OtherWorkout` + strength sessions (defined in `STRENGTH_SCIENCE.md`). |
| `hydrationMlToday` | number \| null | Sum of the day's `HydrationEntry` at session time. |

---

## 7. `SessionSummary`

Computed once on `finishedAt` and stored (denormalized) so Insights and the dashboard read fast without re-scanning sets. Recomputed if the session is later edited.

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `sessionId` | ref | One-to-one with a finished `LoggedSession`. |
| `totalWorkingSets` | int | Count of completed, non-skipped, **non-warm-up** sets. |
| `tonnageKg` | number | Σ over completed **working** (non-warm-up) weighted sets of `weight × reps`. |
| `avgRpe` | number \| null | Mean RPE over completed working sets that carry an RPE. Warm-ups excluded. |
| `peakRpe` | number \| null | Max RPE among working sets. |
| `setsPerMuscle` | object | `{ [primaryMuscle]: setCount }` — **prime-mover-only** attribution: each completed **working** set adds 1 to its exercise's `primaryMuscle`. Warm-up sets contribute nothing. Feeds the Insights volume list. |
| `setsPerPattern` | object | `{ [movementPattern]: setCount }` — the generalized Leg/Hinge/Push/Pull-style buckets, for balance ratios (push:pull, knee:hip). |
| `e1rmTopSet` | object | `{ [exerciseId]: estimated1RM }` for main lifts only; see §8. |
| `prsHit` | ref[] | `PersonalRecord` ids detected in this session. |

**Weekly aggregation** (weekly sets per muscle, weekly tonnage, push:pull, etc., shown in Insights) is computed on read by summing `SessionSummary` rows whose session `date` falls in the selected program-week window. It is not stored as its own entity in v1.

---

## 8. `PersonalRecord`

Tracks bests. Two distinct kinds, kept separate on purpose: **estimated** 1RM (derived, always labelled "est.") and **tested** 1RM (an actual logged single). They never overwrite each other.

### 8.1 Estimated 1RM — how it's calculated

- Computed **only** for `Exercise` records with `isMainLift === true` and `loadType === 'weighted'` — i.e. Squat, Deadlift, Bench Press.
- **Formula: Epley** — `e1RM = weight × (1 + reps / 30)`. Chosen because it is the standard for compound lifts and is reliable across the user's 5–8 rep working range.
- **Rep guardrail:** only sets with `reps ≤ 10` contribute an estimate. Sets above 10 reps are logged but ignored for e1RM, because estimation accuracy degrades sharply past ~10 reps.
- **Warm-up sets (`isWarmup === true`) never contribute** to the estimate — only working sets are considered.
- Per session, the estimate is taken from the **top working set** of that lift (the set producing the highest e1RM). This value is stored in `SessionSummary.e1rmTopSet` and plotted as the strength trend in Insights.
- The estimate is **always displayed as "est."** and is a training guide, not a measured max; it does not account for CNS readiness or daily motivation.

### 8.2 `PersonalRecord` fields

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `exerciseId` | ref | The lift. |
| `kind` | enum | `estimated_1rm`, `tested_1rm`, `weight_pr` (heaviest weight at any reps), `rep_pr` (most reps at a given weight). |
| `value` | number | The record value (kg for 1RM/weight PRs; rep count for rep PRs). |
| `atWeight` | number \| null | For `rep_pr`, the load the reps were done at. |
| `sourceSessionId` | ref \| null | The session it was detected in; null for a manually entered tested max. |
| `date` | date | When achieved. |
| `isTested` | boolean | `true` only for `tested_1rm`. Keeps estimated and tested bests distinct in the UI. |

**Rule:** a `tested_1rm` is only ever created from an explicit logged single (or manual entry). An `estimated_1rm` PR is never promoted to a tested max. Insights can show both on the same trend, visually distinguished, but they are separate records.

---

## 9. Cross-cutting entities

### 9.1 `HydrationEntry`

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `date` | date | The day. |
| `timestamp` | timestamp | When logged (for time-of-day analysis in migraine correlation). |
| `amountMl` | int | Volume added. |

Daily total = sum of the day's entries. Central to the migraine hypothesis (`PRD.md` §6), and snapshotted into `LoggedSession.context.hydrationMlToday`.

### 9.2 `WellnessEntry`

Quick 1–5 daily scales (`PRD.md` §6).

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `date` | date | One entry per day (upserted). |
| `sleep` | int \| null | 1–5. |
| `stress` | int \| null | 1–5. |
| `energy` | int \| null | 1–5. |
| `soreness` | int \| null | 1–5. |

### 9.3 `OtherWorkout`

Jiu-jitsu and other training. Carries the **intensity** signal that feeds the strength deload/load logic (`PRD.md` §5.4).

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `date` | date | The day. |
| `type` | string | E.g. `Jiu-Jitsu`, `Running`, `Conditioning`. Title Case. |
| `durationMinutes` | int \| null | Session length. |
| `intensity` | enum | `Low`, `Med`, `High` (→ 1/2/3). The CNS/systemic-fatigue signal consumed by the dashboard readiness metric (`STRENGTH_SCIENCE.md` §3). Display-only; does not drive load suggestions. |
| `note` | string \| null | Optional. |

### 9.4 `BodyMetric`

Periodic bodyweight (and, rarely, height) capture. Used only for bodyweight-relative strength trends and bodyweight-exercise context — never as a load driver (`STRENGTH_SCIENCE.md` §6).

| Field | Type | Notes |
|---|---|---|
| `id` | string | Primary key. |
| `date` | date | When measured. |
| `weightKg` | number | Bodyweight. |
| `heightCm` | number \| null | Rarely changes; may instead be stored as a single profile value rather than per-entry. |

---

## 10. Cycle and migraine (stubs — expanded in their own docs)

These are defined minimally so the Train context (§6.5) and cross-domain correlations resolve. Full fields, prediction rules, and correlation logic live in `CYCLE_AND_TCM.md` and `MIGRAINE_LOGIC.md`.

### 10.1 `CycleEntry` / `CyclePrediction`

`CycleEntry`: `{ id, date, flow (enum: none/spotting/light/medium/heavy), symptoms (string[]), note }`.

`CyclePrediction` (derived): `{ id, date, phase (enum: menstrual/follicular/ovulation/luteal), cycleDay (int), predictedPeriodStart (date), method (enum: rule_based | learned), confidence (number | null) }`.

The phase and cycle-day used in `LoggedSession.context` come from the `CyclePrediction` for that date. The rule-based starting parameters (typical cycle length, period length) are set in `CYCLE_AND_TCM.md`.

### 10.2 `MigraineEntry`

Minimal shape (full field list in `MIGRAINE_LOGIC.md`):

`{ id, date, onsetTime (timestamp), severity (int, single-hue 1–3 scale per DESIGN_SYSTEM), durationMinutes, symptoms (string[], incl. nausea/vomiting), medications (object[]: { name, dose, timing, helped (boolean) }), context (object: { cyclePhase, hydrationMlToday, precedingTrainingLoad }) }`.

The `context` object mirrors §6.5 so trigger discovery can align migraines against hydration, cycle phase, and training load without joins across incompatible shapes.

---

## 11. Import, normalization, and versioning

- **Seeding.** The exercise library is seeded from the user's catalogue. The importer normalizes on the way in: Title-Cases names and labels; collapses plane variants (`Sagittal Plane` → `Sagittal`, `Tranverse` → `Transverse`); splits comma-combined `exerciseType` into an array; and resolves `regressionOf` / `progressionOf` name references into ids (leaving null when a referenced variant isn't in the library).
- **Migrations.** On load, if the stored `schemaVersion` is below the current, migrations run in order to bring local data up without loss (`ARCHITECTURE.md` §8).
- **Files.** Export/sync files embed `schemaVersion` and a last-modified timestamp, enabling forward-migration on import and the stale-file warning (`ARCHITECTURE.md` §6, §8).

---

## 12. Open items (resolved when their docs are written)

- `intensity` scale for `OtherWorkout` resolved to Low/Med/High (§9.3); the systemic-fatigue metric and sRPE currency are defined in `STRENGTH_SCIENCE.md`.
- Suggestion `adjustmentPct` values → in v1 the engine is reference-only (`STRENGTH_SCIENCE.md` §4.4); `suggestionApplied` is populated as a reference record (schema shape in §6.4 is final and forward-compatible).
- Rule-based cycle length / period length defaults and the learned-prediction fields → `CYCLE_AND_TCM.md`.
- Full migraine field enums (symptom checklist, medication list, severity scale wording) → `MIGRAINE_LOGIC.md`.
