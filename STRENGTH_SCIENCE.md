# Strength Science — Personal Holistic Health App

**Version:** 1.0 (v1 scope)

## Related documents
- `PRD.md` — module 5.1 (strength), 5.4 (jiu-jitsu intensity), the dashboard cards, and the "your data, not population formulas" principle this file is built to honor.
- `DATA_MODEL.md` — the entities this logic reads and writes (`LoggedSession`, `SessionSummary`, `PersonalRecord`, `OtherWorkout`, `Program`), and the suggestion/context shapes.
- `CYCLE_AND_TCM.md` — owns cycle length / period length and the phase-prediction rules this file reads from (not yet written; referenced as a dependency).
- `ARCHITECTURE.md` — offline, local-first constraints; all of the below runs on-device with no network.

---

## 1. Philosophy — a reference-and-readiness tool, not an autopilot

This is the single most important design decision in the strength module, and it governs everything below.

The app **never autonomously changes the user's loads.** The user is an experienced lifter who autoregulates by feel and holds effort at a fixed RPE (§4). The app's job is to **surface the user's own data and current context clearly**, then get out of the way. It shows what RPE 7–8 has recently landed at, and it shows two readiness signals — systemic fatigue and cycle phase — as **display-only** information on the dashboard. The user reads them and decides.

Concretely, this file defines three things:

1. A **systemic fatigue metric** ("CNS") — displayed on the dashboard, computed from the user's whole training week. Informational (§3).
2. A **cycle-phase readiness signal** — displayed on the dashboard, with a soft, phase-specific suggestion available on expand. Informational (§5).
3. A **load reference** — shown in the Train logging flow: what RPE 7–8 has been landing at per lift, derived from the user's own logged performance. A reference, not a command (§4).

There is no mode library (hypertrophy/strength/etc. as configurable programs). The user programs their own sessions (`DATA_MODEL.md` templates); the "science" here is the readiness and reference logic layered on top, not a prescription framework.

**What "science-based" means in this file.** Where the evidence is strong (recent-load and perceived-readiness affecting session quality; sRPE as a session-load measure), the logic leans on it. Where the evidence is weak or mixed (menstrual-cycle-based load prescription; precise "CNS" recovery timelines), this file says so plainly, keeps the signal *soft and display-only*, and defers to the user's own accumulating data rather than overclaiming. Honesty about uncertainty is a feature, not a hedge.

---

## 2. The training-load currency — session RPE (sRPE)

To quantify "how hard was a session" in one consistent unit across both lifting and other training, the app uses **session RPE load (sRPE)**, the standard applied-S&C method:

```
sessionLoad = sessionRPE × durationMinutes
```

- For **lifting sessions**, `sessionRPE` is the session's average set RPE (from `SessionSummary.avgRpe`) and `durationMinutes` comes from the session timer (`LoggedSession.elapsedSeconds`).
- For **other training** (BJJ etc.), the user does not log per-set RPE; they tag the session Low / Med / High on the calendar (`OtherWorkout.intensity`). That tag maps to a coarse sessionRPE (§3.1) and combines with `durationMinutes` if present.

sRPE is a well-established, low-burden way to represent internal training load. It is the raw material for the systemic fatigue metric (§3). It is deliberately simple and uses only data already captured — no extra logging.

---

## 3. Systemic fatigue metric ("CNS") — display only

### 3.1 What it is, honestly

The dashboard shows a rolling **systemic fatigue** state so the user can see accumulated training stress at a glance — a hard roll yesterday reads high today and fades over the following days; back-to-back hard days visibly stack.

An honest naming note that the file and the UI both respect: what gym culture calls "CNS fatigue" from ordinary training is mostly **peripheral and perceived** fatigue — muscle damage, glycogen depletion, connective-tissue and general tiredness — not literal central-nervous-system exhaustion. True CNS fatigue is real but is chiefly a factor after genuinely maximal efforts (heavy singles, competition). This metric therefore tracks **general systemic/recovery load**, and the tap-through copy says as much rather than implying a neural readout. The label "CNS" may appear in the UI as the user's shorthand, but the underlying model and any explanatory text describe it as systemic fatigue.

**This metric never feeds a load suggestion.** It is purely informational. (Per the user's explicit decision: the fatigue signal is for them to see and act on, not for the app to act on.)

### 3.2 Inputs — the whole training week

Both training types contribute, so heavy lifting weeks aren't under-counted:

- **Other training (BJJ etc.)** — the Low / Med / High tag maps to an intensity weight:
  - Low → 1
  - Med → 2
  - High → 3
  - (Optionally scaled by duration if logged, via sRPE; the tag is the primary signal.)
- **Lifting sessions** — an **auto-derived** contribution from data already logged, so the user adds nothing. Derived from the session's sRPE (§2): a high-RPE, high-tonnage session contributes toward High; a light technique day contributes toward Low. The mapping bins each lifting session's sRPE into the same 1 / 2 / 3 scale so both sources share one currency.

### 3.3 Decay — the recovery window

Each training day's contribution **decays over time** so that recent days dominate and older ones fade. Based on the practical recovery window for systemic/peripheral fatigue (roughly 48–72 hours for typical hard sessions):

- **~24h (yesterday):** near-full weight — most of the effect.
- **~48h:** roughly half.
- **~72h:** largely dissipated.
- **Beyond ~72h:** negligible for a single session.

Model: an **exponential decay** applied per elapsed day, halving roughly every ~24–36h. The displayed score is the sum of decayed contributions across the trailing few days:

```
systemicFatigue(today) = Σ over recent sessions [ contribution × 2^(−daysAgo / halfLife) ]
    where halfLife ≈ 1–1.5 days
```

The exact `halfLife` constant is a single tunable in one place. It is intentionally not presented as precise physiology — it is a reasonable, adjustable smoothing constant that produces the "yesterday matters most, fades over a few days" behavior the user asked for.

### 3.4 Output — state word, number on tap

- **Dashboard (collapsed):** a single **state word** — **Low / Moderate / High** — from thresholds on the rolling score. No number at the glance level.
- **Tap-through:** the raw score, a short trailing-days breakdown (what contributed), and the honest "this is systemic/perceived fatigue, not a neural readout" note.

Thresholds (Low/Moderate/High cutoffs on the summed score) are constants tuned during build against the user's own typical weeks, so "High" corresponds to genuinely demanding stretches rather than firing constantly.

---

## 4. Load reference — RPE 7–8 anchored, data-driven

### 4.1 The anchor

The user holds **RPE 7–8** on their lifts by choice — a sustainable, sub-failure, joint-friendly effort band well suited to someone also managing BJJ load and joint health. The app therefore does **not** move target RPE around based on readiness. Effort is fixed; the app simply shows what that effort has been landing at.

This reframes autoregulation correctly: **fixed effort, and the load that produces it is read from the user's own recent data.** When the user is more fatigued, the weight that feels like RPE 7–8 naturally comes down — and the user discovers that at log time through their own RPE feedback, not through an app instruction.

### 4.2 What the reference shows

Shown inline in the Train logging flow, per exercise:

- **Main lifts (Squat, Deadlift, Bench Press — `Exercise.isMainLift`):** the load corresponding to RPE 7–8 derived from the user's **estimated 1RM trend** (`DATA_MODEL.md` §8; Epley, top working set, ≤10-rep guardrail, warm-ups excluded). Presented as a reference band, e.g. "RPE 7–8 ≈ 45–48 kg based on your recent e1RM," always labelled *est.* and always overridable.
- **All other exercises:** no formula. The app shows the **last logged performance** — "last time: 3 × 12 @ 5 kg, RPE 8" — and lets the user decide. For accessories and unconventional movements this is far more trustworthy than any population estimate, and it satisfies the PRD principle directly.

### 4.3 The closed loop

The reference is a **starting-point guess**; the user's **logged RPE is the correction**. If the suggested load felt like RPE 9 (harder than the 7–8 anchor), the next session's reference for that lift adjusts downward from the updated e1RM/last-performance data. This feedback loop is the actual engine, and it runs entirely on the user's own logged data — no population formulas drive load (`PRD.md` §5.1).

### 4.4 Relationship to the `suggestionApplied` schema

`DATA_MODEL.md` §6.4 defines a `suggestionApplied` shape (`direction`, `perLift`, `reason`, `factors`, `overridden`). In v1, given the reference-only philosophy, the app populates this as a **reference record**, not an autonomous adjustment:
- `direction` is typically `maintain` (the app isn't pushing load up or down on its own).
- `perLift[].resolvedTargetWeight` carries the RPE-7–8 reference value shown.
- `reason` is the plain-language basis, e.g. "RPE 7–8 reference from your recent e1RM."
- `factors` records the context shown alongside (current cycle phase, current systemic-fatigue state) **for the user's information**, not as inputs that changed the number.
The schema is thus forward-compatible with a future opt-in autoregulation mode without requiring a change now.

---

## 5. Cycle-phase readiness — display only, soft, learnable

### 5.1 The honest position

The research on menstrual-cycle-based training prescription is **mixed and low-quality** — small studies, high individual variability, no strong consensus that phase-based load adjustment beats simply autoregulating by feel. This file does not pretend otherwise.

But the user personally reports feeling more fatigued during menstruation, and that lived signal is legitimate to surface. So the cycle layer is handled the strongest honest way: **a soft, display-only readiness note that the app's own data can later confirm or quietly retire** — fully in line with the app's correlation-from-your-own-dataset thesis.

### 5.2 Dashboard behavior — progressive disclosure

- **Collapsed (dashboard card):** current **phase + cycle day** only — e.g. "Luteal · day 22." Pure status, no advice at the glance.
- **Expanded (tap):** a **soft, phase-specific** note, framed as the user's tendencies, never as prescription and never auto-applied:
  - **Follicular / Ovulation:** "You often feel strong here — a good window if you want to push."
  - **Luteal / Menstrual:** "Energy may dip here; keeping volume lighter is completely fine."
  - Menstrual specifically carries the user's own reported "more fatigued" note.
- Over time, once enough history exists, the expanded view can add the **your-data** version — e.g. "Your logged RPE-at-load tends to run higher this phase" — or, if the data doesn't bear it out, the app simply stops asserting the tendency. The assumption is testable against the user's own logs, not treated as fixed truth.

### 5.3 Source of phase and cycle day

The phase and cycle-day shown come from the cycle module's prediction (`DATA_MODEL.md` §10.1, `CyclePrediction`). **The rule-based cycle length and period length that drive that prediction are defined in `CYCLE_AND_TCM.md`, not here** — this file reads whatever phase the cycle module reports for the current date. (Placeholder until `CYCLE_AND_TCM.md` sets the real defaults.)

### 5.4 Not an engine input

Consistent with §1 and the user's decision: cycle phase **does not change any load reference or suggestion.** It is shown on the dashboard for the user to factor in themselves. The `LoggedSession.context.cyclePhase` field still records the phase per session (for later migraine/strength correlation in Insights), but recording context is not the same as steering a suggestion.

---

## 6. Anthropometrics — bodyweight-relative strength only

The user's **weight and height** are stored, but their role is deliberately narrow, to respect the "your logged performance, not population formulas" principle:

- **Bodyweight-relative strength:** e1RM ÷ current bodyweight, trended over time. A cleaner progress signal than raw kg — it accounts for whether strength gains are tracking with (or independent of) bodyweight changes. Shown in Insights as an optional view.
- **Bodyweight-exercise context:** for movements loaded by the user's own bodyweight (pull-ups, dips, Copenhagen planks, etc.), bodyweight provides the load context that raw "reps only" logging misses.

Explicitly **not** used:
- Height plays essentially no role in load prescription and is stored only for completeness / future reference (e.g. BMI-style context the user might want).
- Neither weight nor height ever drives a load suggestion via population formulas. The load reference (§4) is anchored to the user's own logged RPE and e1RM, full stop.

Bodyweight is captured as a simple periodic entry (a lightweight `BodyMetric` record — `{ id, date, weightKg, heightCm? }`; heightCm rarely changes and can be a single profile value). This is a small addition to the data model to be finalized when this file is wired in (§8).

---

## 7. Summary — the complete model

| Signal | Source | Where shown | Effect on load |
|---|---|---|---|
| Systemic fatigue ("CNS") | BJJ Low/Med/High tags + auto-derived lifting sRPE, exponentially decayed over ~48–72h | Dashboard: Low/Mod/High state word; number on tap | **None** — display only |
| Cycle phase readiness | `CyclePrediction` (phase + day) | Dashboard: phase+day collapsed; soft phase note on expand | **None** — display only |
| Load reference | e1RM trend (main lifts) / last performance (all others), anchored to RPE 7–8 | Train logging flow, per exercise | **Reference only** — user always decides |
| Bodyweight-relative strength | e1RM ÷ bodyweight | Insights (optional view) | None — a progress metric |

The through-line: **two display-only readiness signals and a data-driven load reference, with no autopilot.** Every number the app shows about load comes from the user's own logged history; every readiness signal is information for the user to weigh, not an instruction the app acts on. This is both the more honest design given the state of the evidence and the more trustworthy one given how this user actually trains.

---

## 8. Data-model hooks (to reconcile when wiring in)

- **`BodyMetric`** (new, §6): `{ id, date, weightKg, heightCm? }`. Height may instead live as a single profile value. To add to `DATA_MODEL.md`.
- **Systemic fatigue** is computed on read from `OtherWorkout.intensity` and `SessionSummary` (sRPE); it is not stored as its own entity in v1. The decay `halfLife` and the Low/Mod/High thresholds are constants in the strength logic module.
- **`OtherWorkout.intensity`** (`DATA_MODEL.md` §9.3) is confirmed as a **Low/Med/High** enum (→ 1/2/3), not a 1–10 scale — reconcile the field note in `DATA_MODEL.md`.
- **`suggestionApplied`** (`DATA_MODEL.md` §6.4) is populated as a reference record in v1 (§4.4); no schema change needed.
- **Cycle length / period length** and the phase-prediction rule remain owned by `CYCLE_AND_TCM.md`; this file only consumes the resulting phase.

---

## 9. Deliberately deferred

- **Opt-in autoregulation mode** — a future setting where the app *does* propose load/volume adjustments from readiness. The schema (§4.4) is already compatible; kept out of v1 by choice, since the user wants a reference tool, not an autopilot.
- **Learned cycle→performance correlation** surfaced as a confirmed pattern — depends on accumulated data and the Insights correlation engine; framed in §5.2 as a later addition.
- **Per-movement fatigue overlap** (e.g. weighting yesterday's hard roll more heavily on leg days) — considered and set aside for simplicity; the systemic metric is whole-session, not movement-specific, in v1.
