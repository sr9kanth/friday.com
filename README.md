# Friday.com

**v1.5.3** · `100426 095007`

A Monday.com-style project board that runs entirely as a single HTML file — no framework, no build step, no server required.

## Features

- **Board view** — tasks in color-coded groups with status, priority, owner, dates, effort, timeframe, and timeline columns
- **Gantt view** — timeline visualization across all groups with month/week headers
- **Drag & drop tasks** — reorder tasks within a group or move them between groups
- **Drag & drop groups** — reorder groups by dragging the ⠿ grip handle on any group header
- **Editable timeline bars** — drag any bar (in both Main table and Gantt) to move a task; drag the right edge to resize (change end date)
- **Task panel** — click any task to open a side panel with full details, owner picker, and comments
- **Custom statuses** — add your own status values beyond Done / In progress / Stuck; each gets a unique color
- **Custom owners** — add people by initials (up to 4 chars); owners are board-specific and don't bleed across boards
- **Delete task** — trash button in the task side panel
- **Delete group** — hover the group header to reveal a trash icon; removes the group and all its tasks
- **Dark mode** — toggle in the topbar; preference saved to localStorage
- **Boards (snapshots)** — save named snapshots locally; load, overwrite, or delete at any time
- **Backup JSON / Load JSON** — export and restore full board state as a file
- **Export HTML** — export current board as a standalone `.html` with all data baked in
- **Share board** — Supabase-backed sharing with a 6-char code; collaborators join via the Join tab
- **Live sync** — auto-pushes changes 3s after every edit; auto-pulls every 15s; footer shows `⟳ Live sync on` and last synced timestamp
- **Visitor logging** — every page load is logged to Supabase (`page_visits` table) with timestamp, URL, referrer, and active share code
- **Smart empty states** — contextual messages when board is empty, all tasks are done, or tasks are overdue
- **First-run tour** — 7-step spotlight walkthrough shown on first visit
- **Save to GitHub** — publishes the app shell (with default demo data) to GitHub Pages; personal board data stays in localStorage only
- **Version stamp** — `v{version}` in the footer so you can always confirm which version is live

## Timeline Editing

Timeline bars in **both** the Main table and Gantt view are interactive:

| Action | Result |
|---|---|
| Drag bar body | Moves the task (shifts start + end date) |
| Drag right edge | Resizes the task (changes end date only) |

Changes snap to whole days and are saved automatically on mouse release.

## Live Sync Flow

Once a share code is active:

| Action | When |
|---|---|
| Auto-push your edits | 3s after any change |
| Auto-pull others' changes | Every 15s (only if remote is newer) |
| Manual push | Share modal → Push update |
| Manual pull | Share modal → Pull latest |

Changes from one window appear in all others within ~15s automatically.

## Supabase Setup (required for Share + Visitor Logging)

1. Create a free project at [supabase.com](https://supabase.com)
2. In the SQL editor, run:
```sql
-- Board sharing / live sync
create table boards (
  code text primary key,
  data jsonb not null,
  updated_at timestamptz default now()
);
alter table boards enable row level security;
create policy "public read"   on boards for select using (true);
create policy "public insert" on boards for insert with check (true);
create policy "public update" on boards for update using (true);

-- Visitor logging
create table page_visits (
  id bigint generated always as identity primary key,
  visited_at timestamptz default now(),
  user_agent text,
  url text,
  referrer text,
  share_code text
);
alter table page_visits enable row level security;
create policy "public insert" on page_visits for insert with check (true);
```
3. In `index.html`, replace the two constants near the top of the `<script>` tag:
```js
const SUPABASE_URL = 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key';  // "publishable" key in newer Supabase UI
```

The anon/publishable key is safe to be public — Supabase Row Level Security is the access control layer.

## Persistence Model

| What | Where |
|---|---|
| Current board state | `localStorage` → `fridayBoardState` |
| Board snapshots | `localStorage` → `boardSnapshots` |
| Theme preference | `localStorage` → `fridayTheme` |
| GitHub token & settings | `localStorage` → `ghToken`, `ghOwner`, `ghRepo`, `ghFile` |
| Active share code | `localStorage` → `fridayShareCode` |
| Last sync timestamp | `localStorage` → `fridayShareUpdatedAt` |
| Last remote timestamp | `localStorage` → `fridayShareRemoteUpdatedAt` |
| Shared boards | Supabase → `boards` table |
| App code | GitHub repo → `index.html` (default demo data only — not personal data) |

## Default Data

On a fresh load (or after Reset), the board shows a **Product Launch** demo with 17 tasks across 5 groups: Planning, Design, Development, Testing, Launch.

## Local Development

```bash
python3 -m http.server 8080
# open http://localhost:8080
```

## Deployment

```bash
git add index.html && git commit -m "..." && git push
```

## Deployed

`https://sr9kanth.github.io/friday.com/`
