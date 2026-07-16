# feat: Add role autocomplete for company entries

**Created:** 2026-07-16
**Type:** feat
**Artifact:** ce-unified-plan/v1
**Readiness:** implementation-ready

---

## Summary

Wire up the existing role-suggestions datalist to activate when the company name changes in the entry form, enabling autocomplete for previously used roles.

## Problem Frame

When adding a job entry, users often apply to multiple roles at the same company. Currently, there's no way to see or reuse roles they've previously entered for that company. The codebase already has the infrastructure (a `<datalist>` element and an `updateRoleSuggestions()` function) but it only fires when editing an existing application — not when creating a new one or when the company name changes.

## Requirements

- R1: Role suggestions should appear when the user types or selects a company name
- R2: Suggestions should show unique roles previously used for that specific company
- R3: Suggestions should update dynamically as the company name changes

## Scope Boundaries

### In scope
- Wiring the existing `updateRoleSuggestions()` function to the company input's `oninput` event
- Calling `updateRoleSuggestions()` when the modal opens with a pre-filled company

### Out of scope
- Fuzzy matching or partial company name matching
- Cross-company role suggestions
- UI redesign of the autocomplete dropdown

---

## Key Technical Decisions

**KTD1: Event-driven approach** — Use the existing `oninput` event pattern rather than debouncing or adding a separate lookup mechanism. The datalist API handles filtering natively, and the IndexedDB queries are fast enough for real-time updates.

---

## Implementation Units

### U1. Wire role autocomplete to company input

**Goal:** Make role suggestions appear when the company name changes

**Requirements:** R1, R2, R3

**Dependencies:** None

**Files:**
- `index.html` (modify)

**Approach:**
1. Add `oninput="updateRoleSuggestions()"` to the company input element
2. Call `updateRoleSuggestions()` after setting the company value in `openLogModal()` when a companyId is provided

**Patterns to follow:**
- Existing `oninput` pattern used on the search input (line 778)
- Existing `updateRoleSuggestions()` function (lines 1497-1510)

**Test scenarios:**
- Happy path: Type a company name that has previous applications → role suggestions appear
- Happy path: Select a company from the datalist → role suggestions update
- Edge case: Type a company name with no previous applications → no suggestions shown
- Edge case: Clear the company name → suggestions disappear
- Integration: Open modal from history with pre-filled company → suggestions load immediately

**Verification:**
- Open the app, add an entry for "Acme" with role "Engineer"
- Add another entry, type "Acme" in company field → "Engineer" should appear as suggestion
- Open history for "Acme", click "+ Add Entry" → suggestions should load immediately

---

## Verification Contract

- All requirements trace to test scenarios in U1
- Manual verification steps cover the primary user flow
