# Test Coverage Analysis — SEO Checklist App

## Current State

**Test coverage: 0%** — The project has no tests, no test framework, and no CI/CD pipeline.

The entire application lives in a single `Index.html` file containing ~100 lines of JavaScript with the following testable functions:

| Function | Lines | Purpose |
|----------|-------|---------|
| `loadState()` | 148 | Deserializes task state from localStorage |
| `saveState()` | 149 | Serializes task state to localStorage |
| `key(pid, si, ti)` | 150 | Generates a unique key for a task |
| `isDone(pid, si, ti)` | 151 | Checks if a task is completed |
| `toggle(pid, si, ti)` | 152 | Toggles task completion and triggers side effects |
| `resetAll()` | 153 | Clears all progress after confirmation |
| `countPhase(p)` | 155 | Counts total/done tasks in a phase |
| `countAll()` | 156 | Counts total/done tasks across all phases |
| `buildUI()` | 158–196 | Renders the entire UI from the `phases` data |
| `updateAll()` | 198–221 | Syncs UI elements with current state |
| `showPhase(pi)` | 223–229 | Switches the active phase tab |
| `checkPhaseComplete(pid)` | 232–244 | Shows celebration modal when a phase is 100% done |
| `closeCelebrate()` | 245–248 | Hides the celebration modal |

---

## Recommended Test Coverage Areas

### Priority 1: Pure Logic Functions (Unit Tests)

These functions have no DOM dependencies and can be tested immediately with any JS test runner (e.g., Vitest or Jest).

#### 1.1 `key(pid, si, ti)` — Key Generation
- Verify it produces the expected `"pid_si_ti"` format
- Test with various input types (strings, numbers, zero values)
- Test edge cases: empty strings, special characters in phase IDs

#### 1.2 `isDone(pid, si, ti)` — State Lookup
- Returns `true` when the state key is truthy
- Returns `false` when the state key is missing or falsy
- Handles an empty state object

#### 1.3 `countPhase(p)` — Per-Phase Counting
- Counts correctly when no tasks are done
- Counts correctly when all tasks are done
- Counts correctly with a mix of done/not-done tasks
- Handles a phase with zero sections/tasks (empty edge case)
- Handles phases with multiple sections of varying sizes

#### 1.4 `countAll()` — Global Counting
- Returns `{total: 0, done: 0}` with no phases
- Sums across all 5 phases correctly
- Handles partial completion across phases

### Priority 2: State Management (Unit Tests with localStorage mock)

#### 2.1 `loadState()`
- Parses valid JSON from localStorage correctly
- Recovers gracefully when localStorage contains invalid JSON (falls back to `{}`)
- Recovers gracefully when localStorage is empty (falls back to `{}`)
- Handles `localStorage.getItem` throwing an exception (e.g., storage disabled)

#### 2.2 `saveState()`
- Writes the current state to localStorage as JSON
- Handles `localStorage.setItem` throwing (e.g., quota exceeded) without crashing

#### 2.3 `toggle(pid, si, ti)`
- Flips a task from incomplete to complete
- Flips a task from complete to incomplete
- Calls `saveState()` after toggling
- Calls `updateAll()` after toggling
- Calls `checkPhaseComplete()` only when toggling **on** (not off)

#### 2.4 `resetAll()`
- Clears state when user confirms
- Does **not** clear state when user cancels
- Calls `saveState()` and `updateAll()` after reset

### Priority 3: DOM/UI Rendering (Integration Tests)

These require a DOM environment (jsdom or a browser-based runner like Playwright).

#### 3.1 `buildUI()`
- Renders the correct number of nav items (5 phases)
- Renders the correct number of task elements per phase
- First phase is active by default
- Task elements have the correct `done` class based on state
- Tags render with the correct CSS class (`tag-red`, `tag-amber`, etc.)
- Tasks with no `note` field don't render a `.task-note` element
- Tasks with no `tags` don't render a `.task-tags` container

#### 3.2 `updateAll()`
- Updates nav counts (`done/total`) for each phase
- Updates percentage displays
- Updates completed/remaining stats in phase headers
- Updates the global progress bar width
- Toggles `done` class on task elements correctly

#### 3.3 `showPhase(pi)`
- Only one `.phase` element has the `active` class at a time
- Only one `.nav-item` has the `active` class at a time
- Scrolls to top (`window.scrollTo(0, 0)`)

#### 3.4 `checkPhaseComplete(pid)`
- Shows celebration modal when all tasks in a phase are done
- Does **not** show modal if phase was already celebrated (idempotency via `celebrated` Set)
- Sets correct title text (`"Cleanup Complete!"`, etc.)
- Sets correct subtitle text with total count

#### 3.5 `closeCelebrate()`
- Removes `show` class from both overlay and celebration elements

### Priority 4: Data Integrity Tests

#### 4.1 `phases` Data Structure
- All 5 phases have a unique `id`
- Every phase has at least one section
- Every section has at least one task
- Every task has a non-empty `text` field
- Every tag tuple has exactly 2 elements: `[label, colorKey]`
- All tag color keys map to valid CSS classes (`red`, `amber`, `blue`, `green`, `gold`, `purple`, `grey`)
- No duplicate phase IDs exist

### Priority 5: End-to-End Tests (Playwright or Cypress)

#### 5.1 Full User Workflow
- Page loads and displays Phase 1 by default
- Clicking a task toggles it to done (visual check: strikethrough, green checkbox)
- Clicking a done task toggles it back to not-done
- Progress bar updates after toggling
- Navigating between phases works correctly
- Completing all tasks in a phase shows the celebration modal
- Closing the celebration modal works
- Reset button clears all progress after confirmation
- State persists across page reloads (localStorage)

#### 5.2 Edge Cases
- Corrupted localStorage data doesn't crash the app
- Opening the app with no localStorage (first visit) works correctly
- App renders correctly on mobile viewport widths

---

## Recommended Setup

### Framework: Vitest + jsdom

Vitest is the lightest-weight option for this project. Suggested setup:

```
npm init -y
npm install -D vitest jsdom
```

### Suggested File Structure

```
Seo-check/
├── Index.html
├── package.json
├── vitest.config.js
├── src/
│   └── logic.js          # Extract pure functions from Index.html
└── tests/
    ├── logic.test.js      # Priority 1 & 2 tests
    ├── ui.test.js          # Priority 3 tests (with jsdom)
    └── data.test.js        # Priority 4 tests
```

### Prerequisite: Extract JavaScript

Before testing is practical, the inline JavaScript in `Index.html` should be extracted into separate modules. At minimum:

1. **`src/logic.js`** — Export `key()`, `isDone()`, `countPhase()`, `countAll()`, `loadState()`, `saveState()`, `toggle()`, `resetAll()`
2. **`src/data.js`** — Export the `phases` array
3. **`src/ui.js`** — Export `buildUI()`, `updateAll()`, `showPhase()`, `checkPhaseComplete()`, `closeCelebrate()`

This separation makes functions importable by test files without needing to parse the full HTML.

---

## Impact Assessment

| Priority | Area | Estimated Tests | Risk Mitigated |
|----------|------|-----------------|----------------|
| P1 | Pure logic | ~15 tests | State corruption, counting bugs |
| P2 | State management | ~12 tests | Data loss, localStorage failures |
| P3 | DOM rendering | ~20 tests | UI not reflecting state, broken navigation |
| P4 | Data integrity | ~8 tests | Malformed phase data, broken tags |
| P5 | E2E workflows | ~10 tests | Full user-flow regressions |
| **Total** | | **~65 tests** | |

The highest-value starting point is **P1 + P2** (~27 tests) since they cover the core logic with zero DOM overhead and catch the most impactful bugs (data loss, incorrect progress tracking).
