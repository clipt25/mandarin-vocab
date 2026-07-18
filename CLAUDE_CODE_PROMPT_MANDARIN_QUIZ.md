# Claude Code Prompt — `mandarin-quiz.html`

## Context

The user has a live Mandarin vocabulary app at:
`https://clipt25.github.io/mandarin-vocab/`

The source file in this folder is `mandarin-vocab.html`. Read it as a reference for:
- Exact CSS variables, colour values, font loading
- The full `CHAPTERS` constant (the vocabulary data)
- The `vocab-mode` localStorage key for theme

Do **not** modify `mandarin-vocab.html` — treat it as read-only reference, except for one small addition (see "Mode switcher" section below).

Your job is to create a new file: **`mandarin-quiz.html`** in the same folder.

---

## Goal

A self-contained, single-file Mandarin vocabulary quiz. Same warm dark aesthetic as `mandarin-vocab.html`. Two drill types, chapter-filtered. No katakana drill (this is Mandarin, not Japanese).

---

## File to create

`mandarin-quiz.html` — all HTML, CSS, JS inline. No external CSS, no build step.

---

## Visual Design — inherit exactly from `mandarin-vocab.html`

- **Fonts:** `Noto Serif JP` (300/400/600) for Chinese characters; `Crimson Pro` (300/400, italic 300) for pinyin and Latin. Same Google Fonts `<link>` tags.
- **Colour palette (dark, default):**
  - Page background: `#0e0d0b`
  - Primary text: `#e8e2d9`
  - Headword (hanzi): `#f0ebe0`
  - Pinyin / accent gold: `#a89880`
  - Muted label: `#6b5e45`
  - Divider: `rgba(255,255,255,0.05)`
  - Filled button: `#c8b99a` text on `#0e0d0b`
  - Ghost button border: `rgba(200,185,154,0.25)`
  - Correct flash: `rgba(120,200,120,0.25)`
  - Wrong flash: `rgba(200,80,80,0.25)`
- **Light mode** (toggled, persisted in `localStorage["vocab-mode"]` — same key as vocab app):
  - Page: `#f5efe3`; text: `#2a1e0a`; headword: `#1a0e00`; accent: `#8a6040`
- **Mobile-first.** Full iPhone X support: `viewport-fit=cover`, `100dvh`, `env(safe-area-inset-*)`, font-size `16px` on inputs. Minimum touch target 44×44px.

---

## Layout

Full-width centred card, max-width `520px`, `margin: 0 auto`. No sidebar, no word-list panel.

```
┌─────────────────────────────────────────────┐
│  [Flashcards] [Quiz]          ☀             │  ← sticky top bar
├─────────────────────────────────────────────┤
│                                             │
│   [Chapter selection screen]                │
│         OR                                  │
│   [Drill selection screen]                  │
│         OR                                  │
│   [Quiz card + options]                     │
│         OR                                  │
│   [Results screen]                          │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Vocabulary Data

At the top of the `<script>` block, copy the entire `CHAPTERS` array from `mandarin-vocab.html` verbatim. Do not shorten or summarise it — every chapter and every word must be present.

Each word object shape (for reference):
```js
{ h: "你好", p: "nǐ hǎo", pos: "phrase", m: "hello", ex: "你好！", exm: "Hello!" }
```

Fields used by the quiz: `h` (hanzi), `p` (pinyin), `m` (English meaning). `pos`, `ex`, `exm` are not needed for quiz logic but are present in the copied data.

---

## App Flow — 4 Screens

### Screen 1 — Chapter Selection

Show a list of all chapters as selectable cards. Each card shows:
- Chapter number (small, muted)
- Chinese title (`zh` field, Noto Serif JP)
- English subtitle (`en` field, italic Crimson Pro, muted)
- Word count (`X words`, small, muted)

Layout: a scrollable list of chapter cards, plus one special card at the top:

```
┌─────────────────────────────────────┐
│ ★  All Chapters  •  NNN words       │  ← "All" option, always first
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│ Ch.1  你好  •  Hello  •  7 words    │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│ Ch.2  谢谢  •  Thank You  •  5 words│
└─────────────────────────────────────┘
...
```

Tapping a chapter card selects it (highlighted border) and immediately advances to Screen 2 (drill selection). The "All Chapters" card uses all words from all chapters combined.

No "Next" button needed — tapping a chapter is the action.

---

### Screen 2 — Drill Selection

Same as the Japanese quiz. Show two drill cards:

| Drill | Label | Description |
|---|---|---|
| `hanzi-to-reading` | Hanzi → Reading & Meaning | See a Chinese character. Pick its pinyin + English meaning from 4 options. |
| `reading-to-hanzi` | Reading → Hanzi | See the pinyin and English meaning. Pick the correct hanzi from 4 options. |

Below the drill cards:
- A small back link: `← Change chapter` — returns to Screen 1
- Round-length selector: **5 / 10 / 20**, default 10, persisted in `localStorage["mn-quiz-round-length"]`
- **START** button (full width, 52px tall)

If the selected chapter (or all chapters) has fewer than 4 words, show a notice and disable both drills.

---

### Screen 3 — Quiz Card

```
┌────────────────────────────────────┐
│  Drill name  •  Ch.X      3 / 10   │
├────────────────────────────────────┤
│  [progress bar]                    │
│                                    │
│   [Question display]               │
│                                    │
├────────────────────────────────────┤
│  A  [option]                       │
│  B  [option]                       │
│  C  [option]                       │
│  D  [option]                       │
└────────────────────────────────────┘
```

**Question display per drill:**

- **`hanzi-to-reading`** — Show the hanzi large (`font-size: min(14vw, 5rem)`, Noto Serif JP 300, `#f0ebe0`).
  - Options A–D: `pinyin • English meaning` (e.g., `nǐ hǎo • hello`). Crimson Pro for both parts.

- **`reading-to-hanzi`** — Show pinyin (`1.8rem`, gold `#a89880`) + English meaning below (`1.1rem`, muted).
  - Options A–D: just the hanzi, displayed large (Noto Serif JP).

**Answer feedback:**
1. Correct → green flash on chosen option for 800ms, then auto-advance.
2. Wrong → red flash on chosen option + green flash on correct option for 800ms, then auto-advance.
3. All buttons disabled during the 800ms window.

**Keyboard:** `A`/`B`/`C`/`D` keys select options. Arrow keys do nothing during quiz.

---

### Screen 4 — Results

```
┌────────────────────────────────────┐
│  Results                           │
│                                    │
│        8 / 10                      │
│     80% correct                    │
│                                    │
│  [Try again — same drill]          │
│  [Change drill]                    │
│  [Change chapter]                  │
└────────────────────────────────────┘
```

Three buttons: retry same drill, back to drill selection, back to chapter selection.

---

## State Machine

```js
const state = {
  screen: 'chapter',      // 'chapter' | 'select' | 'quiz' | 'results'
  chapterIdx: null,        // null = all chapters; number = CHAPTERS index
  drillType: null,         // 'hanzi-to-reading' | 'reading-to-hanzi'
  roundLength: 10,
  questions: [],
  currentIndex: 0,
  score: 0,
  answered: false,
};
```

Single `render()` function reads `state` and re-renders the main content area. Call after every state mutation.

---

## Question Object Shape

```js
{
  word: { h, p, m },        // correct word
  options: [
    { h, p, m, isCorrect: true },
    { h, p, m, isCorrect: false },
    { h, p, m, isCorrect: false },
    { h, p, m, isCorrect: false },
  ]
}
```

Distractors: pick 3 random words from the active word pool (same chapter or all chapters). Never duplicate within a question's options. Shuffle A–D positions randomly each round.

If round length > pool size, allow word repeats across questions.

---

## Mode Switcher (top bar — both files)

### In `mandarin-quiz.html` (Quiz is active):
```html
<div class="mode-switcher">
  <a class="mode-btn inactive" href="./mandarin-vocab.html">Flashcards</a>
  <span class="mode-btn active">Quiz</span>
</div>
```

### Edit `mandarin-vocab.html` (Flashcards is active):
Find the top bar. Add the switcher with Quiz linking to `./mandarin-quiz.html`. This is the **only** change allowed to `mandarin-vocab.html`.

```html
<div class="mode-switcher">
  <span class="mode-btn active">Flashcards</span>
  <a class="mode-btn inactive" href="./mandarin-quiz.html">Quiz</a>
</div>
```

### Mode switcher CSS (same as japanese-quiz.html — copy it):
```css
.mode-switcher {
  display: flex;
  gap: 4px;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 8px;
  padding: 3px;
}
.mode-btn {
  font-family: 'Crimson Pro', Georgia, serif;
  font-size: 12px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  padding: 5px 14px;
  border-radius: 6px;
  border: none;
  cursor: pointer;
  text-decoration: none;
  display: inline-flex;
  align-items: center;
  transition: background 150ms, color 150ms;
  white-space: nowrap;
  min-height: 30px;
}
.mode-btn.active  { background: #c8b99a; color: #0e0d0b; }
.mode-btn.inactive { background: transparent; color: #6b5e45; }
.mode-btn.inactive:hover { color: #a89880; background: rgba(255,255,255,0.04); }
body.light .mode-btn.active   { background: #8a6040; color: #f5efe3; }
body.light .mode-btn.inactive { color: #b09070; }
```

---

## `<head>` boilerplate

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<meta name="google" content="notranslate">
<meta name="theme-color" content="#0e0d0b">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Noto+Serif+JP:wght@300;400;600&family=Crimson+Pro:ital,wght@0,300;0,400;1,300&display=swap" rel="stylesheet">
```

---

## Mobile spec

- `overflow-x: hidden` on `body` and main content area
- Top bar: `padding-top: max(12px, env(safe-area-inset-top))` for iPhone notch
- Bottom: `padding-bottom: max(64px, calc(40px + env(safe-area-inset-bottom)))` for home indicator
- Chapter cards: full width, min 52px tall, `box-sizing: border-box`
- Option buttons: `white-space: normal; word-break: break-word; min-height: 44px; width: 100%`
- `@media (max-width: 400px)`: reduce font sizes slightly, tighten padding

---

## Deploy

After creating `mandarin-quiz.html` and editing `mandarin-vocab.html`:

```bash
cd "/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Mandarin Review"

# Check remote
git remote -v

# If remote is already clipt25/mandarin-vocab:
git add mandarin-quiz.html mandarin-vocab.html
git commit -m "Add mandarin-quiz.html — hanzi quiz with chapter filter and 2 drill types"
git push origin main
```

If no remote exists:
```bash
git remote add origin https://github.com/clipt25/mandarin-vocab.git
git add mandarin-quiz.html mandarin-vocab.html
git commit -m "Add mandarin-quiz.html — hanzi quiz with chapter filter and 2 drill types"
git push origin main
```

Live URL after deploy:
`https://clipt25.github.io/mandarin-vocab/mandarin-quiz.html`

---

## Checklist before finishing

- [ ] All 23 chapters from `CHAPTERS` are copied into `mandarin-quiz.html` verbatim
- [ ] Chapter selection screen shows all chapters + "All Chapters" option
- [ ] Both drill types work end-to-end
- [ ] 800ms feedback window with correct/wrong flash, then auto-advance
- [ ] Round-length selector (5/10/20) works and persists in `localStorage["mn-quiz-round-length"]`
- [ ] Results screen has all 3 action buttons (retry, change drill, change chapter)
- [ ] Mode switcher in top bar of both files, correct active/inactive states
- [ ] Theme toggle works, shares `localStorage["vocab-mode"]` with vocab app
- [ ] Mobile: no horizontal scroll at 375px, all touch targets ≥ 44px, safe-area insets applied
- [ ] `prefers-reduced-motion` handled
- [ ] Pushed to GitHub Pages and live URL confirmed
