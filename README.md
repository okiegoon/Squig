# Squig — a calm task manager for ADHD brains

A small installable web app, also buildable as an Android app. No accounts,
no cloud — everything lives in your phone's local storage.

> Formerly "Lanturn" — renamed to Squig. If you had it installed under the
> old name, your tasks and sessions carry over automatically the first time
> you open the new version (see `migrateFromLanturn` in `index.html`).

**Live app:** https://okiegoon.github.io/Squig/ (GitHub Pages — see section 2)

## What's in here

- **Four tabs, color-coded**: **Daily** (green), **Weekly** (blue), and
  **Monthly** (purple) each hold their own recurring tasks — swipe one done
  and it vanishes until its next occurrence is actually due, no manual
  re-adding. **Focus** (amber) is where one-off, non-repeating tasks live,
  alongside a Pomodoro-style timer for sitting down and grinding through one.
- **The tab you add from decides the type.** Tap + on the Daily tab and
  you're adding a daily task. Repeat type (daily / weekly / monthly) is
  fixed at creation — to change it, delete and re-add from the right tab.
  The one exception: **weekly tasks can be weekly or bi-weekly** (every 2
  weeks), chosen in the add sheet and editable later in the task's detail
  sheet. Bi-weekly tasks show an "every 2 wks" badge.
- **Swipe to complete**, any tab: swipe a card either direction to mark it
  done. Tap it instead to edit its due date/reminder, jump into a focus
  session, or delete it.
- **Time first, date optional**: Daily/Weekly/Monthly tasks require a time,
  since that's what anchors the repeat schedule. A date is optional: leave
  it empty and the task starts today (or tomorrow, if that time has already
  passed). Add a date to start later — for weekly tasks it sets the weekday,
  for monthly tasks the day of the month. One-off (Focus) tasks can have a
  time and date but don't have to.
- **Focus tab's Timer** sub-view is the original Pomodoro ring (25 / 50 / 10
  min, or a 5 min break) — tap "Start focus session" on any task, from any
  tab, to attach it and jump straight there.

## How reminders work

Every repeating task gets **three alerts per occurrence**, each with
different wording, then silence until the next occurrence:

| Alert | When | Tone |
|---|---|---|
| 1 | at the due time | "Heads up, this one is up now." |
| 2 | +30 min | "Still waiting on this one. Just start." |
| 3 | +2h 30m | "Last reminder for now. Do it now." |

The wording lives in `reminderCopy` and the timing in `NAG_OFFSETS_MS`, both
in `index.html`. One-off Focus tasks never nag.

**Missed occurrences are dropped, not piled up.** If a repeating task is a
day or more overdue (its due time is before midnight today), the missed days
are skipped and it rolls forward to its next occurrence with fresh alerts.
This runs whenever the app is open. In the browser, if several alerts have
come due while the app was closed, only the newest one fires.

Reminders behave differently depending on how you run Squig:

- **Web / installed PWA:** Squig asks for notification permission the first
  time you set a due date, then checks every 20 seconds while the app or tab
  is open (including backgrounded). It **can't** notify you once the app is
  fully closed or the phone restarts — browsers can't schedule that without
  a push server.
- **Android app (Capacitor):** alerts are scheduled with the OS through
  `@capacitor/local-notifications`, so they are designed to fire even when
  the app is closed. To survive days of not opening the app, several
  occurrences are pre-scheduled: **5 days of daily alerts, 3 occurrences of
  weekly, 2 of monthly** (`OCCURRENCES_AHEAD`). Opening the app tops the
  window back up, and completing a task cancels its alerts and schedules the
  next one. This path has not yet been verified on a real device.

---

## 1. Try it on this computer

You need a tiny local server — opening `index.html` directly won't let the
service worker or manifest register properly. Either of these works:

```bash
cd Squig
npx serve .
```

```bash
cd Squig
python -m http.server 8080
```

Then open the address it prints (`serve` usually uses `http://localhost:3000`,
the Python one `http://localhost:8080`).

## 2. Install it on your Android phone as a PWA

A PWA needs HTTPS for Android to let you install it, so the app is served
from GitHub Pages. The repo is `https://github.com/okiegoon/Squig`.

1. On GitHub: **Settings → Pages → Build and deployment → Deploy from a
   branch → `main` → `/ (root)`** → Save. (A free account needs the repo to
   be public.) Serve from the **root**, not `www/`.
2. After a minute or two the site is live at
   `https://okiegoon.github.io/Squig/`.
3. Open that URL on your phone in Chrome → menu (⋮) → **Add to Home screen**
   / **Install app**. It gets its own icon, no browser bar, and works
   offline.

After you push a change, a phone with the app installed may need a full close
and reopen to pick up the new version, because the service worker caches it.

## 3. Build the Android app (`.apk`)

The Capacitor project is already set up in this repo: `capacitor.config.json`
(app ID `com.squig`), the `android/` native project, and `www/` as the web
folder it packages. You need Node and Android Studio installed.

```bash
cd Squig
npm install                # node_modules/ is not in git
npx cap sync android       # copies www/ into the native project + updates plugins
npx cap open android       # opens Android Studio → Run, or Build > Generate APK
```

**Keep `www/` in sync.** Capacitor packages `www/`, not the files in the
repo root. After editing `index.html`, `manifest.json`, `sw.js` or `icons/`,
copy them into `www/` before `npx cap sync`. At the moment `www/index.html`
is a straight copy of the root `index.html`.

Your task data, timer logic, and design are exactly the same code as the web
version. The Android build adds the native-only bits: OS-scheduled
notifications that survive the app being closed (see "How reminders work").

---

## Project layout

| Path | What it is |
|---|---|
| `index.html` | The whole app: markup, styles and script in one file |
| `sw.js`, `manifest.json`, `icons/` | PWA service worker, manifest and icons |
| `www/` | Copy of the web files that Capacitor packages into the Android app |
| `android/` | Capacitor-generated native Android project |
| `capacitor.config.json` | App ID (`com.squig`), name, and web folder (`www`) |
| `package.json` | Capacitor dependencies (`@capacitor/android`, `core`, `cli`, `local-notifications`) |

## Notes on the design choices

- **Dark, low-glare palette** (deep indigo, warm coral accent) — meant to be
  calm rather than alerting, since a wall of red badges reads as pressure.
- **No Doing/Done columns** — earlier versions had a three-column board and
  then a single sorted list; splitting by cadence (Daily/Weekly/Monthly/
  one-off) instead makes each tab small and specific rather than one long
  mixed list.
- **Repeat type is locked at creation.** Simpler data model, and it matches
  how these get used in practice — a "take meds" task isn't going to switch
  from daily to monthly later. Weekly vs bi-weekly is the exception, since
  that's a scheduling preference rather than a different kind of task.
- **Three alerts, then quiet.** Escalating wording gets a task noticed
  without nagging all day, and a missed occurrence is dropped rather than
  left to pile up as guilt.
- **Reminders are local, not push-based.** No backend. That's what limits the
  web version to "app must be open"; the Android build gets around it with
  OS-scheduled notifications instead. Server-sent Push for the PWA would
  need a backend you control and is a possible future addition.
