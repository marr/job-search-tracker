# feat: Table Cleanup & Company Grouping

**Target repo:** 2026-07-15-job  
**Date:** 2026-07-16

---

## Problem Frame

The job tracker table has spacing issues: dates wrap to two lines, empty notes waste space, and the "N applications" column takes more width than its content deserves. The user wants a cleaner, more compact layout with inline company grouping.

## Current State

- `index.html` — single-file app with inline CSS and JS
- Table renders via `renderTable()` (line ~1325)
- 6 columns: Company, Stage, Latest Date, Roles (count), Primary Role, Notes
- Notes: `max-width: 200px`, `white-space: nowrap`, `text-overflow: ellipsis`
- Clicking a row opens `history-modal` showing all entries for that company

## Proposed Changes

### 1. Compact Date Formatting

**Goal:** Dates fit on one line, no wrapping.

**Approach:**
- Format dates as `Jan 15` or `Jul 3` (month abbreviation + day)
- Add `white-space: nowrap` to date cell
- Narrow the date column via `width: 80px` or `min-width: 80px`

**Files:** `index.html` (CSS + JS `renderTable`)

### 2. Collapsible Notes

**Goal:** Notes only visible when expanded; otherwise show just an indicator.

**Approach:**
- Replace raw notes text with a toggle button (e.g., `📝` or `•••`)
- Clicking toggles a `expanded` class on the row or a nested div
- When collapsed: show empty space or hidden
- When expanded: show notes inline below the row or in a popover

**Alternative (simpler):** Keep notes truncated but hide when empty — set `display: none` on `.notes-cell` when notes is empty string.

**Files:** `index.html` (CSS + JS `renderTable`)

### 3. Inline Company Grouping (Replace Modal)

**Goal:** Show company history inline instead of modal popup.

**Approach:**
- Each company row expands/collapses to show its history entries
- Click toggles expansion (accordion-style)
- Expanded state shows a sub-table or list of all applications for that company
- Remove or repurpose the `history-modal`

**Sub-table structure:**
```
[▶ Company Name] | [Stage Badge] | [Date] | [Role] | [Notes toggle]
   ┌─────────────────────────────────────────────────────────────┐
   │  Jan 15  Applied    Software Engineer   Resume submitted   │
   │  Feb 02  Interview  Software Engineer   Phone screen      │
   │  Feb 20  No hire    Software Engineer   Rejected           │
   └─────────────────────────────────────────────────────────────┘
```

**Files:** `index.html` (CSS + JS `renderTable` + remove/adapt modal)

### 4. Compact Roles/Count Display

**Goal:** "N applications" takes less space.

**Approach:**
- Remove dedicated "Roles" column
- Show count as a small superscript badge on the company name: `Company Name²` or `Company Name (2)`
- Or show as a subtle pill next to the stage badge

**Files:** `index.html` (CSS + JS `renderTable`)

---

## Implementation Units

### U1. Date Formatting & Column Width

**Goal:** Dates display compactly on one line.

**Files:** `index.html`

**Approach:**
- Add `formatDate()` helper: `new Date(dateStr).toLocaleDateString('en-US', { month: 'short', day: 'numeric' })`
- Update `renderTable()` to use formatted date
- Add CSS: `.date-cell { white-space: nowrap; width: 80px; }`

**Test scenarios:**
- Date `2026-01-15` renders as `Jan 15`
- Date `2026-12-31` renders as `Dec 31`
- No line wrapping on narrow viewports

---

### U2. Notes Collapse Behavior

**Goal:** Notes hidden by default, expandable on click.

**Files:** `index.html`

**Approach:**
- Add toggle state per row (or per company)
- When notes empty: hide cell or show `—`
- When notes present: show truncated with `...` and a small expand icon
- Click icon expands to full notes (could be inline or tooltip)

**Test scenarios:**
- Empty notes cell shows `—` or is hidden
- Long notes truncated with ellipsis
- Clicking expand shows full notes
- Clicking again collapses

---

### U3. Inline Company Expansion

**Goal:** Replace modal with inline accordion for company history.

**Files:** `index.html`

**Approach:**
- Modify `renderTable()` to generate expandable rows
- First row: company summary (stage, date, role)
- Hidden row(s): history entries when expanded
- Toggle on click (replace `showCompanyHistory` onclick)
- Style sub-rows with indented left border or background

**Test scenarios:**
- Clicking company row expands to show history
- Clicking again collapses
- History entries sorted by date (newest first)
- Only one company expanded at a time (accordion mode) — opening one closes others
- Remove or disable `history-modal`

---

### U4. Compact Roles Display

**Goal:** Application count displayed inline, not as separate column.

**Files:** `index.html`

**Approach:**
- Remove "Roles" `<th>` and `<td>` from table
- Append count to company name: `CompanyName <span class="app-count">(3)</span>`
- Style count as small, muted text

**Test scenarios:**
- Single application: no count shown
- Multiple applications: count shown as `(N)`
- Count is visually subordinate to company name

---

## Scope Boundaries

**In scope:**
- Table layout and column cleanup
- Inline expansion replacing modal
- Notes collapse behavior

**Out of scope:**
- Data model changes (IndexedDB schema unchanged)
- New features beyond table cleanup
- Mobile responsiveness overhaul

---

## Verification

1. Load `index.html` in browser
2. Add test entries for multiple companies
3. Verify dates show as `Mon DD` format, no wrapping
4. Verify notes hidden by default, expandable
5. Verify clicking company expands to show history inline
6. Verify application count shown inline on company name
7. Verify modal is removed or disabled
