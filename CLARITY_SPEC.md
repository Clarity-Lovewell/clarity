# Clarity — Life Journal · SPEC v1.3

> **This document is the single source of truth for the Clarity app.**
> Upload this file alongside `index.html` (and optionally a JSON backup) at the start of every Claude session.
> Claude will update this file directly — you do not need to edit it manually.

---

## Purpose

A personal life journal for recording, organising, and understanding everyday life — capturing events, tasks, appointments, problems, learnings, and observations, then surfacing patterns across them over time.

The core difference from a to-do app or calendar: items belong to *living threads* that evolve over time. A project isn't a checkbox — it's an accumulating record. A problem isn't a post-it — it's a thread you work through until resolved. AI assists with processing raw captures, refining entries, generating thread summaries, and detecting patterns across logs — but the human is always in control.

### Relationship to Covenant

Clarity is a **sister app** to Covenant (the prayer journal), not an extension of it. They share the same technical architecture (single HTML file, localStorage, GitHub Pages, Anthropic API) but have distinct purposes and aesthetics:

- **Covenant** — interior/spiritual life. Candlelight warmth, serif devotional tone.
- **Clarity** — exterior/operational life. Clean daylight, practical editorial tone.

They should never be merged. The inbox *pattern* is the same, the *content* is different. A user might open Covenant in the evening and Clarity in the morning — they feel like one ecosystem but serve separate functions.

**Note on scripture and faith features:** Any request to add scripture, devotional content, or interior spiritual life features to Clarity should be flagged as Covenant territory. A "Faith" *life area* (external practices like retreats, books, church attendance) is acceptable in Clarity if the user requests it as an editable area.

---

## Design Decisions

### Aesthetic

- Five selectable visual themes (set in Settings):
  - **Forest** (default): dark brown bg `#0e0a06`, forest green accent `#5a9a68`, warm cream text — earth, bark, moss
  - **Cloud**: warm ivory bg `#f7f3ee`, dark brown/warm slate accent `#6a5040` — no green, warm daylight
  - **Ocean**: deep blue-black bg `#060c18`, cyan accent `#38c8e8` — cool and focused
  - **Dawn**: light lavender-white bg `#faf7ff`, soft purple accent `#7a50c0` — airy morning feel
  - **Stone**: near-black bg `#111010`, warm gold accent `#c89848` — minimal, timeless
- Themes use `[data-theme="..."]` CSS selectors with CSS variable overrides
- Theme persists in `settings.theme`
- Fonts: **DM Serif Display** (headings, titles, thread names) + **DM Sans** (body, UI labels) — warm editorial serif paired with clean humanist sans. Not geometric/futuristic.
- No gamification, no streaks, no badges
- Minimalist but not sterile — colour and symbol accents (◈ app mark, ✦ AI/patterns, ⚡ inbox, ◈ life) provide warmth without clutter

### Navigation

- Bottom nav: Home · Inbox · Calendar · Threads · Patterns
  - "Threads" was formerly labelled "Life" in the nav — renamed for clarity
- Settings opens as a full-screen overlay via **gear icon top-right of Home screen** (not in bottom nav)
- Area detail and Thread detail are sub-screens of Threads (no separate nav tabs)
- Daily Review opens as a full-screen overlay (button on Home screen)
- Thread detail can be reached from Home (today items), Inbox (after filing), Calendar (tap item), or Threads (area or all-threads view)

### Organisational Model

- **Areas** (7 default, now fully editable): Home · Work · Health · Mind · Money · People · Growth
  - Each has an icon, name, and short description
  - Areas are editable in Settings: rename, change icon, change description, add new, or remove
  - Removing an area does not delete threads — it removes the area tag from those threads
  - Areas are tags — threads can belong to multiple areas
- **Threads** are the core unit — a named, typed, living record:
  - Types: Project · Problem · Goal · Habit · Reference
  - Statuses: Active · Watching · Resolved · Archived
  - Each thread has a title, type, areas (multi-tag), status, entries array, and optional AI-generated summary
- **Entries** belong to threads and accumulate over time:
  - Types: Task · Appointment · Log · Update · Learning · Solution · Note
  - Tasks and Appointments have optional due date + time, and a completed toggle
  - All other types are plain records
- **Threads screen** has two views:
  - **By Area** — 2-column grid of area cards, tap to drill into that area's threads
  - **All Threads** — flat list of all threads across all areas, shows area tags, grouped active/resolved
- Inbox → file to existing thread OR create new thread in one step

### Entry Types

| Type | Icon | Use |
|---|---|---|
| Task | ☐ | Something to do, with optional due date — has a completion toggle |
| Action | → | A specific next step or decision — appears in Calendar if dated, has completion toggle |
| Appointment | 📅 | Scheduled event — has due date + time, completion toggle |
| Log | 📓 | Record of something that happened (the source material for pattern detection) |
| Update | ↗ | Progress note on a project or situation |
| Learning | 💡 | Something learned, understood, or researched |
| Solution | ✓ | Resolution to a problem — what worked |
| Note | 📌 | General capture that doesn't fit another type |

### Thread Types

| Type | Icon | Use |
|---|---|---|
| Project | 🗂️ | Multi-step effort with a defined outcome |
| Problem | 🔍 | Something being investigated or worked through |
| Goal | 🎯 | Ongoing aspiration or target |
| Habit | 📈 | Something practised regularly — useful for logging |
| Reference | 📚 | Information to keep and return to |

### Thread Status

- **Active** — currently being worked on or monitored
- **Watching** — paused or waiting on something external
- **Resolved** — completed or no longer relevant
- **Archived** — closed, kept for record

### Dev Notes (Settings)

- Stored in `db.devNotes[]` — survives export/import
- Located in Settings overlay, below Data section
- Each note has a **checkbox** to mark as done/tested
- Ticking a note strikes through its text and records `checkedAt` timestamp
- Checked notes are grouped into a collapsible "✓ Show archived (n)" section
- Unchecked notes can be edited inline; checked notes show "done" date
- Notes are never auto-deleted — archive by ticking, remove manually with Remove button
- **Export as Text** button — downloads all dev notes (open + done) as a plain `.txt` file
- Use for: feature ideas, bugs noticed, things to build next

### Voice Capture

- 🎙 microphone button in the Quick Capture box (left of the Capture button)
- Uses **Web Speech API** (SpeechRecognition) — no API key required, works in Chrome/Edge/Safari
- Tap mic to start: requests microphone permission via `getUserMedia` first (required on Android), then button pulses red (⏹), "● Listening…" status appears below textarea
- Live transcription appears directly in the capture textarea as you speak; final transcripts accumulate across utterances
- Tap ⏹ to stop, or tap Capture → to submit — submitting also stops recording
- **Android Chrome fix**: uses `continuous: false` with automatic restart on `onend` to simulate continuous mode (Android Chrome silently ignores `continuous: true`)
- `no-speech` and `aborted` errors restart silently; only genuine permission errors alert the user
- Language: `en-AU` (can be updated in code)
- If browser does not support SpeechRecognition, shows an alert explaining the limitation

### Calendar

- Accessed via **Calendar** tab in bottom nav (📅)
- **Week view** — opens on current week, always defaults to today's date selected
- 7 day chips (Mon–Sun) across the top — tap any day to see that day's items
- ← → navigation between weeks; week label shows date range (e.g. "10–16 Mar 2026")
- Dot indicator on days that have scheduled items
- Today chip highlighted in accent colour; selected day filled solid
- Items shown below the day chips, sorted by time (untimed items last)
- Each item shows: time (if set), entry text, thread name
- Colour-coded left border: Task = blue, Action = accent, Appointment = green
- Completed items shown struck-through at reduced opacity
- Tapping an item opens its parent thread
- Shows **Task**, **Action**, and **Appointment** entry types that have a `dueDate` set

### Notifications

- Permission requested via **Notifications** section in Settings
- Once granted: on each app open and each `visibilitychange` (app brought to foreground), `scheduleNotifications()` runs
- Scans today's Task, Action, and Appointment entries with a `dueTime` that haven't been completed
- Schedules a `setTimeout` browser notification for each, firing at the exact due time
- Notification title = entry content; body = thread name + formatted time
- Previous timers cleared and rescheduled each time to avoid duplicates
- **Limitation**: fires only while the app is open or running as an installed PWA in the background — no push server, no server-side scheduling
- If permission is denied, Settings shows a clear message to enable in browser settings
- "Reschedule Today" button appears once granted — useful after adding new timed items mid-day

### AI Layer

- Requires Anthropic API key (stored in localStorage)
- AI features:
  - **Inbox processing**: parses raw capture → suggests type, refined text, due date, area, matching/new thread — shows original + editable refined draft side by side
  - **Entry assist**: in Add Entry modal — polishes note, suggests type and due date; if a date is suggested it auto-opens the schedule section
  - **Thread summary**: summarises the full entry thread in 2–3 sentences, second person ("You started this…"), stored on the thread
  - **Pattern analysis**: reads recent Log entries across all threads, identifies 2–4 genuine patterns — shown on the Patterns screen
- All AI is assistive — user reviews/edits before anything is saved
- Manual mode available for all features without API key

### Add Entry / File Entry Modals — Compact Design

- The **Schedule (date & time)** section in both Add Entry and File Entry modals is collapsed by default using a `<details>` element
- Tapping "📅 Schedule (date & time)" expands to reveal date and time inputs
- The section auto-opens when: AI suggests a date/time, or an existing entry being edited already has a date
- This keeps modals compact and focused — most entries don't need scheduling

### Daily Review (3 steps)

1. **Today's Recap** — shows tasks completed vs total for today; log rows to record what happened (filed as `[Daily Log]` inbox captures)
2. **Inbox** — shows unprocessed count; shortcut to Inbox screen
3. **Notice Any Patterns?** — free-text rows for observations (saved directly to `db.patterns`)

### Home Screen

- Greeting with name and date
- **Quick capture** textarea — ⌘/Ctrl+Enter or button to capture; lands in inbox
- **Today** section — tasks and appointments due today, surfaced from all threads; inline completion toggle; tap to open thread
- **Inbox preview** — last 3 unprocessed captures with link to Inbox
- **Daily Review** button — opens review overlay
- **Settings gear** — top-right of home bar, opens settings overlay

### Data Export

- **Export JSON** — full backup of all data (threads, entries, inbox, patterns, dev notes, settings)
- **Export CSV** — tabular export of all threads and entries; columns: Thread, Thread Type, Thread Status, Areas, Entry Type, Content, Due Date, Due Time, Completed, Created. Threads with no entries appear as a single row. Useful for spreadsheet analysis.
- **Import JSON** — restores a full JSON backup

---

## Data Structure

All data stored in `localStorage` under key `clarity_v1`.

```
settings: {
  name,
  apiKey,
  theme,          // 'forest' | 'cloud' | 'ocean' | 'dawn' | 'stone'
}

areas: [{
  id, name, icon, description
}]
// Default 7: home, work, health, mind, money, people, growth
// Fully editable by user — add, rename, remove

threads: [{
  id, title,
  type,           // 'project' | 'problem' | 'goal' | 'habit' | 'info'
  areaIds[],      // multi-tag
  status,         // 'active' | 'watching' | 'resolved' | 'archived'
  entries[],
  summary,        // AI-generated, stored as plain text
  createdAt
}]

entries: [{
  id, type, content,
  // type: 'task' | 'action' | 'appointment' | 'log' | 'update' | 'learning' | 'solution' | 'note'
  dueDate,        // YYYY-MM-DD or null
  dueTime,        // HH:MM or null
  completed,      // bool — for task, action, and appointment types
  editedAt,       // optional timestamp of last edit
  createdAt
}]

inbox: [{
  id, rawText, createdAt, processed
}]

patterns: [{
  id, text, createdAt
}]

devNotes: [{
  id, text, createdAt,
  checked,      // bool — ticked when done/tested
  checkedAt     // timestamp or null
}]
```

---

## Screens

| Screen | How to reach | Description |
|---|---|---|
| Home | Nav: Home | Greeting, quick capture (with voice), today items, inbox preview, daily review |
| Inbox | Nav: Inbox | Unprocessed captures — AI process or file manually |
| Calendar | Nav: Calendar | Week view of scheduled tasks, actions, and appointments |
| Threads — By Area | Nav: Threads | 2-column grid of life areas |
| Threads — All Threads | Nav: Threads → All Threads tab | Flat list of all threads across all areas, area tags shown |
| Area Detail | Tap any area card | Threads within that area, grouped active/resolved |
| Thread Detail | Tap any thread | Entry thread + AI summary + add/edit entries |
| Patterns | Nav: Patterns | User-recorded patterns + AI analysis of log entries |
| Daily Review | Home button | 3-step guided overlay |
| Settings | Gear icon on Home | Name, API key, theme, life areas, notifications, data export, dev notes |
| Setup | First launch | Name + optional API key |

---

## Key Flows

### Quick Capture → File

1. User types anything in the home capture box → ⌘+Enter or tap Capture
2. Item lands in `db.inbox` unprocessed
3. Inbox badge shows count
4. From Inbox: tap **✦ AI Process** → AI suggests type, refined text, due date, area, thread
5. User reviews editable draft → **Accept & File** → File Entry modal pre-filled with AI suggestions
6. User selects existing thread or names a new one → **File Entry ✓**
7. Entry created, inbox item marked processed

### Manual Filing

1. Tap **File manually** on an inbox item
2. File Entry modal opens with raw text pre-filled
3. User fills in type, thread; date/time are in a collapsible section → File Entry ✓

### Add Entry Directly to Thread

1. Open any thread → **+ Entry** button (header or bottom dashed button)
2. Add Entry modal — type content, select type, optionally expand "📅 Schedule" for date/time
3. **✦ AI Assist** refines the text and suggests type/date if API key set

### Pattern Detection

1. User adds **Log** entries to threads over time (what they ate, how they slept, what happened)
2. On Patterns screen → **✦ AI Analyse** reads recent logs across all threads
3. AI surfaces 2–4 genuine patterns (e.g. "Swimming in the morning correlates with better sleep logs")
4. User can also manually record patterns via **+ Record Pattern**

### Daily Review

1. Tap **✦ Daily Review** on Home
2. Step 1: see today's task completion rate; add free-text logs (→ inbox as `[Daily Log]`)
3. Step 2: review inbox count / shortcut to clear it
4. Step 3: record any patterns noticed today (→ saved directly to patterns)
5. Complete Review — closes overlay

---

## PWA / Mobile

- **Manifest**: Dynamically generated as Blob URL at boot
- **Service Worker**: Registered at boot via Blob URL — caches for offline use
- **App icon**: Generated via Canvas API (◈ symbol in forest green on dark brown)
- **Recommended install**: Open in Chrome on Android → ⋮ → Add to Home Screen → launches fullscreen
- Nav uses `padding-bottom: max(14px, env(safe-area-inset-bottom))`
- Screen content uses `padding-bottom: 130px`
- `min-height: 100dvh` on body

---

## Hosting

Single HTML file (`index.html`) on GitHub Pages.
Repo: `https://github.com/[username]/clarity`
Live URL: `https://[username].github.io/clarity/`
Workflow: Edit → commit via GitHub Desktop or GitHub Mobile → push to main → live ~60 seconds

Sits alongside Covenant as a sister repo on the same GitHub account. No conflict.

---

## Session Workflow

- Chris provides: `index.html` + `CLARITY_SPEC.md` + optional JSON backup at the start of each session
- **For small changes (1–2 features):** single prompt is fine
- **For larger batches (3+ changes):** split into two prompts — first handles code-heavy changes (data structure, new screens, new modals), second handles lighter changes (copy, UI polish, confirmation dialogs)
- Changes requested in plain language; Claude handles the code
- Claude delivers: updated `index.html` + updated `CLARITY_SPEC.md`
- Chris commits both files to GitHub

---

## CHANGELOG

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-03-15 | Initial build: quick capture, inbox with AI processing, 7 life areas, threads (5 types), entries (7 types), task/appointment completion toggles, today view on home, By Area + All Threads views, pattern recording + AI analysis, daily review overlay, AI entry assist, AI thread summaries, 5 themes (Forest/Cloud/Ocean/Dawn/Stone), export/import JSON, PWA |
| v1.1 | 2026-03-15 | Dev notes in Settings (checkbox archive, collapsible archived section, inline edit); voice capture via Web Speech API (live transcription, pulsing mic button, en-AU locale); Android Chrome voice fix (continuous:false + auto-restart); Action entry type; modal X close button |
| v1.2 | 2026-03-15 | Calendar week view (nav tab, day chips, week navigation, colour-coded items, dot indicators); browser notifications (permission flow in Settings, setTimeout scheduling on boot and visibility change, reschedule button) |
| v1.3 | 2026-04-13 | Nav "Life" renamed to "Threads" for clarity; screen header updated to match; thread cards made more compact (tighter padding, smaller title font); CSV export added to Settings > Data (threads + entries tabular); Dev Notes text export button added; Life Areas now fully editable in Settings (edit name/icon/description, add new areas, remove areas); date/time fields in Add Entry, Edit Entry, and File Entry modals collapsed into a "📅 Schedule" `<details>` toggle (auto-opens when AI suggests a date or entry already has one) |

---

## Known Limitations

- No cross-device sync (data stays on one browser/device)
- No search across threads and entries
- No recurring tasks
- Pattern analysis requires manual trigger — not automatic
- Service worker Blob URL may not persist after page close on some browsers
- Notifications only fire while app is open or running as installed PWA — no push server

---

## Planned Features

- [ ] Search across all threads and entries
- [ ] Recurring tasks (daily, weekly)
- [ ] Automatic pattern suggestions after N log entries
- [ ] Thread linking (e.g. a Problem thread linked to a Health area Goal)
- [ ] Weekly summary (AI-generated overview of the week's logs and progress)
- [ ] Export as formatted PDF or Markdown
- [ ] People / contacts section — dedicated notes on specific people (beyond the People area)
- [ ] AI thread/area suggestions from inbox processing (suggest restructuring, new areas)

---

*Clarity SPEC v1.3 — Built with Claude Sonnet, April 2026*
