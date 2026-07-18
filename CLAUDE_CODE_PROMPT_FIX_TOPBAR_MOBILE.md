# Claude Code Prompt — Fix crowded top bar + horizontal scroll on Mandarin vocab app

## Files to edit
`index.html` and mirror to `mandarin-vocab.html`.

## Use plan mode before making any changes.

---

## Problem
The top bar has too many elements and causes horizontal scrolling on iPhone (~390px). Currently it contains all of these in one row:
- Two hamburger menu icons (≡ ≡) — likely a duplicate
- FLASHCARDS | QUIZ tab switcher
- 🔥 streak chip
- ⚙ settings gear
- ☀️ theme toggle

This is too crowded. The next → navigation button at the bottom right is also being cut off.

---

## Fix

### Step 1 — Audit the top bar HTML
Read `index.html` and identify every element in the top bar. Check if there are really two hamburger icons — if so, determine which one is the chapter/filter menu and which (if any) is redundant. Remove any duplicate.

### Step 2 — Two-row layout for mobile
Restructure the top bar into two rows on mobile:

**Row 1** (existing app controls):
- Left: hamburger menu (chapter filter) only
- Right: ⚙ settings + ☀️ theme toggle

**Row 2** (navigation):
- FLASHCARDS | QUIZ tab switcher centred, full width
- 🔥 streak chip on the right of this row (or left)

This two-row approach means each row only has 2–3 items and can never overflow.

On **desktop** (≥ 640px), collapse back to a single row if desired, or keep two rows — either is fine.

### Step 3 — Kill horizontal scroll
Add to the global CSS:
```css
html, body {
  overflow-x: hidden;
  max-width: 100%;
}
```

Also audit every element for `min-width` values or fixed-width containers that could force overflow. Remove or cap them.

### Step 4 — Fix bottom navigation bar
The `← prev` / `1/7` / `next →` navigation bar at the bottom of flashcard view is also being clipped on the right. Make sure it is:
```css
width: 100%;
box-sizing: border-box;
padding: 0 12px;
```
So the `next →` button is always fully visible.

### Step 5 — Verify at 390px
After making changes, mentally walk through the layout at 390px width:
- Top bar row 1: fits without wrapping?
- Top bar row 2: tab switcher centred, streak visible?
- No element causes horizontal scroll?
- Bottom nav bar: all three elements visible?

---

## Constraints
- Do not remove the streak chip — just reposition it
- Do not remove the settings gear
- The FLASHCARDS | QUIZ tab switcher must remain clearly visible and tappable
- Keep all existing functionality — only change layout/CSS
- Mirror all changes to `mandarin-vocab.html`
- Single commit

---

## Deploy
```bash
cd "/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Mandarin Review"
git add index.html mandarin-vocab.html
git commit -m "Fix top bar mobile layout: two-row, no horizontal scroll"
git push origin main
```

## Checklist
- [ ] No horizontal scrolling at 390px width
- [ ] All top bar elements visible and not overlapping
- [ ] Tab switcher clearly visible and tappable
- [ ] Bottom prev/next navigation bar fully visible
- [ ] Works in dark and light mode
- [ ] index.html and mandarin-vocab.html identical
- [ ] Pushed to GitHub Pages
