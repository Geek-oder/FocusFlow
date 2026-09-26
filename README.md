# FocusFlow — A simple timer for deep work

A distraction-free Pomodoro-style focus timer: focus sessions, short/long breaks,
task tracking, daily goals, streaks, analytics, and a distraction-free Focus Mode.
Installable as a Progressive Web App (PWA).

## Project structure

```
focusflow-pwa/
 ├── index.html              # entire app: markup, styles, and logic
 ├── manifest.json           # PWA manifest (name, icons, theme)
 ├── sw.js                   # service worker (offline app-shell caching)
 ├── icon-192.png            # app icon (192×192)
 ├── icon-512.png            # app icon (512×512)
 ├── icon-maskable-512.png   # maskable icon for Android adaptive icons
 ├── favicon-32.png          # browser tab favicon
 └── README.md
```

There's no build step and no `node_modules` — this is intentional. The app is
a single self-contained HTML file (vanilla JS/CSS, no framework, no bundler),
which means:
- Zero install friction to run or deploy.
- Nothing to go stale (no dependency versions to update).
- Easy to hand-edit or extend later, or to migrate into a React/Vite project
  if you outgrow this (the `<script>` block is already organized into clear
  sections — timer engine, tasks, analytics, settings — that map cleanly onto
  hooks/components if you do that migration).

## Run it locally

No install needed. Any static file server works, e.g.:

```bash
cd focusflow-pwa
python3 -m http.server 8080
# open http://localhost:8080
```

or, with Node installed:

```bash
npx serve .
```

Opening `index.html` directly via `file://` also works for the timer itself,
but the service worker (offline support) and manifest only activate when
served over `http://localhost` or `https://`.

## Deploy

Pick any static host — there's no environment variables and no server-side code.

### Netlify
1. Go to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Drag the `focusflow-pwa` folder in
3. Done — live instantly, with a shareable URL

Or via CLI: `netlify deploy --dir=focusflow-pwa --prod`

### Vercel
```bash
cd focusflow-pwa
vercel --prod
```
Vercel auto-detects it as a static site — no build command required.

### Cloudflare Pages
1. Create a new Pages project
2. Upload the `focusflow-pwa` folder directly (or connect a git repo containing it)
3. Build command: *(leave blank)* · Output directory: `/`

### GitHub Pages
```bash
cd focusflow-pwa
git init
git add .
git commit -m "FocusFlow"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```
Then in the repo settings, enable **Pages** → deploy from `main` branch, root folder.
Your app will be live at `https://<username>.github.io/<repo>/`.

> Note: GitHub Pages serves from a subpath (`/repo-name/`). The manifest and
> service worker paths in this project use relative URLs (`./manifest.json`,
> `./sw.js`) specifically so this works without edits.

## Installing as an app (PWA)

Once deployed over HTTPS:
- **Desktop Chrome/Edge:** address bar → install icon, or menu → "Install FocusFlow"
- **Android Chrome:** menu → "Add to Home screen"
- **iOS Safari:** Share → "Add to Home Screen"

Installed, it opens in its own window/icon with no browser chrome, and the
service worker lets the app shell load even without a network connection
(your data was already local-only via `localStorage`, so nothing changes there).

## Data & privacy

All data (tasks, stats, streaks, settings) is stored in the browser's
`localStorage`, scoped per browser/device — there is no account system,
server, or sync between devices. Clearing browser data or using a different
browser/device starts fresh. This is called out in Settings → Data, which
includes a one-click local reset.

## Architecture notes

- **Timer accuracy:** the timer never counts down by subtracting 1 each
  second. It stores an `endAt` timestamp when running and computes remaining
  time as `endAt - Date.now()` on every render tick (via
  `requestAnimationFrame`, throttled to ~4x/second). This keeps it accurate
  through tab backgrounding, JS throttling, and device sleep — on
  reload/focus, `init()` recomputes from the stored `endAt` and auto-completes
  a session that finished while the tab was away.
- **Single source of truth:** one `state` object holds settings, tasks, daily
  stats, streaks, and timer state, persisted to `localStorage` on every
  meaningful change (debounced implicitly by only writing on user actions and
  timer transitions, not every animation frame).
- **Sound:** generated in-browser via the Web Audio API (oscillator tones) —
  no external audio files, so there's nothing to license or host.
- **Theming:** CSS custom properties on `:root`, with light/dark/system modes
  and a user-selectable accent color applied via `style.setProperty`.
- **Resilience:** all `localStorage` reads/writes are wrapped in `try/catch`;
  corrupted or missing state falls back to sane defaults rather than crashing.

## Testing checklist

Before shipping changes, manually verify:

- [ ] Start / pause / resume / reset / skip all behave correctly
- [ ] Timer stays accurate after backgrounding the tab for 60+ seconds
- [ ] Focus → short break → focus → long break cycle transitions correctly
- [ ] Auto-start toggles work for both breaks and focus
- [ ] Page refresh mid-session resumes the countdown accurately
- [ ] Tasks: add, edit, complete, delete, select-as-active all persist
- [ ] Daily goal bar updates and triggers the completion animation once
- [ ] Streak increments once per calendar day, not per session
- [ ] Keyboard shortcuts (Space/R/S/F/Esc/M) work outside of text inputs
- [ ] Settings changes (durations, theme, accent, sound) persist after reload
- [ ] Mobile viewport: timer, tasks, and settings all usable one-handed
- [ ] `npx serve .` → app installs as a PWA and reloads offline after first visit

## Known limitations / future improvements

- **No cross-device sync** — would need a backend + auth (the `state` object
  is already a clean shape to send to an API if you add one later).
- **No ambient sound library** (rain/café/etc.) — left out to avoid bundling
  or linking to non-self-hosted audio assets; would be a good next addition
  using a few licensed/public-domain loops.
- **No charting library** — the weekly chart is hand-rolled CSS bars rather
  than Recharts, to keep the app dependency-free. Swappable later if you
  move to a bundled build.
- **Browser notifications** only fire while the browser process is running
  (backgrounded tab is fine; fully quit browser is not) — this is a platform
  limitation, not implementation-specific.
- **No separate marketing landing page** — the app opens directly into the
  Focus view; a landing page is a natural addition if you deploy this
  publicly and want a pre-app pitch.
