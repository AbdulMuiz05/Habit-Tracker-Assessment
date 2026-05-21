# habits. — A Habit Tracker with Weekly Streaks

A single-page habit tracker built with vanilla HTML/CSS/JS. No build step. No dependencies. Just open it.

## How to run

**Option A — open directly:**
```
open index.html
```
Just double-click `index.html` in your file manager or drag it into any browser.

**Option B — local server (recommended to avoid any browser file:// quirks):**
```bash
npx serve .
# or
python3 -m http.server 8080
```
Then open `http://localhost:8080`.

No installs required beyond a browser. All state is stored in `localStorage`.

---

## Features

- Add, rename (click the name), and delete habits
- Weekly grid — habits as rows, days as columns
- Toggle checkmarks per day; future days are locked
- Streak counter per habit, updating live
- Week navigation (prev / next / this week)
- History preserved across page reloads
- Empty state, responsive layout, keyboard-navigable

## Browser support
Chrome 90+, Firefox 90+, Safari 15+, Edge 90+
