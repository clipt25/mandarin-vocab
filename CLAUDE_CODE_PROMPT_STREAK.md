# Claude Code Prompt — Streak System (Mandarin app)

## Files to edit
- `mandarin-quiz.html` — streak is earned here (quiz completion triggers it)
- `mandarin-vocab.html` — displays the streak in the top bar (read-only)

Do NOT touch any other files.

## This is identical to the Japanese streak system — same logic, same design, different keys and file names.

---

## How the streak works

- **Earning a streak day:** completing any quiz round (reaching the results screen) counts as that day's activity.
- **Streak increments:** last activity was yesterday → increment by 1.
- **Already done today:** last activity was today → no change.
- **Missed day(s):** last activity was 2+ days ago → reset current to 1, preserve best.
- **Best streak:** never resets, updates whenever current exceeds it.
- **Timezone:** use local date via `new Date().toLocaleDateString('en-CA')` (YYYY-MM-DD local time, never UTC).

---

## localStorage keys

```js
const LS_STREAK_CURRENT = 'mn-streak-current';
const LS_STREAK_BEST    = 'mn-streak-best';
const LS_STREAK_DATE    = 'mn-streak-date';
```

---

## Streak logic — helper functions

Add to `mandarin-quiz.html`:

```js
function todayStr() {
  return new Date().toLocaleDateString('en-CA');
}

function getStreak() {
  return {
    current: parseInt(lsGet(LS_STREAK_CURRENT) || '0', 10),
    best:    parseInt(lsGet(LS_STREAK_BEST)    || '0', 10),
    date:    lsGet(LS_STREAK_DATE) || null,
  };
}

function recordActivity() {
  const today = todayStr();
  const s = getStreak();

  if (s.date === today) return;

  let newCurrent;
  if (s.date === null) {
    newCurrent = 1;
  } else {
    const last = new Date(s.date);
    const now  = new Date(today);
    const diffDays = Math.round((now - last) / 86400000);
    newCurrent = (diffDays === 1) ? s.current + 1 : 1;
  }

  const newBest = Math.max(newCurrent, s.best);

  lsSet(LS_STREAK_CURRENT, String(newCurrent));
  lsSet(LS_STREAK_BEST,    String(newBest));
  lsSet(LS_STREAK_DATE,    today);

  return { current: newCurrent, best: newBest, increased: newCurrent > s.current };
}
```

---

## Where to call `recordActivity()`

In `mandarin-quiz.html`, call it at the exact moment `state.screen` transitions to `'results'`. After calling it, re-render the streak chip.

---

## Visual — flame counter in the top bar

### Position
Left of the theme toggle in the top bar of both files:
```
[Flashcards] [Quiz]        🔥 7    ☀
```

### HTML
```html
<div class="streak-chip" id="streakChip" title="Current streak">
  <span class="streak-flame">🔥</span>
  <span class="streak-count" id="streakCount">0</span>
</div>
```

### CSS
```css
.streak-chip {
  display: flex;
  align-items: center;
  gap: 3px;
  font-family: 'Crimson Pro', Georgia, serif;
  font-size: 15px;
  line-height: 1;
  cursor: default;
  user-select: none;
  transition: opacity 200ms;
}
.streak-chip.zero { opacity: 0.35; }
.streak-flame { font-size: 16px; line-height: 1; }
.streak-count { font-size: 15px; font-weight: 400; color: #c8b99a; min-width: 14px; }
body.light .streak-count { color: #8a6040; }

@keyframes streak-pulse {
  0%   { transform: scale(1); }
  40%  { transform: scale(1.4); }
  100% { transform: scale(1); }
}
.streak-chip.just-increased .streak-flame {
  animation: streak-pulse 500ms ease-out;
}
@media (prefers-reduced-motion: reduce) {
  .streak-chip.just-increased .streak-flame { animation: none; }
}
```

### JS — render function
```js
function renderStreakChip() {
  const s = getStreak();
  const chip  = document.getElementById('streakChip');
  const count = document.getElementById('streakCount');
  if (!chip || !count) return;
  count.textContent = s.current;
  chip.classList.toggle('zero', s.current === 0);
}
```

Call `renderStreakChip()` on page load.

After `recordActivity()`, if `result.increased`:
```js
const chip = document.getElementById('streakChip');
if (chip) {
  chip.classList.add('just-increased');
  setTimeout(() => chip.classList.remove('just-increased'), 600);
}
renderStreakChip();
```

---

## Results screen — streak info

Below the score on the results screen, add:

```html
<p class="results-streak">
  🔥 <strong id="resultsStreakCurrent">0</strong> day streak
  &nbsp;·&nbsp; Best: <strong id="resultsStreakBest">0</strong>
</p>
```

CSS:
```css
.results-streak {
  font-family: 'Crimson Pro', Georgia, serif;
  font-size: 1rem;
  color: #a89880;
  margin-top: 6px;
  margin-bottom: 24px;
  text-align: center;
}
.results-streak strong { color: #c8b99a; }
body.light .results-streak { color: #8a6040; }
body.light .results-streak strong { color: #6a3010; }
```

After calling `recordActivity()` when rendering results:
```js
const s = getStreak();
const el1 = document.getElementById('resultsStreakCurrent');
const el2 = document.getElementById('resultsStreakBest');
if (el1) el1.textContent = s.current;
if (el2) el2.textContent = s.best;
```

---

## `mandarin-vocab.html` — read-only display

Add the streak chip HTML to the top bar (left of theme toggle), the CSS block, and `renderStreakChip()`. Call it on page load. No `recordActivity()` here.

---

## Bump service worker cache version

In `service-worker.js`, change:
```js
const CACHE = 'mn-vocab-v1';
```
to:
```js
const CACHE = 'mn-vocab-v2';
```

---

## Deploy

```bash
cd "/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Mandarin Review"
git add mandarin-quiz.html mandarin-vocab.html service-worker.js
git commit -m "Add streak system: daily quiz streak with flame counter in top bar"
git push origin main
```

---

## Checklist
- [ ] `recordActivity()` called once per completed quiz round (results screen transition)
- [ ] Streak logic correct: +1 yesterday, reset to 1 if 2+ days, no-op if today
- [ ] Best streak never decreases
- [ ] Flame chip in top bar of both files, greyed at 0
- [ ] Pulse animation on streak increase, respects `prefers-reduced-motion`
- [ ] Results screen shows current + best streak
- [ ] Local timezone used (not UTC)
- [ ] Service worker bumped to `mn-vocab-v2`
- [ ] Pushed to GitHub Pages
