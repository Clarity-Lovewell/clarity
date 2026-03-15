# Clarity — Life Journal · SPEC v1.0

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

- Bottom nav: Home · Inbox · Life · Patterns
- Settings opens as a full-screen overlay via **gear icon top-right of Home screen** (not in bottom nav)
- Area detail and Thread detail are sub-screens of Life (no separate nav tabs)
- Daily Review opens as a full-screen overlay (button on Home screen)
- Thread detail can be reached from Home (today items), Inbox (after filing), or Life (area or all-threads view)

### Organisational Model

- **Areas** (7 default, not editable in v1): Home · Work · Health · Mind · Money · People · Growth
  - Each has an icon, name, and short description
  - Areas are tags — threads can belong to multiple areas
- **Threads** are the core unit — a named, typed, living record:
  - Types: Project · Problem · Goal · Habit · Reference
  - Statuses: Active · Watching · Resolved · Archived
  - Each thread has a title, type, areas (multi-tag), status, entries array, and optional AI-generated summary
- **Entries** belong to threads and accumulate over time:
  - Types: Task · Appointment · Log · Update · Learning · Solution · Note
  - Tasks and Appointments have optional due date + time, and a completed toggle
  - All other types are plain records
- **Life screen** has two views:
  - **By Area** — 2-column grid of area cards, tap to drill into that area's threads
  - **All Threads** — flat list of all threads across all areas, shows area tags, grouped active/resolved
- Inbox → file to existing thread OR create new thread in one step

### Entry Types

| Type | Icon | Use |
|---|---|---|
| Task | ☐ | Something to do, with optional due date — has a completion toggle |
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

### AI Layer

- Requires Anthropic API key (stored in localStorage)
- AI features:
  - **Inbox processing**: parses raw capture → suggests type, refined text, due date, area, matching/new thread — shows original + editable refined draft side by side
  - **Entry assist**: in Add Entry modal — polishes note, suggests type and due date
  - **Thread summary**: summarises the full entry thread in 2–3 sentences, second person ("You started this…"), stored on the thread
  - **Pattern analysis**: reads recent Log entries across all threads, identifies 2–4 genuine patterns — shown on the Patterns screen
- All AI is assistive — user reviews/edits before anything is saved
- Manual mode available for all features without API key

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
  dueDate,        // YYYY-MM-DD or null
  dueTime,        // HH:MM or null
  completed,      // bool — for task and appointment types
  editedAt,       // optional timestamp of last edit
  createdAt
}]

inbox: [{
  id, rawText, createdAt, processed
}]

patterns: [{
  id, text, createdAt
}]
```

---

## Screens

| Screen | How to reach | Description |
|---|---|---|
| Home | Nav: Home | Greeting, quick capture, today items, inbox preview, daily review |
| Inbox | Nav: Inbox | Unprocessed captures — AI process or file manually |
| Life — By Area | Nav: Life | 2-column grid of 7 life areas |
| Life — All Threads | Nav: Life → All Threads tab | Flat list of all threads, area tags shown |
| Area Detail | Tap any area card | Threads within that area, grouped active/resolved |
| Thread Detail | Tap any thread | Entry thread + AI summary + add/edit entries |
| Patterns | Nav: Patterns | User-recorded patterns + AI analysis of log entries |
| Daily Review | Home button | 3-step guided overlay |
| Settings | Gear icon on Home | Name, API key, theme selection, export/import |
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
3. User fills in type, date, thread → File Entry ✓

### Add Entry Directly to Thread

1. Open any thread → **+ Entry** button (header or bottom dashed button)
2. Add Entry modal — type content, select type, optional date/time
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

---

## Known Limitations

- No cross-device sync (data stays on one browser/device)
- No calendar view — due dates on tasks/appointments but no month/week grid
- No search across threads and entries
- No recurring tasks
- Pattern analysis requires manual trigger — not automatic
- Areas cannot be renamed or reordered in v1
- Service worker Blob URL may not persist after page close on some browsers

---

## Planned Features

- [ ] Search across all threads and entries
- [ ] Calendar / week view for tasks and appointments
- [ ] Recurring tasks (daily, weekly)
- [ ] Automatic pattern suggestions after N log entries
- [ ] Rename and reorder life areas
- [ ] Thread linking (e.g. a Problem thread linked to a Health area Goal)
- [ ] Weekly summary (AI-generated overview of the week's logs and progress)
- [ ] Export as formatted PDF or Markdown
- [ ] Reminders / notifications for due tasks (requires PWA notification permission)

---

*Clarity SPEC v1.0 — Built with Claude Sonnet, March 2026*
