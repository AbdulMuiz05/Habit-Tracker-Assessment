# ANSWERS.md

## 1. How to run

Open `index.html` directly in any modern browser. No build step, no package install.

For a local server (avoids any `file://` edge cases):
```bash
npx serve .
# or
python3 -m http.server 8080
```

All data persists in `localStorage` — reload freely.

---

## 2. Stack & design choices

**Stack:** Vanilla HTML/CSS/JS — no framework, no bundler. The whole app is one file. For a self-contained tracker that needs to persist data and render a grid, React would add zero UX value and 40 kB of overhead. Plain DOM manipulation is faster to reason about and ship here.

**Visual decision 1 — the check cell is a button, not a checkbox.**
Every toggleable cell is a `<button>` with `aria-pressed` rather than a styled `<input type="checkbox">`. This gave me full control over the 32px circular target, the SVG checkmark draw animation (stroke-dashoffset transition), and the "pop" scale keyframe on check. A real checkbox needs `appearance: none` hacks and still fights you on focus rings cross-browser. The trade-off: I have to wire up keyboard semantics myself, which I did.

**Visual decision 2 — today's column gets a subtle lime tint, not a heavy highlight.**
Most habit trackers slam a bright stripe on "today" that visually competes with the checked state. I used `rgba(200,241,53,.07)` — barely perceptible as a background wash — so today reads clearly when you scan the header, but checked cells (solid lime `#c8f135`) remain the loudest element in that column. The hierarchy: checked cell > today header label > today column tint. The header also gets a 4px dot under the day name as a secondary "you are here" marker that works even when the column isn't in view.

---

## 3. Responsive & accessibility

**360px phone:** The grid has a `min-width: 560px` and its container is `overflow-x: auto` with `-webkit-overflow-scrolling: touch`. On narrow screens the grid scrolls horizontally while the add-row and week-nav stay full-width above it. The habit name column is `minmax(140px, 1.8fr)` so it never collapses below readable. Font sizes drop slightly at ≤600px (`font-size: .6rem` on headers). The add button stays at a full thumb tap target.

**1440px laptop:** The grid expands to `max-width: 900px` and centres. The habit name column grows via `1.8fr`, giving longer names room without the grid looking sparse.

**Accessibility handled — keyboard navigation:**
Every interactive element is a native `<button>` or `<input>`, so Tab order is correct by default. Check buttons have `aria-pressed` + `aria-label` that includes the habit name and date. The grid uses `role="grid"` / `role="row"` / `role="rowheader"` / `role="columnheader"` / `role="gridcell"`. The week label has `aria-live="polite"` so screen readers announce week changes. Focus rings use `focus-visible` with a 2px outline in the accent colour — never suppressed globally.

**Accessibility skipped — row keyboard shortcuts (arrow-key grid navigation):**
A proper ARIA grid pattern calls for arrow keys moving focus between cells. I didn't implement this — Tab still reaches every cell, but arrow-key traversal is absent. With 7 days × N habits that could be dozens of tabs. The fix would be a `roving tabindex` pattern on the check buttons, which I'd add with another day.

---

## 4. AI usage

I used Claude (this model) during development.

**Where:** I asked for a SVG polyline path for the checkmark that animates nicely with `stroke-dashoffset`. It gave me `<polyline points="2.5,8.5 6.5,12.5 13.5,4.5" />` inside a 16×16 viewBox with `stroke-dasharray: 20` and `stroke-dashoffset: 20 → 0`.

**What I changed:** The AI's initial stroke-dasharray value was `24`, which caused a slight lag before the stroke became visible on check. I reduced it to `20` (matching the actual polyline length more closely) and added `stroke-linecap: round` + `stroke-linejoin: round` so the tick looks drawn rather than mechanical. I also moved the transition from a JS `classList` toggle to a pure CSS transition on `stroke-dashoffset` triggered by the `.checked` class — the AI had used a `requestAnimationFrame` loop which was unnecessary complexity.

---

## 5. Honest gap

**The streak calculation edge case around midnight.** Right now `todayISO()` snapshots the date at render time. If a user leaves the tab open past midnight, the "today" column and streak counts won't update until they reload. A production fix would be a `setInterval` (or `visibilitychange` listener) that re-renders when the calendar date changes. I'd wire that up with a ~60-second interval that checks whether `toISO(new Date()) !== lastRenderedDate` and calls `render()` if so — a 10-line addition.

---

## Design decisions defended

**Week starts Monday:** The ISO 8601 standard defines Monday as day 1. More practically, most habit-forming literature treats Mon–Sun as the work/rest cycle people actually plan around. Starting on Sunday pushes the weekend to split positions (Sat at end, Sun at start) which breaks the natural "two days off" visual cluster. Monday start lets you see the full weekend together on the right side of the grid.

**Streak counts up to today if today is checked, otherwise up to yesterday:** Losing your streak the moment you wake up on day 0 is punishing and wrong — you haven't broken it yet, you just haven't done today's habit. The "grace until end of day" interpretation counts yesterday's chain as your active streak and adds today if you've already ticked it. This matches how Duolingo and Streaks.app handle it and matches user mental models.

---
*Submitted via assessment portal.*
