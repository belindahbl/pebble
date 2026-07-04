# Visual Asset Guide — Personal Holistic Health App

**Version:** 1.2  
**Status:** Draft for implementation  
**Primary audience:** Claude Code / frontend implementation  
**Related documents:**

- `DESIGN_SYSTEM.md` — color tokens, typography, spacing, navigation, components, accessibility.
- `PRD.md` — product modules, five top-level destinations, dashboard card set, logging-first principles.
- `ARCHITECTURE.md` — responsive layout, offline-first app shape, static hosting constraints.
- `DATA_MODEL.md` — entities that drive labels, log types, cycle phases, migraine severity, and training states.
- `STRENGTH_SCIENCE.md` — training-readiness and load-reference behavior.

---

## 1. Purpose

This guide defines how custom illustrations, mascot states, icons, chips, and visual labels should be used across the app.

The goal is to make the app feel personal, calm, and encouraging without compromising speed, clarity, accessibility, or the low-noise design direction defined in `DESIGN_SYSTEM.md`.

The visual system should support the product experience. It should never become decoration for its own sake.

---

## 2. Design direction

The app uses an **Alpine wellness** visual language:

- Cool, calm, natural, and low eye-strain.
- Rounded, soft, and supportive rather than loud or gamified.
- Inspired by mountains, evergreen forests, mist, quiet recovery, and steady training.
- Friendly enough to make logging feel less clinical.
- Restrained enough to remain usable during migraines, cramps, fatigue, or low-energy days.

### Visual keywords

Use these words as implementation anchors:

- Calm
- Grounded
- Alpine
- Soft vector
- Low contrast
- Gentle movement
- Rounded
- Humanist
- Supportive
- Log-first

### Avoid these words as implementation anchors

Do not steer the interface toward:

- Neon
- Cartoonish
- Busy
- Competitive dashboard
- Medical app / hospital UI
- Fitness bro aesthetic
- High-contrast gamification
- Dense analytics-first layout

---

## 3. Reference assets

Store the generated visual references in the repository under `assets/design-references/`.

These files are **style references**, not production UI backgrounds. Claude Code should use them to infer illustration style, spacing, mood, and component treatment, then implement the actual app with reusable components, CSS tokens, SVG icons, and accessible text.

| Reference image | Repository path | Purpose |
|---|---|---|
| Pebble character sheet | `assets/design-references/pebble-character-sheet.png` | Defines mascot shape, personality, illustration style, and wellness states. |
| Icon system sheet | `assets/design-references/pebble-icon-system.png` | Defines navigation icons, quick-log icons, chips, severity labels, and mascot mood icons. |
| Home UI light/dark mockup | `assets/design-references/home-ui-light-dark-reference.png` | Defines overall UI feel, card rhythm, dark-mode treatment, and how illustration integrates into the dashboard. |

### Implementation rule

Use the reference assets for visual direction only. Do **not** hard-code screenshots as UI. Do **not** use the mockup image as a background. Build the interface from components.

---

## 4. Asset folder structure

Use this folder structure for production assets.

```text
assets/
  design-references/
    pebble-character-sheet.png
    pebble-icon-system.png
    home-ui-light-dark-reference.png

  mascots/
    pebble-balanced.png
    pebble-energized.png
    pebble-recovery.png
    pebble-low-energy.png
    pebble-migraine.png
    pebble-period-cramps.png

  icons/
    nav/
      home.png
      calendar.png
      train.png
      insights.png
      cycle.png

    quick-log/
      workout.png
      migraine.png
      period.png
      sleep.png
      hydration.png
      mood.png
      energy.png
      recovery.png
      mobility.png
      wellness.png

    status/
      success.svg
      warning.svg
      danger.svg
      on-track.svg
      fatigue-low.svg
      fatigue-moderate.svg
      fatigue-high.svg

    cycle/
      menstrual.png
      follicular.png
      ovulation.png
      luteal.png

    migraine/
      severity-1.png
      severity-2.png
      severity-3.png

  empty-states/
    no-logs-today.svg
    no-workout-planned.svg
    no-insights-yet.svg

  celebrations/
    workout-logged.svg
    pr-achieved.svg
```

### File naming conventions

- Use lowercase kebab-case.
- Use descriptive names.
- Use the generated `.png` assets for the current implementation.
- Prefer `.svg` for future production redraws of simple icons when the UI is stable.
- Keep the same base filenames if assets are later redrawn as SVG, for example `home.png` can later become `home.svg`.
- Do not include spaces in asset filenames.

Good:

```text
pebble-period-cramps.png
severity-2.png
hydration.png
```

Avoid:

```text
Pebble Period Cramps Final.png
migraineIconNEW.svg
image 4.png
```

---

## 4.1 Generated asset manifest

These are the current individual production assets generated in Batches 1–3. Upload them using the exact paths below. Claude Code should reference these named files directly instead of relying on the larger reference boards.

### Batch 1 — Navigation and first quick-log icons

```text
assets/icons/nav/home.png
assets/icons/nav/calendar.png
assets/icons/nav/train.png
assets/icons/nav/insights.png
assets/icons/nav/cycle.png

assets/icons/quick-log/workout.png
assets/icons/quick-log/migraine.png
assets/icons/quick-log/period.png
assets/icons/quick-log/sleep.png
assets/icons/quick-log/hydration.png
```

### Batch 2 — Remaining quick-log, cycle, and severity starter icons

```text
assets/icons/quick-log/mood.png
assets/icons/quick-log/energy.png
assets/icons/quick-log/recovery.png
assets/icons/quick-log/mobility.png
assets/icons/quick-log/wellness.png

assets/icons/cycle/menstrual.png
assets/icons/cycle/follicular.png
assets/icons/cycle/ovulation.png
assets/icons/cycle/luteal.png
```

### Batch 3 — Pika-designed migraine severity and mascot assets

Use the revised pika-designed severity icons below. Do **not** use the earlier human/lady migraine severity icons.

```text
assets/icons/migraine/severity-1.png
assets/icons/migraine/severity-2.png
assets/icons/migraine/severity-3.png

assets/mascots/pebble-balanced.png
assets/mascots/pebble-energized.png
assets/mascots/pebble-recovery.png
assets/mascots/pebble-low-energy.png
assets/mascots/pebble-migraine.png
assets/mascots/pebble-period-cramps.png
```

### Implementation rule

When an asset exists in this manifest, use the file from the manifest. Only use placeholder icons for assets that are not yet generated, such as status, empty-state, or celebration illustrations.

---

## 5. Mascot system

The primary mascot is **Pebble**, a small alpine pika companion.

Pebble is a soft, calm, pika-inspired alpine wellness guide. Pebble is a small mountain pika with a rounded pebble-like silhouette, soft sage-gray and cream fur, small rounded ears, tiny paws, and a gentle observant expression. Pebble should feel like a quiet companion rather than a loud coach.

### Mascot personality

Pebble is:

- Gentle
- Grounded
- Encouraging
- Non-judgmental
- Calm under fatigue or discomfort
- Supportive of both training and rest

Pebble is not:

- A medical authority
- A diagnosis character
- A hyperactive fitness coach
- A childish cartoon sticker
- A reward mascot that appears after every small action

---

## 6. Mascot states

Use mascot states to create emotional warmth and quick recognition. Do not overuse them.

| State | Asset | Use when | UI placement |
|---|---|---|---|
| Balanced | `assets/mascots/pebble-balanced.png` | Default home hero, neutral good day, no major risk flags. | Home hero card, wellness summary, empty state. |
| Energized | `assets/mascots/pebble-energized.png` | Energy is logged high, workout planned, PR or strong training trend. | Training card, workout completed state, positive insight. |
| Recovery | `assets/mascots/pebble-recovery.png` | Recovery day, high soreness, rest emphasis, post-migraine or low readiness. | Home card, wellness card, recovery guidance. |
| Low energy | `assets/mascots/pebble-low-energy.png` | Low energy, poor sleep, high fatigue, user logs low wellness. | Wellness card, fatigue expanded view. |
| Migraine | `assets/mascots/pebble-migraine.png` | Migraine logged or migraine watch card is active. | Migraine watch card, migraine logging confirmation. |
| Period cramps | `assets/mascots/pebble-period-cramps.png` | Menstrual phase with cramps or period symptoms logged. | Cycle card, symptom summary, period logging confirmation. |

### Mascot usage rules

- Use at most **one large mascot illustration per screen**.
- Do not place mascots in every card.
- Do not animate mascots in v1.
- Do not use mascots as buttons unless the button has a clear text label.
- Always pair mascot-led states with readable text.
- Use calm, supportive copy. Avoid shame, urgency, or fear.

Good:

```text
Balanced
Body, mind, and energy feel steady today.
```

Good:

```text
Recovery may be useful today
Your recent training load is high. Keeping things light is completely fine.
```

Avoid:

```text
Warning! Your body is failing!
You should not train today.
```

---

## 7. Navigation icons

The app has exactly five top-level destinations. These are fixed by `PRD.md` and `DESIGN_SYSTEM.md`.

| Destination | Icon | Asset | Label |
|---|---|---|---|
| Home | Alpine cabin / home with mountain cue | `assets/icons/nav/home.png` | Home |
| Calendar | Calendar grid with small natural marker | `assets/icons/nav/calendar.png` | Calendar |
| Train | Kettlebell / mountain training mark | `assets/icons/nav/train.png` | Train |
| Insights | Chart with small leaf or mountain cue | `assets/icons/nav/insights.png` | Insights |
| Cycle | Circular arrows with leaf / cycle mark | `assets/icons/nav/cycle.png` | Cycle |

### Navigation rules

- Phone: use bottom tab bar with icon + label.
- Laptop: use left sidebar with the same icon + label set.
- Active destination: use `--evergreen`.
- Inactive destinations: use `--text-muted`.
- Do not introduce nested tab bars inside a destination.
- Do not add a sixth top-level destination in v1.

---

## 8. Quick-log icons

Quick logging is central to the product. Icons should help the user log quickly without needing to read dense text.

| Log type | Icon concept | Asset | Data/entity relationship |
|---|---|---|---|
| Workout | Dumbbell or small weight | `assets/icons/quick-log/workout.png` | Strength `LoggedSession`; also entry point to Train. |
| Migraine | Head silhouette with soft bolt/cloud | `assets/icons/quick-log/migraine.png` | `MigraineEntry`. |
| Period | Droplet | `assets/icons/quick-log/period.png` | `CycleEntry`. |
| Sleep | Crescent moon | `assets/icons/quick-log/sleep.png` | `WellnessEntry.sleep`. |
| Hydration | Water droplet | `assets/icons/quick-log/hydration.png` | `HydrationEntry`. |
| Mood | Simple calm face | `assets/icons/quick-log/mood.png` | Optional wellness/mood extension. |
| Energy | Soft lightning bolt | `assets/icons/quick-log/energy.png` | `WellnessEntry.energy`. |
| Recovery | Heart with leaf | `assets/icons/quick-log/recovery.png` | Recovery/wellness state; may map to soreness or rest. |
| Mobility | Stretching figure | `assets/icons/quick-log/mobility.png` | Other workout or recovery/mobility log. |
| Wellness | Leaf-heart hybrid | `assets/icons/quick-log/wellness.png` | `WellnessEntry`: sleep, stress, energy, soreness. |

### Quick-log rules

- Each quick-log control must include both icon and text label.
- Minimum tap target is 44 × 44 px.
- Quick-add should remain reachable from Home.
- Logging flows should be reachable in 3 taps or fewer where possible.
- Do not rely on icon-only controls for migraine or period logging.

---

## 9. Cycle phase visuals

Cycle phase colors are semantic and must match `DESIGN_SYSTEM.md`.

| Phase | Token | Icon | Chip label | Asset |
|---|---|---|---|---|
| Menstrual | `--phase-menstrual` | Droplet | Menstrual | `assets/icons/cycle/menstrual.png` |
| Follicular | `--phase-follicular` | Sprout / leaf | Follicular | `assets/icons/cycle/follicular.png` |
| Ovulation | `--phase-ovulation` | Soft sun / bloom | Ovulation | `assets/icons/cycle/ovulation.png` |
| Luteal | `--phase-luteal` | Autumn leaf / inward leaf | Luteal | `assets/icons/cycle/luteal.png` |

### Cycle visual rules

- Always pair phase color with a text label.
- Do not rely on color alone.
- Use phase chips in Calendar, Home cycle card, and Cycle screen.
- Use the same phase color consistently across all screens.
- Do not invent alternate colors for cycle phases.
- If a phase is uncertain, show confidence in text rather than changing the color system.

Good:

```text
Follicular · day 7
```

Good:

```text
Predicted period in 6 days
```

Avoid:

```text
Green phase
Purple phase
```

---

## 10. Migraine severity visuals

Migraine severity uses a single-hue intensity scale. Severity should read as intensity, not as unrelated categories.

| Severity | Token | Label | Asset | Visual treatment |
|---|---|---|---|---|
| 1 | `--severity-1` | Mild | `assets/icons/migraine/severity-1.png` | Light tint, one dot or low pulse. |
| 2 | `--severity-2` | Moderate | `assets/icons/migraine/severity-2.png` | Medium tint, two dots or medium pulse. |
| 3 | `--severity-3` | Severe | `assets/icons/migraine/severity-3.png` | Darker tint, three dots or stronger pulse. |

### Migraine visual rules

- Always show the severity label: Mild, Moderate, Severe.
- Do not use alarming red flashing visuals.
- Do not use migraine visuals as diagnosis language.
- Use low-contrast, migraine-safe layouts.
- Avoid decorative animation, pulsing, or shaking effects.
- Keep logging screens especially quiet and readable.

Good:

```text
Migraine logged · Moderate
```

Good:

```text
Possible pattern: migraines often appear after lower hydration days.
```

Avoid:

```text
Cause detected
Danger migraine event
```

---

## 11. Home dashboard visual mapping

The Home screen is the app's summary and quick-entry surface. It should be glanceable and calm.

Use these visual mappings for Home dashboard cards.

| Home card | Visual asset | Usage notes |
|---|---|---|
| Hero / daily state | `pebble-balanced.png`, `pebble-low-energy.png`, or `pebble-recovery.png` | Show one large mascot state based on the most relevant wellness context. |
| Cycle status | Cycle phase icon + phase chip | Show current phase, cycle day, and next period prediction. |
| Today's training | Train icon; optional Pebble energized/recovery small illustration | Show planned split or prompt to plan/log. Do not imply load is automatically changed. |
| Strength highlight | PR icon or small mountain/chart icon | Use for recent PRs or progress trends. |
| Migraine watch | Migraine icon or `pebble-migraine.png` if active | Show days since last migraine and possible risk association. |
| Hydration | Hydration droplet icon | Show today's intake and one-tap add. |
| Wellness | Mood/energy/sleep icons | Show sleep, stress, energy, soreness summary or prompt to log. |
| Quick-add | Quick-log icons | Persistent entry point for fast logging. |

### Hero-state selection logic

Use this priority order when choosing a Home hero mascot:

1. Active migraine or migraine logged today → `pebble-migraine.png`.
2. Menstrual phase with cramps logged → `pebble-period-cramps.png`.
3. High systemic fatigue or low energy / poor sleep → `pebble-low-energy.png` or `pebble-recovery.png`.
4. Workout completed, energy high, or PR achieved → `pebble-energized.png`.
5. Default state → `pebble-balanced.png`.

The mascot state should support the data shown on screen. It should not contradict the logged state.

---

## 12. Train screen visual mapping

The Train destination covers splits, planning, exercise library, and logging.

### Use these assets

- `assets/icons/nav/train.png` for the destination.
- `assets/icons/quick-log/workout.png` for workout logging.
- `assets/mascots/pebble-energized.png` for positive training states.
- `assets/mascots/pebble-recovery.png` for recovery-oriented states.
- `assets/icons/status/fatigue-low.svg`, `fatigue-moderate.svg`, `fatigue-high.svg` for systemic-fatigue display.

### Training-readiness implementation rule

Follow `STRENGTH_SCIENCE.md`: readiness signals are display-only in v1.

- Systemic fatigue may be shown as Low / Moderate / High.
- Cycle phase may be shown as context.
- Load references may be shown in the logging flow.
- The app should not automatically prescribe heavier or lighter loads in v1.
- Avoid UI copy that says the app has changed the user's training load.

Good:

```text
Systemic fatigue: Moderate
Recent hard sessions may still be contributing to tiredness.
```

Good:

```text
RPE 7–8 reference: 45–48 kg based on your recent e1RM.
```

Avoid:

```text
The app reduced your squat by 10% today.
```

---

## 13. Insights screen visual mapping

Insights should be clear and analytical, but not visually cold.

### Use these assets

- `assets/icons/nav/insights.png` for the destination.
- Chart icons for section headers.
- Small phase chips for cycle-related filters.
- Migraine severity chips for migraine trend breakdowns.
- Avoid large mascot use unless the screen is empty or celebrating a meaningful achievement.

### Insights rules

- Prioritize charts, labels, and plain-language explanations.
- Use mascots only for empty states or milestone moments.
- Present migraine relationships as associations, not causes.
- Keep color meaning consistent with cycle, severity, and status tokens.

Good:

```text
Association found
Migraine entries appear more often on days after lower hydration.
```

Avoid:

```text
Hydration caused your migraine.
```

---

## 14. Calendar screen visual mapping

The Calendar destination is the primary way to move through time and view per-day markers.

### Calendar markers

Use small, quiet markers rather than dense labels inside each day cell.

| Marker | Icon/color |
|---|---|
| Cycle phase | Small phase-colored dot or phase icon. |
| Workout logged/planned | Train icon or small evergreen marker. |
| Migraine | Migraine icon or severity-colored dot. |
| Hydration logged | Small hydration droplet. |
| Wellness logged | Small leaf or mood icon. |

### Calendar rules

- Do not overload each date cell.
- Use the day-detail sheet for full information.
- Tapping a day opens view/add actions.
- Always keep labels available in the day-detail view.

---

## 15. Empty states and celebration states

Use illustrations sparingly to make the app feel personal.

| State | Asset | Copy direction |
|---|---|---|
| No logs today | `assets/empty-states/no-logs-today.svg` | Gentle prompt to log one thing. |
| No workout planned | `assets/empty-states/no-workout-planned.svg` | Invite planning or ad-hoc logging. |
| No insights yet | `assets/empty-states/no-insights-yet.svg` | Explain that insights need more logs. |
| Workout logged | `assets/celebrations/workout-logged.svg` | Quiet confirmation, no confetti overload. |
| PR achieved | `assets/celebrations/pr-achieved.svg` | Warm celebration using `--success`; no aggressive gamification. |

### Empty-state rule

Empty states should reduce friction, not make the user feel behind.

Good:

```text
No logs yet today
Start with one quick entry.
```

Avoid:

```text
You have not logged anything. Your data is incomplete.
```

---

## 16. Color implementation rules

Use the tokens from `DESIGN_SYSTEM.md`.

Do not hard-code random colors in components if a design token exists.

### Required token usage

| UI element | Token |
|---|---|
| Primary actions | `--evergreen` |
| Secondary actions / links | `--slate-blue` |
| Calm highlights | `--sage` |
| Attention-only accent | `--accent-sand` |
| Cards | `--surface-1` |
| Secondary fills | `--surface-2` |
| Borders | `--border` |
| Main text | `--text-primary` |
| Supporting text | `--text-secondary` |
| Muted labels | `--text-muted` |

### Accent-sand rule

Use `--accent-sand` sparingly for:

- Migraine attention details.
- Pain or cramp emphasis.
- Warnings that need visibility.
- Small highlights in illustrations.

Do not use accent-sand as a general brand color.

---

## 17. Typography rules

Follow `DESIGN_SYSTEM.md` exactly.

- Use one humanist sans-serif stack unless otherwise specified by the build.
- Use only 400 and 500 weights.
- Use sentence case everywhere.
- Minimum font size is 13px.
- Do not introduce a decorative heading font in v1 unless explicitly approved.

### Text style mapping

| UI text | Style role |
|---|---|
| Screen title | Screen title |
| Card title | Section heading |
| Main body | Body |
| Labels and field names | Secondary/label |
| Helper text | Caption |

---

## 18. Accessibility rules

The app may be used while the user feels unwell. Accessibility is part of the product experience.

- Never rely on color alone.
- Pair icons with labels.
- Keep contrast readable in light and dark mode.
- Avoid flashing, pulsing, shaking, or decorative animation.
- Do not make migraine states visually intense.
- Maintain 44 × 44 px tap targets.
- Keep copy calm and non-alarming.
- Do not use tiny labels inside complex illustrations.

---

## 19. Dark mode rules

Dark mode is first-class.

- Use dark-mode tokens from `DESIGN_SYSTEM.md`.
- Do not simply invert light-mode illustrations.
- Use deeper teal/charcoal backgrounds and muted highlights.
- Keep borders visible but subtle.
- Keep colored chips readable.
- Use simplified illustration backgrounds in dark mode to avoid visual clutter.

For mascot illustrations, prefer either:

1. SVGs that adapt through CSS variables; or
2. Separate light/dark exports only if needed for readability.

---

## 20. Component implementation guidance

### Recommended component names

```text
AppShell
ResponsiveNavigation
BottomTabBar
SidebarNavigation
Card
QuickAddButton
QuickAddSheet
IconButtonWithLabel
PhaseChip
SeverityChip
MascotHeroCard
DashboardMetricCard
CalendarDayMarker
EmptyState
CelebrationBanner
```

### Recommended asset helpers

```text
getMascotForHomeState(context)
getCyclePhaseIcon(phase)
getCyclePhaseLabel(phase)
getCyclePhaseToken(phase)
getMigraineSeverityLabel(severity)
getMigraineSeverityIcon(severity)
getQuickLogIcon(entryType)
```

### Example mascot resolver

```js
function getMascotForHomeState(context) {
  if (context.hasMigraineToday) return 'auro-migraine';
  if (context.hasPeriodCrampsToday) return 'auro-period-cramps';
  if (context.systemicFatigue === 'High') return 'auro-recovery';
  if (context.energy != null && context.energy <= 2) return 'auro-low-energy';
  if (context.workoutCompletedToday || context.hasPrToday) return 'auro-energized';
  return 'auro-balanced';
}
```

This function is illustrative. Final implementation should use the actual app state and data model.

---

## 21. Copy and labels

Use calm, direct, non-judgmental copy.

### Approved top-level labels

```text
Home
Calendar
Train
Insights
Cycle
```

### Approved quick-log labels

```text
Workout
Migraine
Period
Sleep
Hydration
Mood
Energy
Recovery
Mobility
Wellness
```

### Approved cycle labels

```text
Menstrual
Follicular
Ovulation
Luteal
```

### Approved migraine severity labels

```text
Mild
Moderate
Severe
```

### Approved fatigue labels

```text
Low
Moderate
High
```

### Copy tone examples

Use:

```text
One calm step at a time.
```

```text
Energy looks steady today.
```

```text
Recovery may be useful today.
```

```text
Log what matters. Skip what does not.
```

Avoid:

```text
You failed to log.
```

```text
Bad recovery score.
```

```text
Your migraine was caused by training.
```

```text
You must deload today.
```

---

## 22. Do-not rules

Claude Code must not:

- Add new top-level destinations.
- Add nested tab bars inside destinations.
- Use generated mockup screenshots as UI backgrounds.
- Rely on color alone to convey cycle phase, migraine severity, or status.
- Use decorative animation in v1.
- Use aggressive fitness language.
- Use diagnosis language.
- Present migraine correlations as causes.
- Present cycle phase or systemic fatigue as automatic load instructions.
- Use accent-sand as a general decorative color.
- Add more mascot appearances when a simple icon would be clearer.
- Replace app data labels with vague mood labels.

---

## 23. Claude Code implementation prompt

Use this prompt when asking Claude Code to implement or revise UI.

```text
Use DESIGN_SYSTEM.md, PRD.md, ARCHITECTURE.md, DATA_MODEL.md, STRENGTH_SCIENCE.md, and VISUAL_ASSET_GUIDE.md as the source of truth.

Implement the app UI in the Alpine wellness style:
- Calm, low-eye-strain, natural, rounded, and log-first.
- Use the color tokens and component rules from DESIGN_SYSTEM.md.
- Use exactly five top-level destinations: Home, Calendar, Train, Insights, Cycle.
- Use bottom tabs on phone and a left sidebar on laptop.
- Use the visual asset guide for mascot states, icon names, labels, chips, and dashboard visual mapping.
- Reference images in assets/design-references/ are style anchors only. Do not use them as UI backgrounds.
- Build reusable components rather than one-off layouts.
- Keep dark mode first-class.
- Keep migraine and period-related screens especially calm and readable.
- Follow STRENGTH_SCIENCE.md: training readiness is display-only in v1; do not automatically change loads.

Before coding, briefly list the components you will create or modify. Then implement the smallest coherent UI slice first.
```

---

## 24. Acceptance criteria

The implementation is acceptable when:

- The app visually matches the Alpine wellness direction.
- Navigation uses only Home, Calendar, Train, Insights, Cycle.
- Bottom tabs appear on phone and sidebar appears on laptop.
- Cards, buttons, chips, and sheets follow `DESIGN_SYSTEM.md`.
- Quick-add uses icon + label controls.
- Cycle phases use the correct semantic labels and colors.
- Migraine severity uses Mild / Moderate / Severe with a single-hue intensity scale.
- Mascots are used sparingly and contextually.
- Dark mode is readable and not an afterthought.
- No medical-diagnosis, causation, or automatic-load-prescription language appears.
- The UI remains usable without any images loaded, because labels and structure still communicate meaning.

---

## 25. Future asset work

After the first UI implementation, export final production assets individually:

1. Navigation icons as SVG.
2. Quick-log icons as SVG.
3. Cycle phase icons as SVG.
4. Migraine severity icons as SVG.
5. Mascot states as SVG or optimized transparent PNG.
6. Empty states and celebration illustrations as SVG.

Production assets should be optimized before shipping:

- Remove unused metadata.
- Use accessible filenames.
- Keep file sizes small.
- Ensure light and dark mode compatibility.
- Confirm icons remain legible at 24 px.
- Confirm mascot illustrations remain clear inside cards on phone.
