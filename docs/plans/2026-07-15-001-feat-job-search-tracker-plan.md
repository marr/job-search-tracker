---
title: "feat: Job Search Tracker Web App"
date: 2026-07-15
type: feat
artifact_contract: ce-unified-plan/v1
artifact_readiness: implementation-ready
execution: code
product_contract_source: ce-plan-bootstrap
---

# Job Search Tracker Web App

## Goal Capsule

Build a standalone single-file web app that transforms a spreadsheet-style job search log into a clean, company-centric dashboard. The app deduplicates entries by company, shows current pipeline status at a glance, and lets the user add/update entries — all without a build step or server.

## Problem Frame

The user tracks job applications in a Google Sheets spreadsheet with ~71 entries across ~53 unique companies. The spreadsheet shows the full history — every date-stamped entry is a separate row — making it hard to see the current state of the search. Many companies appear multiple times (e.g., Figma 3x, Stellar Health 3x, Blitzy 2x). The user needs a view that answers "where does everything stand right now?" without scrolling through chronological noise.

Current data shape (from exported HTML):
- **Columns:** Company, Date, Stage, Role, Score (star rating), Notes, Week
- **Stages:** Applied (32), No hire (26), Interviewing (7), Networking/empty (13)
- **Quirks:** Some entries have empty stages that represent networking/reaching out; some role fields contain action descriptions ("Reached out to Ben S") instead of job titles

## Requirements

- R1: Deduplicated company view — one row per company showing current status
- R2: Dashboard with summary stats (total apps, active pipeline, stage breakdown)
- R3: Filter by stage, search by company/role, sort by date/company/stage
- R4: Add new entries and update stage/status via a form
- R5: LocalStorage persistence — data survives page refresh
- R6: Standalone single HTML file — no build tools, no server, no dependencies
- R7: Responsive design — usable on mobile and desktop
- R8: Seed data from existing spreadsheet (~71 entries, 53 companies)
- R9: Weekly task tracking — qualifying activities (Applied, Interviewing, Networking, Offer) count toward a 3/week quota for unemployment insurance eligibility
- R10: Weekly progress indicator — prominent display showing current week completion (e.g., "Week 29: 2/3 tasks — 1 more needed")
- R11: Weekly history — past weeks shown with completion status so user can verify compliance at a glance
- R12: Non-job task logging — ability to add standalone weekly tasks (e.g., "updated resume", "networking event", "attended career fair") with date and description, which also count toward the 3/week quota

## Key Technical Decisions

### KTD1: Single-file architecture
All HTML, CSS, and JS in one `index.html` file. Data is embedded as a JS array on first load, then localStorage takes over.
- Rationale: Maximum portability — open the file in any browser, everything works. No deployment needed.
- Trade-off: File will be larger (~500-800 lines), but still manageable for a personal tool.

### KTD2: Company deduplication via normalization
Group entries by normalized company name (lowercased, stripped of punctuation). "Cadence" and "Cadence.care" map to the same entity. The most recent non-"No hire" entry determines current status; if all entries are "No hire", the company is marked rejected.
- Rationale: The core UX problem is duplicate companies. Normalization handles variant spellings.

### KTD3: Stage model
Five stages: Networking, Applied, Interviewing, Offer, No hire. Empty-stage entries from the spreadsheet map to "Networking". This keeps the pipeline clean while preserving the distinction between "I reached out" and "I formally applied".
- Alternatives considered: Merging Networking into Applied (rejected — loses signal about outreach vs. formal application).

### KTD4: localStorage with embedded seed data
First load reads from an embedded JS data array (parsed from the spreadsheet). Subsequent loads read from localStorage. A "Reset Data" button re-seeds from the embedded data.
- Rationale: Avoids needing a server or file import on every visit. The embedded seed is the "initial state" backup.

### KTD5: Weekly task tracking — dual source
Weekly tasks come from two sources: (1) job entries with qualifying stages (Applied, Interviewing, Networking, Offer) — derived automatically from existing entries, and (2) standalone weekly tasks logged explicitly for non-job activities (e.g., "updated resume", "networking event"). Both count toward the 3/week quota. Weeks use ISO week numbering (Monday–Sunday).
- Rationale: The primary use case is job activities, which are already tracked as entries. But some qualifying activities (resume updates, career fairs) aren't job entries, so a lightweight task logger covers the gap.
- Standalone tasks are stored separately (`weeklyTasks` array) to avoid polluting the job tracker with non-company entries.

### KTD6: Current week definition
Use the ISO week number of today's date. The weekly progress bar always shows the current calendar week. Past weeks are shown in a scrollable history below.
- Alternatives considered: Using the most recent entry's week number (rejected — the user needs to know if THIS week is on track, not which week had the last activity).

## Scope Boundaries

### In scope
- Single-file standalone web app
- Company-deduplicated tracker view
- Dashboard summary
- Filter, search, sort
- Add/edit entries via modal form
- localStorage persistence
- Responsive mobile/desktop layout
- Seed data from existing spreadsheet

### Deferred to Follow-Up Work
- CSV/JSON export and import
- Multi-user or cloud sync
- LinkedIn or job board integrations
- Interview scheduling or calendar view
- Salary/compensation tracking
- Standalone weekly task logging (non-job activities like "updated resume")

---

## Implementation Units

### U1. Data Extraction and App Scaffold

**Goal:** Parse the existing spreadsheet data into a usable JS structure and create the single-file app skeleton with base styling.

**Requirements:** R6, R8

**Dependencies:** None

**Files:**
- `index.html` — create (main app file)

**Approach:**
- Extract data from `Sheet1.html` using Python, output as a JS array of objects
- Each entry: `{ id, company, date, stage, role, score, notes, week }`
- Map empty stages to "Networking"
- Create `index.html` with embedded CSS (CSS variables for theme, typography, layout), embedded JS (data array, app state object), and basic HTML shell: header, dashboard container, table container
- CSS: modern clean look — neutral palette, good whitespace, subtle shadows, rounded corners
- Use system font stack

**Technical design:**

Data shape per entry:
```
{
  id: number,
  company: string,      // "Suno", "Figma", etc.
  date: string,         // "2026-03-17" (ISO format)
  stage: string,        // "Networking" | "Applied" | "Interviewing" | "Offer" | "No hire"
  role: string,         // "EM", "Staff Software Engineer", etc.
  score: number,        // 0-5 (star rating)
  notes: string,        // Free text
  week: number          // Week number from spreadsheet
}
```

App state:
```
{
  entries: Entry[],
  filters: { stage: string|null, search: string, sort: string },
  view: 'dashboard' | 'tracker',
  currentWeek: { year: number, week: number }  // ISO week of today
}
```

**Test expectation:** none — scaffold and data extraction only; verified by opening `index.html` in a browser and seeing the page load with data visible in console.

---

### U2. Dashboard Summary with Weekly Progress

**Goal:** Display summary cards and weekly progress tracking showing the current state of the job search and unemployment insurance eligibility.

**Requirements:** R1, R2, R9, R10, R11

**Dependencies:** U1

**Files:**
- `index.html` — modify (add dashboard section, CSS, JS logic)

**Approach:**
- **Weekly progress banner** (top of dashboard, most prominent element):
  - Current week number and date range (e.g., "Week 29 · Jun 30 – Jul 6")
  - Progress: "2/3 tasks" with a visual progress bar (3 segments, filled based on count)
  - Status message: green "Eligible ✓" when ≥3, amber "1 more needed" when 2, red "2 more needed" when 1, red "No tasks logged" when 0
  - "Qualifying" means stage is Networking, Applied, Interviewing, or Offer (NOT No hire)
- **Four summary cards** in a responsive grid:
  - **Total Applications** — count of unique companies
  - **Active Pipeline** — companies in Networking + Applied + Interviewing stages
  - **Interviews** — companies currently in Interviewing stage
  - **No Hires** — companies with terminal "No hire" status
- **Stage breakdown bar** — horizontal stacked bar showing proportion of each stage
- **Weekly history** (below summary cards):
  - Scrollable list of past weeks, each showing: week number, date range, task count, eligibility status
  - Current week highlighted at top
  - Weeks with 3+ tasks shown with green checkmark, weeks under 3 shown with warning icon
  - Date range derived from ISO week: Monday to Sunday
- **Deduplication logic:** group entries by normalized company name, pick latest stage per company
- Company is "active" if any entry has a non-"No hire" stage; most recent stage wins

**Technical design:**

Weekly aggregation:
```
function getWeekTasks(entries, weeklyTasks, weekNumber) {
  const qualifying = ['Networking', 'Applied', 'Interviewing', 'Offer'];
  const jobTasks = entries.filter(e => {
    const entryWeek = getISOWeekNumber(new Date(e.date));
    return entryWeek === weekNumber && qualifying.includes(e.stage);
  });
  const standaloneTasks = weeklyTasks.filter(t => {
    return getISOWeekNumber(new Date(t.date)) === weekNumber;
  });
  return [...jobTasks.map(e => ({ type: 'job', desc: `${e.stage} — ${e.company}`, date: e.date })),
          ...standaloneTasks.map(t => ({ type: 'task', desc: t.description, date: t.date }))];
}

function getISOWeekNumber(date) {
  // ISO 8601 week number: weeks start on Monday
  // Returns { year, week } object
}

function getWeekDateRange(year, week) {
  // Returns { start: Monday, end: Sunday } for display
}
```

Progress bar rendering:
```
// 3 segments, each fills when task count reaches that threshold
// Segment 1: green at 1 task
// Segment 2: green at 2 tasks  
// Segment 3: green at 3 tasks
// Over-3: stays fully green, shows ">3" badge
```

**Test scenarios:**
- Happy path: Current week shows correct count (e.g., entries in week 29 with qualifying stages)
- Happy path: Progress bar fills correctly — 0/3 empty, 1/3 one segment, 2/3 two segments, 3/3 full green
- Happy path: Weekly history shows past weeks in reverse chronological order
- Happy path: Week with 3+ qualifying tasks shows green checkmark eligibility
- Happy path: Week with fewer than 3 shows warning and deficit count
- Edge case: No entries in current week → shows "No tasks logged" with 0/3
- Edge case: Entry with "No hire" stage does NOT count toward weekly quota
- Edge case: Entry with empty stage (mapped to Networking) DOES count
- Edge case: Entry spanning week boundary — counts for the week its date falls in (Monday-based)
- Happy path: Log standalone task ("Updated resume") → counts toward weekly quota, progress bar updates
- Happy path: Standalone tasks shown alongside job entries in weekly history
- Happy path: Mix of job entries and standalone tasks in same week — total count is correct
- Integration: Adding a new entry for this week immediately updates the progress bar
- Integration: Adding a standalone task immediately updates the progress bar

---

### U3. Company Tracker Table

**Goal:** Display a deduplicated, sortable table showing one row per company with current status.

**Requirements:** R1, R3

**Dependencies:** U1, U2

**Files:**
- `index.html` — modify (add table section, CSS, JS rendering)

**Approach:**
- Table columns: Company, Current Stage (color badge), Latest Date, Roles Applied, Primary Role, Notes (truncated)
- Each row represents one company (deduplicated)
- Stage rendered as a colored pill/badge:
  - Networking: gray
  - Applied: blue
  - Interviewing: amber/orange
  - Offer: green
  - No hire: red/muted
- "Roles Applied" shows count when more than 1 role applied to at that company
- Primary Role shows the most recent role title
- Notes show first 50 chars with ellipsis; full notes on hover or click
- Click a row to expand and see all entries for that company (history drawer or expandable row)
- Default sort: by latest date descending (most recent activity first)

**Technical design:**

Deduplication function:
```
function deduplicateEntries(entries) {
  // Group by normalized company name
  // Sort each group by date descending
  // Pick latest non-"No hire" as current stage
  // Return one row per company with aggregated data
}
```

**Test scenarios:**
- Happy path: Table shows ~53 rows (one per unique company), sorted by most recent date
- Edge case: Figma shows 3 entries collapsed into one row with "3 roles" indicator
- Edge case: Company with only "No hire" entries shows "No hire" stage
- Edge case: Empty notes field renders as "—" or is hidden
- Integration: Expanding a row shows all historical entries for that company

**Verification:** Compare table row count against unique company count in spreadsheet. Verify Figma row shows multiple roles.

---

### U4. Filtering and Search

**Goal:** Let the user filter the tracker table by stage, search by company/role text, and change sort order.

**Requirements:** R3

**Dependencies:** U3

**Files:**
- `index.html` — modify (add filter controls, JS filter/sort logic)

**Approach:**
- **Stage filter:** Row of toggle buttons (All, Networking, Applied, Interviewing, No hire). Active filter highlighted. Click to toggle; "All" clears filter.
- **Search input:** Text input with magnifying glass icon. Filters rows where company name or any role contains the search text (case-insensitive).
- **Sort dropdown:** Options — Most Recent, Company A-Z, Stage. Default: Most Recent.
- Filters compose: stage filter + search text + sort order all apply simultaneously.
- Filter state is part of the app state object; table re-renders on any filter change.

**Test scenarios:**
- Happy path: Click "Interviewing" filter → only shows companies in Interviewing stage
- Happy path: Type "figma" in search → shows only Figma row
- Happy path: Combine stage filter "Applied" + search "open" → shows only Applied-stage rows containing "open"
- Edge case: Search with no matches → shows empty state message ("No companies match your search")
- Edge case: Clear all filters → returns to full list
- Integration: Adding a new entry (U5) that matches current filter immediately appears in filtered view

**Verification:** Apply each filter individually, verify correct rows shown. Combine filters, verify intersection. Clear filters, verify full list returns.

---

### U5. Add/Edit Entry

**Goal:** Let the user add new job applications and edit existing entries through a modal form.

**Requirements:** R4

**Dependencies:** U1, U3

**Files:**
- `index.html` — modify (add modal, form, JS CRUD logic)

**Approach:**
- **Add button:** Floating action button (+ icon) in bottom-right corner, opens modal
- **Modal form fields:**
  - Company (text, required)
  - Date (date picker, defaults to today)
  - Stage (dropdown: Networking, Applied, Interviewing, Offer, No hire)
  - Role (text, required)
  - Score (clickable star rating, 0-5)
  - Notes (textarea, optional)
- **Edit:** Click any table row → opens same modal pre-filled with that entry's data
- **Save:** Validates required fields, generates new ID (timestamp-based), saves to entries array, persists to localStorage, closes modal, re-renders table and dashboard
- **Delete:** Available in edit mode only — red "Delete" button, with confirmation prompt
- If company already exists, new entry is added as additional application for that company (deduplication handles display)

**Test scenarios:**
- Happy path: Add new entry → appears in table under correct company, dashboard counts update
- Happy path: Edit existing entry → stage change reflected in table immediately
- Happy path: Delete entry → removed from table, dashboard counts update
- Edge case: Add entry for existing company → company row shows updated role count
- Edge case: Submit form with empty required fields → validation error shown, form not submitted
- Edge case: Add entry with future date → accepted (user may pre-log)
- Integration: New entry persists after page refresh (localStorage)

**Verification:** Add an entry, refresh page, verify it's still there. Edit a stage, refresh, verify change persisted.

---

### U6. LocalStorage Persistence and Data Management

**Goal:** Persist all app state to localStorage and provide data management controls.

**Requirements:** R5, R6

**Dependencies:** U1, U2, U3, U4, U5

**Files:**
- `index.html` — modify (add persistence layer, settings UI)

**Approach:**
- On every state change (add, edit, delete, reorder), serialize `entries` and `weeklyTasks` arrays to localStorage
- On page load, check localStorage first; if empty, seed from embedded data array
- Keys: `job-tracker-entries`, `job-tracker-weekly-tasks`
- **Settings gear icon** in header opens a small dropdown with:
  - "Export as JSON" — downloads entries as `.json` file
  - "Reset to seed data" — clears localStorage, re-seeds from embedded array (with confirmation)
- "Last updated" timestamp shown in footer

**Test scenarios:**
- Happy path: Add entries, close tab, reopen → all entries and weekly tasks preserved
- Happy path: Export produces valid JSON file with all entries and tasks
- Happy path: Reset to seed data restores original spreadsheet data (standalone tasks cleared)
- Edge case: localStorage unavailable (private browsing) → app still works in-memory, shows warning
- Edge case: Corrupted localStorage → falls back to seed data gracefully

**Verification:** Add entries, refresh, verify persistence. Export JSON, verify file contains all entries. Reset, verify original data restored. Add a standalone weekly task, refresh, verify it persists.

---

## Verification Contract

- Open `index.html` in a modern browser (Chrome, Firefox, Safari)
- Dashboard shows accurate counts: ~53 unique companies, correct stage breakdown
- Weekly progress bar shows current week with correct qualifying task count
- Tracker table shows one row per company, sorted by most recent date
- Figma row shows 3 roles; Stellar Health shows 3 roles; Blitzy shows 2 roles
- Stage badges render with correct colors
- Stage filters work individually and compose with search
- Add new entry → appears in table and dashboard updates
- Edit entry → change persists after refresh
- Export JSON → valid file with all entries
- Reset → original seed data restored
- Weekly history shows past weeks with eligibility status
- Responsive: table stacks or scrolls on mobile viewport

## Definition of Done

- [ ] `index.html` is a standalone file that works by opening directly in a browser
- [ ] All ~71 seed entries are loaded and deduplicated into ~53 company rows
- [ ] Dashboard shows correct summary statistics
- [ ] Weekly progress bar shows current week with correct qualifying task count (job entries + standalone tasks)
- [ ] "Log Task" button works — adds standalone non-job tasks that count toward weekly quota
- [ ] Weekly history shows past weeks with eligibility status (green for ≥3, warning for <3)
- [ ] Stage filters, text search, and sort all work correctly
- [ ] Add/edit/delete entries work and persist to localStorage
- [ ] Export and reset functions work
- [ ] Responsive layout works on mobile viewports
- [ ] No external dependencies — pure HTML/CSS/JS
