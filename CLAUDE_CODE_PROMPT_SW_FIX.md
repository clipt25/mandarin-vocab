# Claude Code Prompt — Fix PWA updates not reflecting on iOS (Mandarin)

## Files to edit
- `service-worker.js`
- `index.html`
- `mandarin-vocab.html`
- `mandarin-quiz.html`

## Same two bugs as the Japanese app — same fixes, different file names and cache key.

---

## Fix 1 — Rewrite `service-worker.js`

Replace the entire file with:

```js
const CACHE = 'mn-vocab-v3';
const ASSETS = [
  './index.html',
  './mandarin-vocab.html',
  './mandarin-quiz.html',
];

self.addEventListener('install', e => {
  e.waitUntil(
    caches.open(CACHE)
      .then(cache => cache.addAll(ASSETS))
      .then(() => self.skipWaiting())
  );
});

self.addEventListener('activate', e => {
  e.waitUntil(
    caches.keys()
      .then(keys => Promise.all(
        keys.filter(k => k !== CACHE).map(k => caches.delete(k))
      ))
      .then(() => self.clients.claim())
      .then(() =>
        self.clients.matchAll({ type: 'window', includeUncontrolled: true })
          .then(clients => clients.forEach(c => c.postMessage({ type: 'SW_UPDATED' })))
      )
  );
});

self.addEventListener('fetch', e => {
  const url = new URL(e.request.url);

  if (!url.origin.includes('github.io') && !url.origin.includes('fonts.googleapis.com') && !url.origin.includes('fonts.gstatic.com')) {
    return;
  }

  // Network First for HTML
  if (e.request.mode === 'navigate' || e.request.headers.get('accept')?.includes('text/html')) {
    e.respondWith(
      fetch(e.request)
        .then(response => {
          if (response.ok) {
            const clone = response.clone();
            caches.open(CACHE).then(cache => cache.put(e.request, clone));
          }
          return response;
        })
        .catch(() => caches.match(e.request))
    );
    return;
  }

  // Cache First for fonts and static assets
  e.respondWith(
    caches.match(e.request)
      .then(cached => cached || fetch(e.request).then(response => {
        if (response.ok) {
          const clone = response.clone();
          caches.open(CACHE).then(cache => cache.put(e.request, clone));
        }
        return response;
      }))
  );
});
```

---

## Fix 2 — Update SW registration in all HTML files

In `index.html`, `mandarin-vocab.html`, AND `mandarin-quiz.html`, find the SW registration script near `</body>` and replace with:

```html
<script>
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('./service-worker.js', {
      updateViaCache: 'none'
    }).catch(err => console.warn('SW registration failed:', err));
  });

  navigator.serviceWorker.addEventListener('message', event => {
    if (event.data && event.data.type === 'SW_UPDATED') {
      window.location.reload();
    }
  });
}
</script>
```

---

## Deploy

```bash
cd "/Users/geoffreyjames/Desktop/Claude Obsidian/1-Projects/Mandarin Review"
git add service-worker.js index.html mandarin-vocab.html mandarin-quiz.html
git commit -m "Fix PWA updates on iOS: Network First for HTML + updateViaCache none"
git push origin main
```

---

## After deploying — one-time manual reset on iPhone

1. Open regular Safari → go to `https://clipt25.github.io/mandarin-vocab/`
2. Wait for the page to reload once (the new SW activating)
3. Close Safari fully
4. Open the PWA from your home screen
5. ✅ Future updates now load automatically

---

## Checklist
- [ ] `service-worker.js` rewritten, cache bumped to `mn-vocab-v3`
- [ ] `updateViaCache: 'none'` in all 3 HTML file registrations
- [ ] `SW_UPDATED` reload listener in all 3 HTML files
- [ ] Pushed to GitHub Pages
