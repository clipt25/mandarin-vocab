# Claude Code Prompt — Mandarin app catch-up (mirror Japanese improvements)

## Your approach
**Read the Japanese files first.** Before touching anything in the Mandarin folder, read these files to understand the exact patterns already working in production:
- `/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Japanese Review/japanese-quiz.html`
- `/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Japanese Review/index.html`

Then implement the same patterns in the Mandarin folder. Do not invent new approaches — copy what works.

## Use plan mode before making any changes.

---

## Files you will edit
- `mandarin-quiz.html` (quiz app)
- `index.html` (deployed vocab app — same as mandarin-vocab.html)
- `mandarin-vocab.html` (local mirror of index.html)
- `service-worker.js` if needed

## Mandarin data model (hardcoded CHAPTERS array, no localStorage vocab)
Each word: `{ h, p, pos, m, ex, exm }`
- `h` = hanzi (e.g. 你好)
- `p` = pinyin (e.g. nǐ hǎo)
- `pos` = part of speech
- `m` = English meaning
- `ex` = example sentence in Chinese
- `exm` = example sentence English translation

Unlike Japanese, vocab is **hardcoded** — there is no user-generated word list and no jp-vocab-deck equivalent. The Gist sync (Task 4) will only sync settings/streak, not vocab.

---

## TASK 1 — Flashcard/Quiz tab switcher

### Problem
The Mandarin vocab app (`index.html`) has no way to navigate to the quiz. The quiz is a separate URL (`mandarin-quiz.html`). On mobile, users can't discover the quiz at all.

### What to build
Add a tab switcher to **both** `index.html`/`mandarin-vocab.html` AND `mandarin-quiz.html`, exactly matching the Japanese implementation.

Read `japanese-quiz.html` and `japanese-vocab.html` to find the exact HTML/CSS/JS for the FLASHCARDS | QUIZ switcher. Copy it verbatim, changing:
- Link targets: `./index.html` (flashcards) and `./mandarin-quiz.html` (quiz)
- Active state: `index.html` highlights FLASHCARDS, `mandarin-quiz.html` highlights QUIZ

The top bar in the Mandarin vocab app currently has: hamburger menu · VOCABULARY title · theme toggle · card counter. The tab switcher should be centred in the top bar, same as Japanese. If it's too crowded, the VOCABULARY label can be removed (the tabs make it self-evident).

---

## TASK 2 — Next button after correct answer in mandarin-quiz.html

### What to build
Exactly matching the Japanese quiz behaviour:
- **Correct answer** → show example sentence → show "Next →" button → wait for user tap to advance
- **Wrong answer** → auto-advance after ~800ms (no Next button)
- Enter/Space keyboard shortcut triggers Next when visible

Read `japanese-quiz.html` to find the `btnNext` implementation (HTML, CSS, JS including the `advanceQuestion()` shared function). Mirror it exactly into `mandarin-quiz.html`.

---

## TASK 3 — Show example sentence after correct answer

### What to build
After the user answers correctly, reveal a section below the answer options showing:
1. The example sentence in Chinese (`word.ex`)
2. The pinyin of the **quiz word** itself (e.g. `word.p`) as a label — shown as italic muted text
3. The English translation of the example (`word.exm`)
4. Then the Next → button (from Task 2)

Only show this if `word.ex` exists and is non-empty.

### Visual style
Match the Japanese example sentence reveal style from `japanese-quiz.html`:
- Chinese sentence: large, Noto Serif SC font, white/gold
- Pinyin: small italic muted gold, below the Chinese sentence  
- English: small italic muted, below pinyin
- Reveal block hidden by default, shown on correct answer

### Note on Mandarin vs Japanese
The Mandarin data does NOT use `[hanzi|pinyin]` bracket annotation in `ex` — it's plain Chinese text. So no ruby rendering needed for the example sentence. Just display `word.ex` as plain text. The pinyin label comes from `word.p` (the quiz word's own pinyin), not from inline annotations.

---

## TASK 4 — Gist sync for streak (cross-device / PWA sync)

### Background
iOS PWA and Safari have completely separate localStorage — same origin, different contexts. The only reliable cross-context sync is GitHub Gist. The Japanese app already has full Gist sync infrastructure. For Mandarin, since vocab is hardcoded, we only need to sync **settings** (streak).

### What to build

#### 4a. Settings modal in index.html/mandarin-vocab.html
Read `index.html` (Japanese) to find the Settings modal. Build a minimal version for Mandarin containing:
- GitHub Token input (stored as `mn-vocab-gh-token`)
- Gist ID input (stored as `mn-vocab-gh-gist`)
- "⎘ Copy setup link" button — copies `https://clipt25.github.io/mandarin-vocab/?gist=GIST_ID`
- Save button

Settings gear icon in the top bar opens the modal.

#### 4b. Gist sync — settings only
The Gist file for Mandarin stores only: `{ version: 1, settings: { streak: { current, best, date } } }`

Implement in `index.html`/`mandarin-vocab.html`:
- `syncNow()` — GET the Gist file, merge settings (streak: take max current, max best, later date), PUT back, update `mn-vocab-last-sync`
- Auto-sync on page load if token is configured and last sync was >5 min ago
- ✓ Synced / ⚠ stale / Sync Error badge in top bar (same pattern as Japanese)
- First-time banner: "Tip: set up Gist sync in Settings to keep your streak in sync across devices." — one-shot, dismissed to `mn-banner-sync-dismissed`

#### 4c. Auto-fill Gist ID from setup link
On page load, check `?gist=` URL param. If present: store to `mn-vocab-gh-gist`, open Settings modal, pre-fill the field, clean the URL.

#### 4d. Streak stored in Gist-synced settings
Move Mandarin streak storage from standalone keys (`mn-streak-current`, etc.) into the sync object, same pattern as Japanese:

```js
function getMnDeckRaw() {
  try {
    const raw = localStorage.getItem('mn-vocab-settings');
    return raw ? JSON.parse(raw) : null;
  } catch(e) { return null; }
}
function saveMnDeckRaw(deck) {
  try { localStorage.setItem('mn-vocab-settings', JSON.stringify(deck)); } catch(e) {}
}
function getStreak() {
  const deck = getMnDeckRaw();
  const s = (deck && deck.settings && deck.settings.streak) || {};
  return {
    current: parseInt(s.current || '0', 10),
    best:    parseInt(s.best    || '0', 10),
    date:    s.date || null,
  };
}
function recordActivity() {
  // same pattern as japanese-quiz.html — writes to mn-vocab-settings.settings.streak
  // also writes standalone mn-streak-* keys as secondary fallback
}
```

#### 4e. Quiz pulls streak from Gist on load
In `mandarin-quiz.html`, add `syncStreakFromGist()` — a read-only GET of the Gist (same as `japanese-quiz.html`'s implementation). Call it on init before rendering the streak chip. 3-second timeout. No pushing from quiz.

Add the sync badge to the quiz top bar (glyph-only: `✓` or `⚠`) — same as Japanese quiz.

---

## TASK 5 — Mobile friendliness pass

After the above tasks, do a final mobile pass on both `index.html` and `mandarin-quiz.html`:
- No horizontal scrolling on iPhone (max-width: 100%, overflow-x: hidden on body)
- All tap targets at least 44px tall
- Top bar elements don't overlap at 390px width
- Quiz answer options are full width on mobile
- Example sentence text wraps cleanly (no overflow)

---

## localStorage keys (Mandarin)
- `mn-vocab-settings` — `{ version:1, settings:{ streak:{ current, best, date } } }` (new, replaces standalone keys)
- `vocab-mode` — theme (already exists)
- `mn-vocab-gh-token`, `mn-vocab-gh-gist`, `mn-vocab-last-sync` — Gist sync (new)
- `mn-quiz-round-length` — already exists
- `mn-streak-current`, `mn-streak-best`, `mn-streak-date` — secondary fallback (keep writing, stop reading)
- `mn-banner-sync-dismissed` — one-shot banner

---

## Constraints
- Do not break existing quiz logic (chapter filter, drill selection, results screen)
- Do not push from quiz — read-only Gist pull only
- Vocab data stays hardcoded in CHAPTERS array — do not move it to localStorage
- Mirror all changes: `index.html` ↔ `mandarin-vocab.html` (they should be identical)
- Single commit at the end

---

## Deploy
```bash
cd "/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Mandarin Review"
git add index.html mandarin-vocab.html mandarin-quiz.html service-worker.js
git commit -m "Mandarin catch-up: tab switcher, Next button, example sentences, Gist streak sync, mobile fixes"
git push origin main
```

---

## Checklist
- [ ] FLASHCARDS | QUIZ tab switcher on both vocab and quiz pages
- [ ] Next button appears after correct answer in quiz
- [ ] Example sentence (Chinese + pinyin label + English) shown on correct answer
- [ ] No example shown if word.ex is empty
- [ ] Settings modal with Gist token + ID + Copy setup link
- [ ] Gist sync for settings/streak (auto on load, badge in top bar)
- [ ] Quiz pulls streak from Gist on load (read-only)
- [ ] Streak stored in mn-vocab-settings.settings.streak
- [ ] Auto-fill Gist ID from ?gist= URL param
- [ ] No horizontal scroll or overlap on iPhone (~390px)
- [ ] index.html and mandarin-vocab.html are identical
- [ ] Pushed to GitHub Pages
