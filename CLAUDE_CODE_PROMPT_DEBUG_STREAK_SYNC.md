# Claude Code Prompt — Debug Mandarin streak sync across contexts

## Use plan mode before making any changes.

## The symptom
- Mobile Safari browser: 🔥1 (correct — quiz was done here)
- PWA home screen: 🔥0
- Desktop Chrome: 🔥0

All three open the same origin. The streak earned in mobile Safari is not reaching other contexts.

## Critical background — learn from Japanese fixes first

The Japanese app had the **exact same bug**. Before investigating Mandarin, read:
- `/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Japanese Review/index.html`
- `/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Japanese Review/japanese-quiz.html`

Understand specifically how these were fixed in Japanese:
1. `mergeStreak(a, b)` — takes current/date from later date, best = max
2. `mergeDecks()` — counts streak differences toward the `changed` flag so `syncNow()` adopts a remote streak
3. `saveDeck()` — re-merges persisted streak before writing so stale in-memory deck can't wipe it
4. `syncStreakFromGist()` in quiz — read-only GET, 3s timeout, calls `renderStreakChip()` after pull

Then apply these exact patterns to Mandarin.

---

## Investigation steps

### Step 1 — Read mandarin-quiz.html
Find `recordActivity()`. Verify:
1. Does it write streak to `mn-vocab-settings.settings.streak`? (Not just standalone `mn-streak-*` keys)
2. Does it call `saveMnDeckRaw()` or equivalent to persist?
3. Check: is `mn-vocab-settings` the correct localStorage key, or was a different key used?

### Step 2 — Read index.html (Mandarin vocab app)
Find `syncNow()` or equivalent Gist push function. Check:
1. Does it read from `mn-vocab-settings` and include `settings.streak` in the Gist payload?
2. Does `mergeStreak()` exist? Does it have the "local-wins" bug — i.e. `Object.assign({}, remote, local)` where local streak=0 beats remote streak=1?
3. Does the `changed` flag check include streak differences? If not, a streak-only change won't trigger a Gist push.
4. Does `saveDeck()` / `saveSettings()` re-merge the persisted streak before writing (to prevent in-memory stale data from wiping it)?

### Step 3 — Read syncStreakFromGist() in mandarin-quiz.html
Verify:
1. Does it correctly parse `settings.streak` from the Gist response?
2. Does it call `renderStreakChip()` after a successful pull?
3. Could `renderStreakChip()` be called before the async pull finishes?
4. Is the Gist file name correct? (Should be `mandarin-settings.json`)

### Step 4 — Check Gist sync setup
The sync only works if a GitHub token + Gist ID is configured in localStorage. Each browser context (Safari, PWA, Desktop) has separate localStorage. 
- Check if there's a way to verify whether the Gist token is set in each context
- If Gist sync isn't configured yet in any context, note this clearly — the user needs to set it up once per context via Settings → Copy setup link

---

## Expected root causes (based on Japanese diagnosis)

1. **mergeStreak local-wins bug** — `Object.assign({}, remote, local)` means a context with streak=0 always "wins" over remote streak=1, then pushes 0 back to Gist, corrupting it. Same bug Japanese had.

2. **`changed` flag doesn't include streak** — If `syncNow()` only checks `words`/`tombstones` for changes (Mandarin has neither — it's hardcoded), the changed flag is always false and `syncNow()` never pushes. The streak change never reaches Gist.

3. **Quiz `renderStreakChip()` called before pull resolves** — streak chip renders stale localStorage value, pull finishes and writes new value but chip isn't re-rendered.

4. **Gist sync not configured** — user hasn't set up token + Gist ID in each context yet (separate localStorage per context).

---

## The fix

### Fix 1 — mergeStreak() function
Implement exactly as in Japanese:
```js
function mergeStreak(a, b) {
  // a and b are {current, best, date} objects
  if (!a && !b) return { current: 0, best: 0, date: null };
  if (!a) return b;
  if (!b) return a;
  // Take the record from the later date
  const aDate = a.date || '';
  const bDate = b.date || '';
  let base = aDate >= bDate ? a : b;
  // best is always the max
  return { ...base, best: Math.max(a.best || 0, b.best || 0) };
}
```

### Fix 2 — mergeSettings() must use mergeStreak()
When merging remote and local settings:
```js
function mergeSettings(remote, local) {
  const merged = { ...remote, ...local };
  merged.streak = mergeStreak(remote?.streak, local?.streak);
  return merged;
}
```
NOT `Object.assign({}, remote, local)` which lets streak=0 beat streak=1.

### Fix 3 — changed flag must include streak
When deciding whether to push to Gist, count streak differences:
```js
const remoteStreak = JSON.stringify(remote?.settings?.streak || {});
const localStreak = JSON.stringify(local?.settings?.streak || {});
if (remoteStreak !== localStreak) changed = true;
```

### Fix 4 — saveSettings() must re-merge before writing
Before saving settings to localStorage, re-read the persisted version and merge streak:
```js
function saveSettings(newSettings) {
  const existing = getMnSettingsRaw();
  const merged = mergeSettings(existing?.settings || {}, newSettings);
  saveMnRaw({ ...existing, settings: merged });
}
```

### Fix 5 — syncStreakFromGist() must call renderStreakChip() after pull
In `mandarin-quiz.html`, ensure:
```js
// After successful pull and localStorage write:
renderStreakChip();
// Show badge
badge.textContent = '✓';
badge.hidden = false;
setTimeout(() => { badge.hidden = true; }, 4000);
```

---

## Constraints
- Do not add push logic to mandarin-quiz.html — read-only pull only
- Do not break existing quiz or chapter filter logic
- Mirror all changes: index.html ↔ mandarin-vocab.html (identical)
- Single commit at end

## Deploy
```bash
cd "/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Mandarin Review"
git add index.html mandarin-vocab.html mandarin-quiz.html
git commit -m "Fix Mandarin streak sync: mergeStreak, changed flag, render timing"
git push origin main
```

## Checklist
- [ ] mergeStreak() takes later date's current, max best — no local-wins bug
- [ ] changed flag includes streak differences
- [ ] saveSettings() re-merges persisted streak before writing
- [ ] syncStreakFromGist() calls renderStreakChip() after successful pull
- [ ] Gist file name verified as mandarin-settings.json
- [ ] index.html and mandarin-vocab.html identical after changes
- [ ] Pushed to GitHub Pages

## How to test after fix
1. Mobile Safari: open vocab app → confirm ✓ Synced (pushes streak=1 to Gist)
2. Desktop: open quiz page → wait 3 seconds → streak chip should update to 🔥1
3. PWA: open vocab app → sync → streak should pull to 🔥1
4. Do a quiz on desktop → open vocab app → sync → open mobile quiz → should show updated streak
