# 🎯 FocusFlow

**A simple, beautiful timer for deep work.**

FocusFlow is a distraction-free Pomodoro-style productivity timer — focus
sessions, intentional breaks, task tracking, daily goals, streaks, and
analytics, wrapped in a calm, modern interface. It's a single self-contained
web app: no build step, no backend, no dependencies to install. Open it and
go.

**[▶ Try the live demo](https://claude.ai/artifact/UrPm6uNaNaSSD22gCoUz8a)**

## ✨ Features

- ⏱ **Accurate, reliable timer** — timestamp-based, so it stays correct even
  when the tab is backgrounded or the computer sleeps
- 🔁 **Smart Pomodoro cycle** — configurable focus/short break/long break
  durations, with optional auto-start between sessions
- ✅ **Task tracking** — link tasks to sessions and watch estimated vs.
  completed sessions add up
- 📊 **Analytics dashboard** — daily focus time, weekly chart, plain-language
  insights, longest session, and more
- 🔥 **Streaks & daily goals** — build consistency without pressure, with a
  satisfying completion animation when you hit your goal
- 🧘 **Focus Mode** — a fullscreen, distraction-free view for when you really
  need to lock in
- 🌗 **Light / dark / system themes** with 6 accent colors
- 🔔 **Notifications & generated sounds** — no external audio files, nothing
  to license
- ⌨️ **Full keyboard control** — `Space` start/pause, `R` reset, `S` skip,
  `F` focus mode, `Esc` exit, `M` mute
- 📱 **Installable PWA** — add it to your home screen, works offline
- 💾 **Private by default** — all data stays in your browser's `localStorage`;
  no account, no server, no tracking

## Quick start

```bash
git clone https://github.com/yourusername/focusflow.git
cd focusflow
python3 -m http.server 8080
# open http://localhost:8080
```

No install, no build — it's plain HTML/CSS/JS. See **Deploy** below to put
it on the web.

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

Any static file server works — see **Quick start** above (`npx serve .` is
an equally good alternative to the Python server shown there).

Opening `index.html` directly via `file://` also works for the timer itself,
but the service worker (offline support) and manifest only activate when
served over `http://localhost` or `https://`.

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

## Contributing

Issues and pull requests are welcome. Since there's no build step, the whole
app lives in `index.html` — open it, find the relevant section (timer engine,
tasks, analytics, or settings, each clearly commented), and edit directly.
