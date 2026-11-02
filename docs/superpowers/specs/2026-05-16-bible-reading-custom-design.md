# Bible Reading Plan — Custom Version Design

**Date:** 2026-05-16  
**Status:** Approved  
**Scope:** New standalone file `bible_reading_custom.html` — a flexible Bible reading plan app with a plan builder, support for OT/NT/Full Bible, sequential/chronological order, custom day counts, and multiple simultaneous saved plans. The existing `bible_reading_plans.html` is left completely untouched.

---

## Goals

- Plan builder UI: pick corpus, order, days, and optional name before generating a plan
- Support OT, NT, and Full Bible corpora
- Support sequential and chronological order for all three corpora
- Custom day count: integer 7–365
- Multiple plans saved simultaneously as tabs
- Same parchment/gold theme, localStorage patterns, and card/stats/progress UI as the existing app
- Deployable to the same GitHub Pages repo alongside the existing file

---

## Page Structure

Layout top to bottom:

1. **Header** — ✝ icon + Georgia serif "Custom Bible Reading Plans" title + subtitle
2. **Plan builder** — inline form; always expanded when no plans exist; collapses to a "＋ New Plan" button after first plan is created
3. **Tabs row** — one tab per saved plan with a ✕ delete button; "＋ New Plan" button lives in the top-right of this row when builder is collapsed
4. **Plan display** — plan title, start date line, stats (3 boxes), progress bar, search + controls, card grid

---

## Data Model

Single `localStorage` key: `bible-custom-plans`

Stores a JSON array of plan objects:

```json
[
  {
    "id": "plan_1747423200000",
    "name": "My NT Plan",
    "corpus": "NT",
    "order": "sequential",
    "days": 30,
    "progress": [false, false, true],
    "start": "2026-05-16"
  }
]
```

| Field | Type | Notes |
|---|---|---|
| `id` | string | `"plan_" + Date.now()` — stable unique key |
| `name` | string | User-entered or auto-generated |
| `corpus` | `"OT"` \| `"NT"` \| `"Full"` | Determines chapter pool |
| `order` | `"sequential"` \| `"chronological"` | Determines book order |
| `days` | integer 7–365 | Plan length |
| `progress` | boolean[] | Length = days; toggled per card |
| `start` | string \| null | ISO date `"YYYY-MM-DD"` or null |

**Persistence:** Entire array re-serialized and saved (wrapped in try/catch) on every mutation: plan create, plan delete, card toggle, markAll, start date change.

**On load:** Hydrate from storage. If empty or missing, show builder expanded with no tabs.

---

## Plan Builder UI

Inline form with fields in a single row (wraps on narrow screens):

| Field | Type | Behavior |
|---|---|---|
| Corpus | 3-button toggle | `OT` / `NT` / `Full Bible`; one active (gold highlight) |
| Order | 2-button toggle | `Sequential` / `Chronological`; one active |
| Days | `<input type="number">` | min=7, max=365; red border + "Enter 7–365" hint if invalid |
| Name | `<input type="text">` | Optional; placeholder `"Plan name (optional)"` |
| Create | `<button>` | Gold; disabled when days is invalid; triggers plan creation |

**Auto-name format** (when name left blank): `"{Corpus} · {Order} · {days} days"` e.g. `"NT · Chronological · 30 days"`

**On Create:**
1. Validate days (7–365)
2. Build plan using `buildPlan(order, days)` with selected corpus pool
3. Push new plan object to plans array, save to localStorage
4. Switch active tab to new plan
5. Collapse builder

**Builder collapse/expand:**
- After any plan exists: builder collapses; a `＋ New Plan` button appears in the tabs row
- Clicking `＋ New Plan` expands the builder above the tabs
- Builder fields reset to defaults (OT, sequential, empty days/name) when expanded

---

## Bible Data

### OT (39 books, 929 chapters)

Same `OT_CHAPTERS` object and `SEQ_ORDER` / `CHRON_ORDER` arrays as the existing app.

### NT (27 books, 260 chapters)

```js
const NT_CHAPTERS = {
  Matthew:28, Mark:16, Luke:24, John:21, Acts:28,
  Romans:16, "1 Corinthians":16, "2 Corinthians":13,
  Galatians:6, Ephesians:6, Philippians:4, Colossians:4,
  "1 Thessalonians":5, "2 Thessalonians":3,
  "1 Timothy":6, "2 Timothy":4, Titus:3, Philemon:1,
  Hebrews:13, James:5, "1 Peter":5, "2 Peter":3,
  "1 John":5, "2 John":1, "3 John":1, Jude:1, Revelation:22
};
```

**NT Sequential order:** canonical (Matthew → Revelation)

**NT Chronological order** (by writing date):
```
James, Galatians, 1 Thessalonians, 2 Thessalonians,
1 Corinthians, 2 Corinthians, Romans, Philippians,
Colossians, Ephesians, Philemon, 1 Timothy, Titus,
2 Timothy, Hebrews, Matthew, Mark, Luke, Acts,
John, 1 Peter, 2 Peter, Jude, 1 John, 2 John, 3 John,
Revelation
```

### Full Bible (66 books, 1189 chapters)

**Sequential:** OT canonical order followed by NT canonical order (Genesis → Revelation)

**Chronological:** OT chronological order followed by NT chronological order

---

## Tab Management

- Tabs appear in creation order
- Each tab: plan name (truncated to 20 chars with ellipsis if longer) + ✕ button
- **Delete flow:** clicking ✕ replaces tab label inline with `"Delete [name]? [Confirm] [Cancel]"` — no browser `confirm()` dialog
- On confirm: remove plan from array, save, switch to nearest remaining tab (prefer left, then right)
- If no plans remain after delete: collapse tabs row, expand builder
- Active tab highlighted with gold border/text (same `.active` style as existing app)

---

## Plan Display

Identical to `bible_reading_plans.html` for everything below the tabs:

- **Plan title** — auto-generated or user-provided name
- **Start date line** — "Set start date" button → inline `<input type="date">` → "Started: [date] · Change" + status note
- **Stats** — total days, days completed, % complete (gold values)
- **Progress bar** — 6px gold fill
- **Controls** — search box, "Mark all done", "Reset" buttons
- **Card grid** — responsive, `minmax(220px, 1fr)`; today card gold-highlighted; done card sage green
- **Today logic** — same `getTodayIndex` calculation (local date, safe Safari parsing)
- **Scroll-to-today** — fires once per plan load, guarded by per-plan `scrolledToToday` flag
- **Reset** — clears progress and start date for active plan only

---

## Files

| File | Action |
|---|---|
| `bible_reading_custom.html` | Create — new standalone page |
| `bible_reading_plans.html` | Untouched |

---

## Out of Scope

- Reordering tabs
- Renaming existing plans
- Sharing or exporting plans
- New Testament plans in the original fixed-plan app
- Any server-side component
