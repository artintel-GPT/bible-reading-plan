# Bible Reading Plan UI Redesign

**Date:** 2026-05-16  
**Status:** Approved  
**Scope:** Refine and upgrade `bible_reading_plans.html` into a full standalone page with persistent progress tracking, auto today-highlighting, and a warm parchment/gold visual theme.

---

## Goals

- Convert the embeddable HTML fragment into a self-contained standalone page
- Add `localStorage` persistence so progress survives page reloads
- Auto-highlight the current day based on a per-plan start date
- Apply a warm parchment/gold color palette and a serif heading for a more polished feel
- Preserve the existing layout structure (tabs → stats → progress bar → controls → card grid)

---

## Visual Design

### Color Palette (self-contained CSS custom properties)

| Variable | Value | Usage |
|---|---|---|
| `--bg` | `#fdf6e3` | Page background |
| `--bg-card` | `#fefaf0` | Card and stat box background |
| `--bg-done` | `#eef5e8` | Completed day card background |
| `--gold` | `#b5860d` | Active tabs, today border, stat values, progress fill |
| `--gold-light` | `#fdf0c0` | Today card background tint |
| `--text` | `#3b2f10` | Primary text |
| `--text-muted` | `#8a7045` | Secondary/muted text |
| `--green` | `#4a7c59` | Done state text and border |
| `--border` | `#e8d9b0` | Default card/input borders |
| `--border-done` | `#a8c9a0` | Done card border |
| `--shadow` | `rgba(100,70,20,0.08)` | Hover/card shadow |

### Typography

- Body: system sans-serif stack
- Page heading only: `Georgia, 'Times New Roman', serif` — built into iPadOS/macOS, no network request required
- All other type sizes and weights stay the same as current

---

## Page Structure

```
<!DOCTYPE html>
<html>
  <head>
    charset, viewport, <style> block (no external font link needed)
  </head>
  <body>
    <div class="container">
      <header>          ← NEW: cross icon + serif title + subtitle
      <div class="tabs">
      <div class="plan-header">
        plan title
        plan subtitle
        start-date line  ← NEW: "Started: May 10 · Change" or "Set start date" button
      </div>
      <div class="stats">
      <div class="progress-bar">
      <div class="controls">  ← search + Mark all done + Reset
      <div class="grid">
    </div>
  </body>
</html>
```

---

## Data & Persistence

### localStorage Keys (per plan index `i` = 0–3)

- `bible-plan-{i}-progress` — JSON-serialized boolean array, length = plan days
- `bible-plan-{i}-start` — ISO date string `"YYYY-MM-DD"`, absent if not set

### Behavior

- On page load: hydrate all four progress arrays from storage, falling back to all-false arrays
- On every toggle, `markAll`, or reset: immediately re-serialize and save the affected plan's progress
- Resetting a plan clears both its progress and its start date from storage
- Start date is set via an inline native `<input type="date">` (no modal, no library); shown inline when user clicks "Set start date" or "Change"

---

## Today Highlighting

### Calculation

```
todayIndex = floor((today - startDate) / 86400000)   // milliseconds per day
```

- If `todayIndex` is in range `[0, days-1]`: apply `.today` class to that card
- If `todayIndex >= days`: show *"Plan complete"* note next to start date line
- If `todayIndex < 0` (start date in future): show *"Starting in N days"* note

### `.today` Card State

- Background: `--gold-light`
- Border: `--gold` (1px solid)
- "Today" badge replaces "Day N" label in top-left
- On render: `card.scrollIntoView({ behavior: 'smooth', block: 'center' })`

---

## Card States Summary

| State | Background | Border | Text | Badge |
|---|---|---|---|---|
| Default | `--bg-card` | `--border` | `--text` | Day N |
| Hover | `--bg-card` | `--gold` (soft) | `--text` | Day N |
| Today | `--gold-light` | `--gold` | `--text` | **Today** |
| Done | `--bg-done` | `--border-done` | `--green` | Day N + ✓ |
| Today + Done | `--bg-done` | `--green` | `--green` | **Today** + ✓ |

---

## Component Details

### Header (new)
```html
<header>
  <span class="header-icon">✝</span>
  <div>
    <h1 class="header-title">Old Testament Reading Plans</h1>
    <p class="header-sub">Track your journey through the Old Testament</p>
  </div>
</header>
```

### Start Date Line (new, inside `.plan-header`)
- When no start date: `<button class="start-btn">Set start date</button>`
- When clicked: replaces button with `<input type="date">` inline; on change, saves date and re-renders
- When start date set: `Started: May 10, 2026 · <button class="link-btn">Change</button>`
- "Change" re-shows the date input

### Tabs
- Active tab: `--gold` border, `--gold` text, `--gold-light` background
- Inactive hover: `--bg-card` background

### Stats
- Box: `--bg-card` background, subtle box shadow
- Stat value: `--gold` color

### Progress Bar
- Height: 6px (up from 4px)
- Fill: `--gold`

### Buttons (Mark all done / Reset)
- Border: `--gold`
- Text: `--gold`
- Background: `--bg-card`
- Hover: `--gold-light` background

---

## Files Changed

- `bible_reading_plans.html` — full rewrite as standalone page (all logic preserved, CSS upgraded)

## Files Added

- `docs/superpowers/specs/2026-05-16-bible-reading-plan-ui-redesign.md` — this document

---

## Out of Scope

- New Testament plans
- User accounts or cloud sync
- Mobile app / PWA
- Notifications or reminders
- Verse content display
