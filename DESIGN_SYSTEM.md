# Design System — Personal Holistic Health App

**Version:** 1.0 (v1 scope)

## Related documents
- `PRD.md` — navigation destinations and dashboard card set.
- `ARCHITECTURE.md` — responsive layout requirement.

---

## 1. Direction

"Alpine": cool, calm, natural, low eye-strain. Evergreen and slate-blue anchors evoke mountains; misty neutrals keep surfaces from feeling clinical; a warm accent is held in reserve for elements that must catch the eye. The app is often opened when the user feels unwell (migraine, cramps), so legibility and low visual noise take priority over decoration.

## 2. Color tokens

Defined as CSS custom properties. Every token has a light and dark value; the app supports both modes and follows the system preference by default with a manual override.

### Brand / anchors
| Token | Light | Dark | Use |
|---|---|---|---|
| `--evergreen` | #1F4D3D | #2E6B54 | Primary brand, primary actions |
| `--slate-blue` | #2E5A73 | #4A7C99 | Secondary brand, links, secondary actions |
| `--sage` | #7FA88C | #8FB89C | Calm highlight, subtle fills |
| `--accent-sand` | #D98555 | #E09A6F | Accent — alerts and attention only, used sparingly |

### Surfaces & text — light mode
| Token | Value | Use |
|---|---|---|
| `--bg` | #F4F6F3 | Page background |
| `--surface-1` | #FFFFFF | Cards, sheets |
| `--surface-2` | #E7EBE7 | Raised/secondary fills, input backgrounds |
| `--border` | #CDD5CE | Hairline borders, dividers |
| `--text-primary` | #2B322D | Body and headings |
| `--text-secondary` | #5A6560 | Supporting text |
| `--text-muted` | #8A968D | Hints, placeholders, captions |

### Surfaces & text — dark mode
| Token | Value | Use |
|---|---|---|
| `--bg` | #12181A | Page background |
| `--surface-1` | #1A2224 | Cards, sheets |
| `--surface-2` | #243033 | Raised/secondary fills, input backgrounds |
| `--border` | #3A4A4D | Hairline borders, dividers |
| `--text-primary` | #C7D2CC | Body and headings |
| `--text-secondary` | #9BAAA2 | Supporting text |
| `--text-muted` | #6E7E77 | Hints, placeholders, captions |

### Semantic — status
| Token | Light | Dark | Use |
|---|---|---|---|
| `--success` | #3B6D11 | #97C459 | Positive state, PR achieved |
| `--warning` | #BA7517 | #EF9F27 | Caution, risk flag |
| `--danger` | #A32D2D | #E24B4A | Errors, destructive actions |

### Semantic — cycle phases
Each menstrual phase has one assigned color, used consistently on the calendar, dashboard, and cycle screens.
| Token | Light | Dark | Phase |
|---|---|---|---|
| `--phase-menstrual` | #A32D2D | #E24B4A | Menstrual |
| `--phase-follicular` | #639922 | #97C459 | Follicular |
| `--phase-ovulation` | #1D9E75 | #5DCAA5 | Ovulation |
| `--phase-luteal` | #7159B8 | #A99BE0 | Luteal |

### Semantic — migraine severity
A single-hue scale (light → dark = mild → severe), so severity reads as intensity, not category.
| Token | Light | Dark | Severity |
|---|---|---|---|
| `--severity-1` | #F0997B | #F0997B | Mild |
| `--severity-2` | #D85A30 | #D85A30 | Moderate |
| `--severity-3` | #993C1D | #C24A22 | Severe |

Rule: text placed on any colored fill uses the darkest shade of that same color family (or `--text-primary` on light neutral fills), never pure black.

## 3. Typography

- **Typeface:** one humanist sans-serif for the whole app (system font stack acceptable for a dependency-light build; a single web font is permitted if chosen in build). No secondary display face in v1.
- **Two weights only:** 400 regular, 500 medium. Avoid 600/700.
- **Scale** (rem, 16px base):

| Role | Size | Weight | Line-height |
|---|---|---|---|
| Screen title | 1.5rem (24px) | 500 | 1.3 |
| Section heading | 1.125rem (18px) | 500 | 1.4 |
| Body | 1rem (16px) | 400 | 1.6 |
| Secondary/label | 0.875rem (14px) | 400 | 1.5 |
| Caption | 0.8125rem (13px) | 400 | 1.4 |

- **Minimum font size 13px.** Sentence case everywhere (titles, buttons, labels).

## 4. Spacing & layout

- **Spacing scale** (use these steps only): 4, 8, 12, 16, 24, 32, 48 px.
- Vertical rhythm between sections: 24px. Card internal padding: 16px. Gap between cards in a list/grid: 12px.
- **Corner radius:** 8px for controls (buttons, inputs), 12px for cards and sheets.
- **Borders:** hairline `--border`, 1px. Rounded corners only with full borders (no rounded single-side accents).
- **Tap targets:** minimum 44×44px for any interactive element.
- **Content max-width on laptop:** primary content column caps at 1100px; dashboard and Insights may use a two-column grid within that width. Phone is always single-column.

## 5. Navigation pattern

The app has exactly five fixed top-level destinations (Home, Calendar, Train, Insights, Cycle — defined in `PRD.md`). Navigation is one system, not nested tab bars.

- **Phone (< 768px):** bottom tab bar, five items, each with icon + short label. Always visible. Content area scrolls above it.
- **Laptop (≥ 768px):** left sidebar, same five items with icon + label. Content area fills the remaining width.
- **Depth within a destination:** handled by a pushed screen (slides in, with a back control) or a sheet (slides up for quick entry/edit). Never introduce a second row of tabs inside a destination.
- **Active state:** the current destination is marked with `--evergreen` (icon + label); inactive items use `--text-muted`.

## 6. Core components

Defined once and reused on every screen; consistency is the source of the app's UX quality.

- **Card:** `--surface-1` background, 1px `--border`, 12px radius, 16px padding. The dashboard, calendar day detail, and list items are all cards.
- **Button — primary:** `--evergreen` fill, light text, 8px radius, 44px min height. One primary action per screen.
- **Button — secondary:** transparent fill, 1px `--border`, `--text-primary` label.
- **Button — quiet/ghost:** no border, `--slate-blue` label; for low-emphasis actions.
- **Input / select / stepper:** `--surface-2` background, 1px `--border`, 8px radius, 44px height.
- **Quick-add control:** persistent on Home; opens a sheet listing loggable entry types (workout, hydration, wellness, migraine, cycle). Any single entry is loggable in ≤3 taps.
- **Chip / tag:** small rounded fill using a semantic color (cycle phase, severity) with same-family dark text.
- **Sheet:** slides up from bottom, `--surface-1`, 12px top corners; used for quick logging and edits.

## 7. Mode & accessibility rules

- Dark mode is a first-class mode, not an afterthought; every token above is defined for both. Default to system preference with a manual toggle.
- Never rely on color alone to convey meaning — pair cycle-phase and severity colors with a text label or icon.
- Maintain readable contrast: body text against its surface must remain legible in both modes (the tokens above are chosen to satisfy this).
- Motion is minimal and purposeful (push/sheet transitions only); no decorative animation.
