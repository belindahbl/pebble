# Cycle & Nutrition — Personal Holistic Health App

**Version:** 1.0 (v1 scope)

## Related documents
- `PRD.md` — module 5.2 (period tracker + suggestions), the dashboard cycle card, and the "correlation not causation / no medical-diagnosis" principles this file honors.
- `DATA_MODEL.md` — the `CycleEntry` / `CyclePrediction` entities (§10.1) this file owns the rules for, and `LoggedSession.context` / `MigraineEntry.context` which read the phase.
- `STRENGTH_SCIENCE.md` — consumes the phase this file produces (§5); this file does **not** set any load logic.
- `ARCHITECTURE.md` — offline, local-first, no-backend constraints; all prediction and all guidance content runs on-device with no network and no live text generation.
- `DESIGN_SYSTEM.md` — the phase color tokens and the card/component styling the dashboard card uses.

---

## 1. Purpose and scope

This file owns two things:

1. **Rule-based menstrual-phase prediction** — how the app decides which of the four phases the user is in on any given date, and when the next period is expected.
2. **Per-phase guidance content** — the short, practical hormone / food / training copy shown on the dashboard cycle card and the Cycle screen.

**What this file does NOT own.** It does not set any training load or volume adjustment — that stays in `STRENGTH_SCIENCE.md`, which merely *reads* the phase this file reports (`STRENGTH_SCIENCE.md` §5.4). Cycle phase is display-only across the whole app; it never steers a load suggestion.

**A framing note carried from `STRENGTH_SCIENCE.md` §5.1, applied here too.** The research on cycle-phase–based training and nutrition prescription is mixed and of variable quality, with high individual variability. This file therefore treats all guidance as **soft, personal, and learnable from the user's own data** — never as prescription, never as a medical claim (`PRD.md` §8). The app supports self-understanding; it does not diagnose.

---

## 2. How the guidance text works — no live AI, no data-reading prose

This is a deliberate design decision worth stating plainly, because it is easy to assume otherwise.

The per-phase copy (hormone summary, food groups, training note) is **fixed, pre-written content** — one short block authored per phase, stored in the app. The only thing computed from the user's data is **which phase and cycle day they are in** (§3). That value selects which pre-written block to show and fills in the live numbers (phase name, cycle day, days until next period).

- **No language model runs at display time.** The app is offline and server-free (`ARCHITECTURE.md`); there is no live text generation describing the user's body.
- **The prediction reads the user's data; the words do not.** Phase detection is a rule over logged cycles. The prose is static copy keyed to the resulting phase.
- This keeps the app honest (no invented medical statements), fast, and fully offline.

Guidance content is versioned with the app, not per-user. Editing the copy is a content change, not a data migration.

---

## 3. Rule-based phase prediction

### 3.1 The four phases

| Phase | Plain definition | Color token (`DESIGN_SYSTEM.md`) |
|---|---|---|
| Menstrual | The logged flow days (period start through last flow day). | `--phase-menstrual` |
| Follicular | Period end up to ovulation. **This is the phase that stretches or shrinks** with cycle length. | `--phase-follicular` |
| Ovulation | The ~1–2 day window around predicted ovulation. | `--phase-ovulation` |
| Luteal | Ovulation to the next period. Biologically **stable at ~14 days**. | `--phase-luteal` |

### 3.2 The method — count backward from the next predicted period

The single most important fact driving this design: **the luteal phase has a roughly fixed ~14-day length, while cycle-length variability lives almost entirely in the follicular phase.** The corpus luteum has a fixed lifespan, so when a cycle runs long or short, it is the *pre-ovulation* stretch that changes, not the post-ovulation one.

Basic apps forward-count from day 1 and place ovulation on a fixed day (e.g. day 14). For a user whose cycles swing (this user's range is 27–35 days — §4), that is often wrong. The sound method is to anchor on the stable end:

```
predictedNextPeriodStart = lastPeriodStart + avgCycleLength
ovulationDay             = predictedNextPeriodStart − LUTEAL_LENGTH   (LUTEAL_LENGTH = 14)
lutealPhase              = the 14 days before predictedNextPeriodStart
menstrualPhase           = actual logged flow days (fallback: avgPeriodLength from lastPeriodStart)
follicularPhase          = period end → ovulationDay   (absorbs all cycle-length variability)
```

The menstrual phase is driven by **actual logged flow** when available (the user logs period and spotting days), falling back to `avgPeriodLength` only for future/unlogged dates.

### 3.3 Defaults (rule-based starting parameters)

Seeded from this user's own recent Flo history (§4), refined to a rolling personal average as they log in-app:

| Parameter | Default | Notes |
|---|---|---|
| `avgCycleLength` | **31 days** | Mean of the user's five most recent cycles (27–35 range). Rolling average once in-app logging begins. |
| `avgPeriodLength` | **7 days** | Rolling average; the user's observed range is 5–10 days. |
| `LUTEAL_LENGTH` | **14 days** | Fixed. The stable anchor the whole method depends on. |

`avgCycleLength` and `avgPeriodLength` are **rolling personal averages** — the app recomputes them from the trailing N logged cycles (N ≈ 6 suggested) so the prediction tracks the user rather than a textbook.

### 3.4 Confidence — widens with the user's own variability

Because this user's cycles span 27–35 days (an 8-day spread), a single-day ovulation guess would overstate precision. The prediction therefore carries a **confidence inversely tied to cycle-length variability**:

- Compute the spread (e.g. standard deviation, or simply max − min) of recent cycle lengths.
- **Low variability → narrow ovulation window, higher confidence.**
- **High variability → wider ovulation window (e.g. ±2–3 days), lower stated confidence,** and the UI leans harder on the "est." framing.
- This populates `CyclePrediction.confidence` (`DATA_MODEL.md` §10.1). Method is `rule_based` in v1.

Every predicted value in the UI is labelled **"est."** This is the same honesty posture as the strength e1RM ("est.") and readiness signals.

### 3.5 A note on this user's data (internal — not a diagnosis, not shown in-app)

The user's period length varies 5–10 days, and one recent cycle had a 10-day period inside a 32-day cycle. The phase math must handle a long period gracefully (menstrual phase is flow-driven, so it already does). Per `PRD.md` §8 the app makes no medical claims and surfaces none of this as a warning; it is noted here only so the logic is built to be robust to that variability.

---

## 4. Seed data — the user's recent cycles

Extracted from the user's Flo calendar (Jan–Jul 2026). Period start → next start:

| Cycle start | Next start | Cycle length | Period length |
|---|---|---|---|
| Jan 28 | Mar 4 | 35 days | 7 days |
| Mar 4 | Apr 5 | 32 days | 10 days |
| Apr 5 | May 9 | 34 days | 6 days |
| May 9 | Jun 6 | 28 days | 9 days |
| Jun 6 | Jul 3 | 27 days | 5 days |

- **Mean cycle length ≈ 31 days** (range 27–35) → `avgCycleLength` default.
- **Mean period length ≈ 7 days** (range 5–10) → `avgPeriodLength` default.
- These seed the rolling averages; in-app logging replaces them over time.

*(Numbers read from screenshots — worth the user eyeballing against Flo's own "average cycle length" stat if shown, to confirm.)*

---

## 5. Per-phase guidance content

Each phase provides a small, fixed content block. Structure is identical across phases for a consistent card (§6), in this order:

- **State chips:** two glanceable words — `Energy` and `Mood`.
- **Hormone line ("What's going on"):** one plain sentence — what's high/low and what to expect. Fun but practical.
- **Food groups ("Eat for this phase"):** grouped **by nutrient** — the group label *is* the nutrient or food-type ("Magnesium", "Iron", "Vitamin B6", "Complex carbs & fiber"). Foods are chips under each label with a short "why". Vitamins/minerals are folded into the group labels, not a separate row. Each phase's **last** group is the TCM group, labelled **"Warming (TCM)"** or the phase-appropriate TCM action (§5.5), so the traditional guidance is visibly marked as TCM without cluttering the rest.
- **Sleep section ("Rest & sleep"):** its own section (per user decision). One or two sentences on what changes and what helps.
- **Training section:** its own section. One soft, display-only sentence. Never an instruction; always overridable; never auto-applied (consistent with `STRENGTH_SCIENCE.md` §1, §5).

**Evidence basis.** The nutrition and sleep content below is drawn from current reviews and studies (see §5.6 for the through-line). The TCM group is explicitly labelled "(TCM)" on screen so the user always knows which guidance is traditional versus evidence-based — this replaces the earlier internal-only split, at the user's request the label is now visible and honest.

### 5.1 Menstrual

- **Chips:** Energy `Low` · Mood `Tender`
- **Hormone line:** "Estrogen and progesterone are both at their floor — energy's lowest here, so rest counts as productive. You're losing iron with the bleed, so top it up."
- **Food groups:**
  - **Iron** *(replaces what the bleed takes)* — red meat, lentils, spinach, tofu, pumpkin seeds.
  - **Vitamin C** *(helps iron absorb)* — citrus, bell pepper, strawberries, tomato.
  - **Warming (TCM)** *(moves blood & qi)* — ginger tea, red date tea, cinnamon, bone broth. *(TCM: menstrual phase favours warm foods and moving blood/qi; avoid cold/raw and iced drinks if you cramp.)*
- **Sleep:** "Cramps and discomfort can break up sleep on the heaviest days. A warm compress and an earlier night help; sleep usually settles as flow eases."
- **Training:** "Low energy is expected — light movement or full rest is completely fine."

### 5.2 Follicular

- **Chips:** Energy `Rising` · Mood `Upbeat`
- **Hormone line:** "Estrogen's climbing — energy, mood, and strength usually climb with it. A good window to push if you want to."
- **Food groups:**
  - **Protein** *(supports hormone synthesis & the training you can handle now)* — eggs, chicken, Greek yogurt, fish.
  - **Complex carbs** *(fuel the higher output)* — oats, quinoa, brown rice, wholegrain bread.
  - **Fresh & fermented** *(matches the light, growing energy)* — leafy greens, berries, sauerkraut, sprouts.
  - **Nourishing (TCM)** *(builds blood & yin after the bleed)* — goji berries, black sesame, beetroot, dark leafy greens. *(TCM: replenish blood and qi lost in menstruation; fresh, lighter cooking suits this phase.)*
- **Sleep:** "Usually your easiest sleep of the month — estrogen supports melatonin and body temp is low. A good stretch to bank rest before the luteal dip."
- **Training:** "Often your strongest window — a natural time to add load or volume if you feel it."

### 5.3 Ovulation

- **Chips:** Energy `Peak` · Mood `Confident`
- **Hormone line:** "Estrogen peaks and testosterone gives a small bump — often a real energy and strength high point. (A little spotting here is normal.)"
- **Food groups:**
  - **Antioxidants & zinc** *(support egg release, counter oxidative stress)* — berries, bell pepper, oysters, pumpkin seeds.
  - **Fiber** *(helps the liver clear the estrogen peak)* — cruciferous veg, flaxseed, beans, oats.
  - **Anti-inflammatory fats** *(smooth the surge)* — salmon, walnuts, olive oil, avocado.
  - **Moving qi (TCM)** *(supports the yin-to-yang shift)* — citrus peel, mint, lightly cooked greens. *(TCM: ovulation is the pivot from yin to yang; keep qi and blood moving.)*
- **Sleep:** "Sleep is still solid but body temp starts ticking up. Nothing special needed — keep the routine that's working."
- **Training:** "Peak-strength window for many — a fine time for a heavier session if you want one."

### 5.4 Luteal

- **Chips:** Energy `Dipping` · Mood `Sensitive`
- **Hormone line:** "Progesterone's high, estrogen's low. Expect lower energy, cravings, and maybe cramps or mood swings. Your body's also burning a bit more, so eat a little more — it's normal."
- **Food groups:**
  - **Magnesium** *(eases cramps & cravings)* — dark chocolate, pumpkin seeds, spinach, almonds, black beans.
  - **Vitamin B6** *(steadies mood & PMS)* — banana, chickpeas, salmon, potato.
  - **Complex carbs & fiber** *(curb hunger, less bloating)* — sweet potato, oats, brown rice, broccoli.
  - **Warming (TCM)** *(supports yang)* — red date tea, ginger tea, cinnamon, bone broth. *(TCM: luteal is the yang-dominant half; warming foods support yang, jujube/red date nourishes blood and qi.)*
- **Sleep:** "Body runs warmer now, so sleep often gets patchy before your period. Keep the room cool, wind down earlier, and ease off caffeine after noon."
- **Training:** "Energy may dip — keeping volume lighter is completely fine. Nothing is auto-applied."

### 5.5 TCM group labels per phase

The final food group each phase is the TCM one, always suffixed "(TCM)" on screen:

| Phase | TCM label | TCM rationale (short) |
|---|---|---|
| Menstrual | Warming (TCM) | Move blood & qi; warm over cold/raw. |
| Follicular | Nourishing (TCM) | Rebuild blood, yin, and essence after the bleed. |
| Ovulation | Moving qi (TCM) | Support the yin→yang transition; keep qi/blood moving. |
| Luteal | Warming (TCM) | Yang-dominant half; support yang, nourish blood. |

### 5.6 Evidence notes (internal — the basis for the copy above)

Kept here so future edits stay grounded. Not shown verbatim in-app.

- **Iron in menstrual phase:** iron requirements rise with menstrual blood loss; pairing with vitamin C aids absorption. Well supported.
- **Magnesium & B6 in luteal:** serum magnesium measurably declines from early-follicular to mid-luteal, and magnesium-deficiency prevalence rises in the luteal phase; magnesium plus B6 is a commonly evidenced combination for PMS symptom relief. Well supported.
- **Complex carbs / fiber in luteal:** luteal metabolism runs higher (resting energy expenditure up ~7% with the ~0.3–0.5°C temperature rise), appetite and cravings increase; complex carbs and fiber help steady mood/energy and curb hunger. Supported.
- **Follicular protein / carbs / fats:** support hormone synthesis and follicle growth; higher-output window. Supported.
- **Ovulation antioxidants / zinc / anti-inflammatory fats / fiber:** support egg release, counter ovulation-related oxidative stress, and aid estrogen clearance. Moderate support.
- **Sleep:** luteal progesterone raises core body temperature and blunts the temperature rhythm, suppressing melatonin and reducing REM/sleep efficiency; subjective sleep quality is lowest premenstrually and around menses. Cooler sleep environment, earlier wind-down, and reduced afternoon caffeine follow from this. Well supported. Follicular sleep is easiest (low temp, estrogen supports melatonin).
- **TCM groups:** traditional framework (yin/yang/qi/blood across the four phases). Labelled "(TCM)" on screen. Included because the user values it (red date and ginger tea are her own pre-period go-to's) and the foods are benign; framed as traditional, not clinical.
- **Cycle-phase *training* prescription overall:** evidence remains mixed and low-quality (carried from `STRENGTH_SCIENCE.md` §5.1); all training lines are soft, display-only, and never auto-applied.

---

## 6. Dashboard cycle card — spec

Progressive disclosure, consistent with `STRENGTH_SCIENCE.md` §5.2 and `DESIGN_SYSTEM.md`.

### 6.1 Collapsed (dashboard glance)

- Phase name + cycle day (e.g. "Luteal · day 22").
- One sub-line: days until next period, "est." (e.g. "Period in ~9 days").
- The phase color dot uses the matching `--phase-*` token. **Never color alone** — the phase name is always spelled out (`DESIGN_SYSTEM.md` §7).

### 6.2 Expanded (tap)

A per-phase **character/graphic** at the top (a small friendly illustration whose pose reflects the phase — resting for menstrual, energized for follicular, glowing/peak for ovulation, sleepy/mug-in-hand for luteal), then, in order:

1. **Two state chips** — Energy and Mood (§5), so the takeaway lands before any reading.
2. **"What's going on"** — the one-line hormone summary.
3. **"Eat for this phase"** — the nutrient-grouped food chips. The last group is the TCM group, labelled "(TCM)".
4. **"Rest & sleep"** — its own section (per user decision), the phase's sleep note.
5. **Training** — its own separate section, the soft display-only note.

Tone throughout: short, fun, practical. Phrases like "Expected low energy" over clinical sentences. **Standard typography** — sentence case, regular weight; no all-caps or letter-spaced labels. The only sourcing label shown is the "(TCM)" suffix on the traditional food group; everything else is presented plainly.

### 6.3 Framing rules (always applied)

- Every prediction shows "est.".
- Nothing on this card changes a training load or is auto-applied; the training line is permission-framed ("is completely fine"), never instruction.
- A small standing line is acceptable: "Suggestions, not prescriptions. Phase is estimated from your own history and improves as you track."

---

## 7. Data-model reconciliation (`DATA_MODEL.md` §10.1)

Confirms the stubbed cycle entities; no schema change required.

- **`CycleEntry`** `{ id, date, flow (none/spotting/light/medium/heavy), symptoms (string[]), note }` — the user logs period days, spotting days, flow level, and symptoms. Spotting is a first-class `flow` value (it clusters pre-period and sometimes at ovulation — useful signal, not noise).
- **`CyclePrediction`** `{ id, date, phase, cycleDay, predictedPeriodStart, method, confidence }`:
  - `method` = `rule_based` in v1.
  - `confidence` populated per §3.4 (lower when cycle-length variability is high).
  - `phase` / `cycleDay` are the values `LoggedSession.context` and `MigraineEntry.context` read for cross-domain correlation.
- Rolling `avgCycleLength` / `avgPeriodLength` are **computed on read** from logged `CycleEntry` history (not stored as their own entity), mirroring how weekly training aggregates work (`DATA_MODEL.md` §7).

---

## 8. Deliberately deferred

- **Pattern-learning prediction** — `method: 'learned'` in the schema anticipates it. v1 is `rule_based` only. Once enough in-app cycles accrue, the rolling average and variability model can graduate to a learned per-user pattern (`PRD.md` §10, `ARCHITECTURE.md` §10).
- **Cycle → performance / symptom correlations surfaced as confirmed patterns** — e.g. "your logged RPE-at-load tends to run higher in luteal," or cycle-phase × migraine associations. Depends on accumulated data and the Insights correlation engine (`STRENGTH_SCIENCE.md` §5.2, `MIGRAINE_LOGIC.md`). Framed as associations, never causes (`PRD.md` §8).
- **Symptom-informed phase adjustment** — using logged symptoms/spotting to nudge phase boundaries, beyond flow-driven menstrual detection. Kept simple for v1.
