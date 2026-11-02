# Bible Reading Plan UI Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `bible_reading_plans.html` as a full standalone page with a warm parchment/gold theme, `localStorage` persistence, per-plan start dates, and auto today-highlighting optimized for iPad Pro.

**Architecture:** Single self-contained HTML file — no build system, no external requests, no dependencies. All CSS, HTML, and JS live in one file. The existing plan-building and chapter-formatting logic is preserved intact; the CSS palette, HTML wrapper, and the three new features (persistence, start dates, today-highlighting) are added on top.

**Tech Stack:** Vanilla HTML5, CSS custom properties, vanilla JS, Web Storage API (`localStorage`), native `<input type="date">`.

---

## File Map

| File | Action | Responsibility |
|---|---|---|
| `bible_reading_plans.html` | Full rewrite | Standalone page: styles, markup, all logic |
| `docs/superpowers/specs/2026-05-16-bible-reading-plan-ui-redesign.md` | Already exists | Reference spec |

---

## Task 1: Write the complete new file

**Files:**
- Modify: `bible_reading_plans.html`

This is a full rewrite. The existing plan-building functions (`buildPlan`, `formatDay`, `totalChapters`) and data constants (`OT_CHAPTERS`, `SEQ_ORDER`, `CHRON_ORDER`, `PLANS`) are kept exactly as-is. Everything else is new or upgraded.

- [ ] **Step 1: Replace the entire contents of `bible_reading_plans.html` with the following**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Old Testament Reading Plans</title>
  <style>
    :root {
      --bg: #fdf6e3;
      --bg-card: #fefaf0;
      --bg-done: #eef5e8;
      --gold: #b5860d;
      --gold-light: #fdf0c0;
      --text: #3b2f10;
      --text-muted: #8a7045;
      --green: #4a7c59;
      --border: #e8d9b0;
      --border-done: #a8c9a0;
      --shadow: rgba(100, 70, 20, 0.08);
      --radius: 8px;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: var(--bg);
      color: var(--text);
      min-height: 100vh;
      padding: 1.5rem 1rem;
    }

    .container {
      max-width: 900px;
      margin: 0 auto;
    }

    /* ── Header ── */
    header {
      display: flex;
      align-items: center;
      gap: 14px;
      margin-bottom: 2rem;
      padding-bottom: 1.25rem;
      border-bottom: 1px solid var(--border);
    }

    .header-icon {
      font-size: 34px;
      color: var(--gold);
      line-height: 1;
      flex-shrink: 0;
    }

    .header-title {
      font-family: Georgia, "Times New Roman", serif;
      font-size: 22px;
      font-weight: normal;
      color: var(--text);
      letter-spacing: 0.01em;
    }

    .header-sub {
      font-size: 13px;
      color: var(--text-muted);
      margin-top: 3px;
    }

    /* ── Tabs ── */
    .tabs {
      display: flex;
      gap: 6px;
      flex-wrap: wrap;
      margin-bottom: 1.5rem;
    }

    .tab-btn {
      padding: 7px 14px;
      font-size: 13px;
      font-weight: 500;
      border-radius: var(--radius);
      border: 1px solid var(--border);
      background: var(--bg-card);
      color: var(--text-muted);
      cursor: pointer;
      transition: all 0.15s;
    }

    .tab-btn:hover {
      background: var(--gold-light);
      color: var(--text);
      border-color: var(--gold);
    }

    .tab-btn.active {
      background: var(--gold-light);
      color: var(--gold);
      border-color: var(--gold);
      font-weight: 600;
    }

    /* ── Plan header ── */
    .plan-header { margin-bottom: 1.25rem; }

    .plan-title {
      font-size: 18px;
      font-weight: 500;
      color: var(--text);
      margin-bottom: 4px;
    }

    .plan-sub {
      font-size: 13px;
      color: var(--text-muted);
      margin-bottom: 8px;
    }

    /* ── Start date ── */
    .start-date-line {
      display: flex;
      align-items: center;
      gap: 6px;
      font-size: 12px;
      color: var(--text-muted);
      flex-wrap: wrap;
      min-height: 28px;
    }

    .start-btn {
      font-size: 12px;
      padding: 5px 11px;
      border: 1px solid var(--border);
      border-radius: var(--radius);
      background: var(--bg-card);
      color: var(--text-muted);
      cursor: pointer;
      transition: border-color 0.15s, color 0.15s;
    }

    .start-btn:hover {
      border-color: var(--gold);
      color: var(--gold);
    }

    .link-btn {
      background: none;
      border: none;
      color: var(--gold);
      font-size: 12px;
      cursor: pointer;
      text-decoration: underline;
      padding: 0;
    }

    .date-input {
      font-size: 12px;
      padding: 5px 9px;
      border: 1px solid var(--gold);
      border-radius: var(--radius);
      background: var(--bg-card);
      color: var(--text);
      outline: none;
    }

    .plan-status-note {
      font-style: italic;
      color: var(--text-muted);
    }

    /* ── Stats ── */
    .stats {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 10px;
      margin-bottom: 1.5rem;
    }

    .stat {
      background: var(--bg-card);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      padding: 14px;
      box-shadow: 0 1px 3px var(--shadow);
    }

    .stat-val {
      font-size: 24px;
      font-weight: 600;
      color: var(--gold);
    }

    .stat-lbl {
      font-size: 12px;
      color: var(--text-muted);
      margin-top: 3px;
    }

    /* ── Progress bar ── */
    .progress-bar {
      height: 6px;
      background: var(--border);
      border-radius: 3px;
      margin-bottom: 1.5rem;
      overflow: hidden;
    }

    .progress-fill {
      height: 100%;
      background: var(--gold);
      border-radius: 3px;
      transition: width 0.3s ease;
    }

    /* ── Controls ── */
    .controls {
      display: flex;
      gap: 8px;
      align-items: center;
      margin-bottom: 1rem;
      flex-wrap: wrap;
    }

    .search-box {
      padding: 8px 12px;
      font-size: 13px;
      border: 1px solid var(--border);
      border-radius: var(--radius);
      background: var(--bg-card);
      color: var(--text);
      outline: none;
      flex: 1;
      min-width: 120px;
      max-width: 220px;
    }

    .search-box:focus { border-color: var(--gold); }

    .ctrl-btn {
      font-size: 12px;
      padding: 7px 14px;
      border: 1px solid var(--gold);
      border-radius: var(--radius);
      background: var(--bg-card);
      color: var(--gold);
      cursor: pointer;
      font-weight: 500;
      transition: background 0.15s;
    }

    .ctrl-btn:hover { background: var(--gold-light); }

    /* ── Card grid ── */
    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
      gap: 8px;
    }

    /* ── Day cards ── */
    .day-card {
      border: 1px solid var(--border);
      border-radius: var(--radius);
      padding: 12px 14px;
      background: var(--bg-card);
      cursor: pointer;
      transition: border-color 0.15s, box-shadow 0.15s;
      position: relative;
      min-height: 64px;
    }

    .day-card:hover {
      border-color: var(--gold);
      box-shadow: 0 2px 8px var(--shadow);
    }

    /* Done */
    .day-card.done {
      background: var(--bg-done);
      border-color: var(--border-done);
    }

    .day-card.done .day-num  { color: var(--green); }
    .day-card.done .chapters { color: var(--green); opacity: 0.85; }

    /* Today */
    .day-card.today {
      background: var(--gold-light);
      border: 1.5px solid var(--gold);
    }

    .day-card.today .day-num {
      color: var(--gold);
      font-weight: 700;
    }

    /* Today + Done */
    .day-card.today.done {
      background: var(--bg-done);
      border-color: var(--green);
    }

    .day-card.today.done .day-num { color: var(--green); }

    .day-num {
      font-size: 11px;
      font-weight: 500;
      color: var(--text-muted);
      margin-bottom: 5px;
      text-transform: uppercase;
      letter-spacing: 0.05em;
    }

    .chapters {
      font-size: 13px;
      color: var(--text);
      line-height: 1.55;
    }

    .check {
      position: absolute;
      top: 10px;
      right: 12px;
      font-size: 14px;
      color: var(--green);
      display: none;
    }

    .day-card.done .check { display: block; }

    /* ── Responsive ── */
    @media (max-width: 768px) {
      body { padding: 1rem 0.75rem; }
      .header-title { font-size: 19px; }
      .grid { grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); }
    }

    @media (max-width: 480px) {
      .grid { grid-template-columns: 1fr 1fr; }
    }
  </style>
</head>
<body>
<div class="container">

  <header>
    <span class="header-icon">✝</span>
    <div>
      <h1 class="header-title">Old Testament Reading Plans</h1>
      <p class="header-sub">Track your journey through the Old Testament</p>
    </div>
  </header>

  <div class="tabs" id="tabs">
    <button class="tab-btn active" onclick="switchPlan(0)">30-day sequential</button>
    <button class="tab-btn" onclick="switchPlan(1)">30-day chronological</button>
    <button class="tab-btn" onclick="switchPlan(2)">60-day sequential</button>
    <button class="tab-btn" onclick="switchPlan(3)">60-day chronological</button>
  </div>

  <div class="plan-header">
    <div class="plan-title" id="plan-title"></div>
    <div class="plan-sub" id="plan-sub"></div>
    <div class="start-date-line" id="start-date-line"></div>
  </div>

  <div class="stats">
    <div class="stat">
      <div class="stat-val" id="s-days"></div>
      <div class="stat-lbl">total days</div>
    </div>
    <div class="stat">
      <div class="stat-val" id="s-done"></div>
      <div class="stat-lbl">days completed</div>
    </div>
    <div class="stat">
      <div class="stat-val" id="s-pct"></div>
      <div class="stat-lbl">% complete</div>
    </div>
  </div>

  <div class="progress-bar"><div class="progress-fill" id="prog"></div></div>

  <div class="controls">
    <input class="search-box" type="text" placeholder="Search book…" id="search" oninput="renderCards()" />
    <button class="ctrl-btn" onclick="markAll(true)">Mark all done</button>
    <button class="ctrl-btn" onclick="markAll(false)">Reset</button>
  </div>

  <div class="grid" id="grid"></div>

</div>
<script>
// ── Data (unchanged from original) ──────────────────────────────────────────
const OT_CHAPTERS = {
  Genesis:50,Exodus:40,Leviticus:27,Numbers:36,Deuteronomy:34,
  Joshua:24,Judges:21,Ruth:4,"1 Samuel":31,"2 Samuel":24,
  "1 Kings":22,"2 Kings":25,"1 Chronicles":29,"2 Chronicles":36,
  Ezra:10,Nehemiah:13,Esther:10,Job:42,Psalms:150,Proverbs:31,
  Ecclesiastes:12,"Song of Solomon":8,Isaiah:66,Jeremiah:52,Lamentations:5,
  Ezekiel:48,Daniel:12,Hosea:14,Joel:3,Amos:9,Obadiah:1,Jonah:4,
  Micah:7,Nahum:3,Habakkuk:3,Zephaniah:3,Haggai:2,Zechariah:14,Malachi:4
};

const SEQ_ORDER = Object.keys(OT_CHAPTERS);

const CHRON_ORDER = [
  "Genesis","Job","Exodus","Leviticus","Numbers","Deuteronomy",
  "Joshua","Judges","Ruth","1 Samuel","2 Samuel","Psalms",
  "Proverbs","Ecclesiastes","Song of Solomon",
  "1 Kings","2 Kings","1 Chronicles","2 Chronicles",
  "Isaiah","Jeremiah","Lamentations","Ezekiel","Daniel",
  "Hosea","Joel","Amos","Obadiah","Jonah","Micah","Nahum",
  "Habakkuk","Zephaniah","Haggai","Zechariah","Malachi",
  "Ezra","Nehemiah","Esther"
];

const PLANS = [
  { title:"30-day Old Testament — sequential",    sub:"Genesis through Malachi, in canonical order",      order:SEQ_ORDER,  days:30 },
  { title:"30-day Old Testament — chronological", sub:"Books arranged in historical/narrative order",     order:CHRON_ORDER,days:30 },
  { title:"60-day Old Testament — sequential",    sub:"Genesis through Malachi, in canonical order",      order:SEQ_ORDER,  days:60 },
  { title:"60-day Old Testament — chronological", sub:"Books arranged in historical/narrative order",     order:CHRON_ORDER,days:60 },
];

// ── Plan builder (unchanged from original) ──────────────────────────────────
function totalChapters(order) {
  return order.reduce((s,b) => s + OT_CHAPTERS[b], 0);
}

function buildPlan(order, days) {
  const total = totalChapters(order);
  const perDay = total / days;
  const plan = [];
  const pool = [];
  for (const book of order)
    for (let c = 1; c <= OT_CHAPTERS[book]; c++) pool.push({ book, ch: c });
  let idx = 0;
  for (let d = 0; d < days; d++) {
    const target = Math.round((d + 1) * perDay);
    const dayChaps = [];
    while (idx < target && idx < pool.length) { dayChaps.push(pool[idx]); idx++; }
    if (dayChaps.length === 0 && idx < pool.length) { dayChaps.push(pool[idx]); idx++; }
    plan.push(dayChaps);
  }
  return plan;
}

function formatDay(chaps) {
  if (!chaps || chaps.length === 0) return "—";
  const groups = [];
  let i = 0;
  while (i < chaps.length) {
    const book = chaps[i].book;
    const start = chaps[i].ch;
    let end = start;
    while (i + 1 < chaps.length && chaps[i+1].book === book && chaps[i+1].ch === end + 1) { i++; end = chaps[i].ch; }
    groups.push(start === end ? `${book} ${start}` : `${book} ${start}–${end}`);
    i++;
  }
  return groups.join("; ");
}

const builtPlans = PLANS.map(p => buildPlan(p.order, p.days));

// ── Storage helpers ──────────────────────────────────────────────────────────
const progressKey = i => `bible-plan-${i}-progress`;
const startKey    = i => `bible-plan-${i}-start`;

function loadProgress(i) {
  try {
    const raw = localStorage.getItem(progressKey(i));
    if (raw) {
      const arr = JSON.parse(raw);
      if (Array.isArray(arr) && arr.length === PLANS[i].days) return arr;
    }
  } catch(e) {}
  return new Array(PLANS[i].days).fill(false);
}

function saveProgress(i) {
  localStorage.setItem(progressKey(i), JSON.stringify(progress[i]));
}

function loadStartDate(i) {
  return localStorage.getItem(startKey(i)) || null;
}

function saveStartDate(i, dateStr) {
  if (dateStr) localStorage.setItem(startKey(i), dateStr);
  else         localStorage.removeItem(startKey(i));
}

// ── State ────────────────────────────────────────────────────────────────────
const progress   = PLANS.map((_, i) => loadProgress(i));
const startDates = PLANS.map((_, i) => loadStartDate(i));
let current = 0;
let scrolledToToday = false;

// ── Today index ──────────────────────────────────────────────────────────────
function getTodayIndex(i) {
  const start = startDates[i];
  if (!start) return null;
  // Parse as local date (avoid UTC shift from bare YYYY-MM-DD in Date constructor)
  const startMs = new Date(start + 'T00:00:00').getTime();
  const todayMs = new Date(new Date().toDateString()).getTime();
  return Math.floor((todayMs - startMs) / 86400000);
}

// ── Start date UI ────────────────────────────────────────────────────────────
function renderStartDateLine() {
  const line = document.getElementById('start-date-line');
  const start = startDates[current];
  const todayIdx = getTodayIndex(current);
  const days = PLANS[current].days;

  if (!start) {
    line.innerHTML = `<button class="start-btn" onclick="showDateInput()">Set start date</button>`;
    return;
  }

  const dateLabel = new Date(start + 'T00:00:00')
    .toLocaleDateString('en-US', { month: 'long', day: 'numeric', year: 'numeric' });

  let statusNote = '';
  if (todayIdx !== null) {
    if (todayIdx >= days)
      statusNote = ' &nbsp;·&nbsp; <span class="plan-status-note">Plan complete 🎉</span>';
    else if (todayIdx < 0) {
      const n = -todayIdx;
      statusNote = ` &nbsp;·&nbsp; <span class="plan-status-note">Starting in ${n} day${n === 1 ? '' : 's'}</span>`;
    }
  }

  line.innerHTML =
    `Started: ${dateLabel} &nbsp;·&nbsp; <button class="link-btn" onclick="showDateInput()">Change</button>${statusNote}`;
}

function showDateInput() {
  const line = document.getElementById('start-date-line');
  const existing = startDates[current] || '';
  line.innerHTML = `<input class="date-input" type="date" id="date-picker" value="${existing}">`;
  const input = document.getElementById('date-picker');
  input.focus();
  input.onchange = () => {
    startDates[current] = input.value || null;
    saveStartDate(current, input.value || null);
    scrolledToToday = false;
    renderStartDateLine();
    renderCards();
  };
  // If user taps away without picking, restore the label
  input.onblur = () => setTimeout(renderStartDateLine, 200);
}

// ── Render ───────────────────────────────────────────────────────────────────
function switchPlan(i) {
  current = i;
  scrolledToToday = false;
  document.querySelectorAll('.tab-btn').forEach((b, j) => b.classList.toggle('active', j === i));
  document.getElementById('search').value = '';
  renderPlan();
}

function renderPlan() {
  const p = PLANS[current];
  document.getElementById('plan-title').textContent = p.title;
  document.getElementById('plan-sub').textContent   = p.sub;
  renderStartDateLine();
  updateStats();
  renderCards();
}

function updateStats() {
  const prog = progress[current];
  const done = prog.filter(Boolean).length;
  const pct  = Math.round(done / prog.length * 100);
  document.getElementById('s-days').textContent      = prog.length;
  document.getElementById('s-done').textContent      = done;
  document.getElementById('s-pct').textContent       = pct + '%';
  document.getElementById('prog').style.width        = pct + '%';
}

function renderCards() {
  const q        = document.getElementById('search').value.toLowerCase().trim();
  const plan     = builtPlans[current];
  const prog     = progress[current];
  const todayIdx = getTodayIndex(current);
  const grid     = document.getElementById('grid');
  grid.innerHTML = '';
  let todayCard  = null;

  plan.forEach((chaps, d) => {
    const label   = formatDay(chaps);
    if (q && !label.toLowerCase().includes(q)) return;

    const isToday = (todayIdx === d);
    const isDone  = prog[d];

    const card = document.createElement('div');
    card.className = 'day-card' + (isDone ? ' done' : '') + (isToday ? ' today' : '');

    const dayLabel = isToday
      ? `<strong style="font-size:11px;letter-spacing:.05em;text-transform:uppercase;">Today</strong>`
      : `Day ${d + 1}`;

    card.innerHTML =
      `<div class="day-num">${dayLabel}</div>` +
      `<div class="chapters">${label}</div>` +
      `<span class="check" aria-hidden="true">✓</span>`;

    card.onclick = () => {
      prog[d] = !prog[d];
      saveProgress(current);
      updateStats();
      renderCards();
    };

    grid.appendChild(card);
    if (isToday) todayCard = card;
  });

  // Scroll to today once per plan load (not on every toggle/search)
  if (todayCard && !scrolledToToday && !q) {
    scrolledToToday = true;
    setTimeout(() => todayCard.scrollIntoView({ behavior: 'smooth', block: 'center' }), 150);
  }
}

function markAll(val) {
  progress[current].fill(val);
  if (!val) {
    startDates[current] = null;
    saveStartDate(current, null);
  }
  saveProgress(current);
  updateStats();
  renderCards();
  if (!val) renderStartDateLine();
}

// ── Boot ─────────────────────────────────────────────────────────────────────
renderPlan();
</script>
</body>
</html>
```

- [ ] **Step 2: Confirm the file was saved**

Open a terminal in the project directory and run:

```bash
wc -l "bible_reading_plans.html"
```

Expected output: a line count above 300 (the new file is ~340 lines).

---

## Task 2: Verify visual design in browser

**Files:**
- Read: `bible_reading_plans.html` (open in browser)

- [ ] **Step 1: Open the file in Safari (or any browser)**

```bash
open "bible_reading_plans.html"
```

- [ ] **Step 2: Verify the header**

You should see a gold ✝ cross icon on the left, "Old Testament Reading Plans" in a serif (Georgia) font, and a smaller muted subtitle below it. A horizontal rule separates it from the tabs.

- [ ] **Step 3: Verify the tab styles**

The first tab ("30-day sequential") should have a gold border and gold text on a warm yellow-cream background. The other tabs should be muted gray on cream, and should highlight gold on hover.

- [ ] **Step 4: Verify stat boxes**

Three stat boxes with a subtle shadow, gold numbers, and muted gray labels.

- [ ] **Step 5: Verify the progress bar**

A 6px tall gold bar (initially at 0%) on a cream-colored track.

- [ ] **Step 6: Verify card styles**

Cards should have a cream background (`#fefaf0`) and a warm tan border. Hover over a card — it should get a gold border and a soft shadow. Click a card — it should turn sage green with a ✓ checkmark in the top-right. Click it again — it should return to the default state.

- [ ] **Step 7: Verify control buttons**

"Mark all done" and "Reset" buttons should have gold borders and gold text on a cream background, turning gold-light on hover. The search box should gain a gold border on focus.

- [ ] **Step 8: Verify "Set start date" button appears**

Below the plan subtitle, a small "Set start date" button should be visible with a tan border.

---

## Task 3: Verify localStorage persistence

**Files:**
- Read: `bible_reading_plans.html` (open in browser with DevTools)

- [ ] **Step 1: Mark a few days as done**

Click 3–4 day cards so they turn green.

- [ ] **Step 2: Confirm storage was written**

Open browser DevTools → Application → Local Storage → `file://`. You should see a key `bible-plan-0-progress` with a JSON array like `[true, true, false, true, false, ...]`.

- [ ] **Step 3: Reload the page**

Press Cmd+R (or Ctrl+R). The same cards should still be green — progress was restored from storage.

- [ ] **Step 4: Switch plans and verify isolation**

Click the "30-day chronological" tab. It should start at 0 completed — its progress is independent. Mark a card done on this plan, reload, and confirm it persists separately.

- [ ] **Step 5: Test Reset clears storage**

Click "Reset". Cards go back to default. Check DevTools Local Storage — `bible-plan-1-progress` should be gone (key removed) or an all-false array.

---

## Task 4: Verify start date and today highlighting

**Files:**
- Read: `bible_reading_plans.html` (open in browser)

- [ ] **Step 1: Set today as the start date**

Click "Set start date". The native date picker appears. Choose today's date. The line should update to show "Started: [today's date] · Change".

- [ ] **Step 2: Verify Day 1 is highlighted**

Day 1's card should have a gold border, gold-light background, and its label should read "Today" in gold uppercase instead of "Day 1". The page should have smoothly scrolled so Day 1 is centered in the viewport.

- [ ] **Step 3: Verify "Today" + done state**

Click the "Today" card to mark it done. It should turn sage green with a ✓, and the "Today" label should stay visible in green (not revert to "Day 1").

- [ ] **Step 4: Test a past start date (simulates being mid-plan)**

Click "Change" and set the start date to 5 days ago. Day 6 should now be the "Today" card. Days 1–5 are not auto-marked (user manually marks progress), but Day 6 gets the gold highlight.

- [ ] **Step 5: Test a future start date**

Set the start date to tomorrow. No card should be gold-highlighted. The start date line should show "Starting in 1 day".

- [ ] **Step 6: Verify start date persists across reload**

Set a start date, reload the page. The start date line should show the same date and the same card should be gold-highlighted.

- [ ] **Step 7: Verify Reset clears start date**

Click "Reset". The start date line should revert to the "Set start date" button and all cards should return to default cream.

---

## Task 5: Verify responsive layout for iPad Pro

**Files:**
- Read: `bible_reading_plans.html` (open in browser DevTools)

- [ ] **Step 1: Open DevTools responsive mode**

In Safari or Chrome DevTools, open the device toolbar (Cmd+Shift+M in Chrome). Set dimensions to **1024 × 1366** (iPad Pro 12.9" portrait).

- [ ] **Step 2: Verify card grid at iPad portrait width**

The grid should show 3–4 columns of cards. Cards should be comfortable to read with no text overflow.

- [ ] **Step 3: Rotate to landscape (1366 × 1024)**

The grid should reflow to 4–5 columns. The layout should remain clean.

- [ ] **Step 4: Test at iPad mini / iPad Air width (768 × 1024)**

At 768px width, the `@media (max-width: 768px)` rule kicks in. The grid should use `minmax(180px, 1fr)` — verify cards are still readable and at least 2 columns wide.

- [ ] **Step 5: Verify touch target size**

Each card has `min-height: 64px`. In DevTools, inspect a day card and confirm `min-height: 64px` is applied. This ensures comfortable tap targets on the iPad touchscreen.

- [ ] **Step 6: Verify header scales correctly**

At 768px width, the header title should be 19px (down from 22px) — confirm in DevTools computed styles.

- [ ] **Step 7: Transfer file to iPad and open in Safari**

AirDrop `bible_reading_plans.html` to your iPad. Open it from the Files app in Safari. Confirm:
- Parchment background renders correctly
- Georgia serif heading is visible
- Day cards are easy to tap
- "Set start date" opens the native iOS date wheel
- Progress persists after closing and reopening the tab
