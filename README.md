# Friday.com

**v1.2** · `100426 001500`

A Monday.com-style project board that runs entirely as a single HTML file — no framework, no build step, no server required.

## Features

- **Board view** — tasks in color-coded groups with status, priority, owner, dates, effort, timeframe, and timeline columns
- **Gantt view** — timeline visualization across all groups with month headers
- **Drag & drop** — reorder tasks or move them between groups
- **Task panel** — click any task to open a side panel with full details, owner picker, and comments
- **Custom statuses** — add your own status values beyond Done / In progress / Stuck; each gets a unique color
- **Custom owners** — add people by initials (up to 4 chars); owners are board-specific and don't bleed across boards
- **Delete task** — trash button in the task side panel
- **Delete group** — hover the group header to reveal a trash icon; removes the group and all its tasks
- **Dark mode** — toggle in the topbar; preference saved to localStorage
- **Boards (snapshots)** — save named snapshots, load, overwrite, or push/fetch to GitHub
- **Backup JSON / Load JSON** — export and restore full board state as a file
- **Export HTML** — export current board as a standalone `.html` with all data baked in
- **Share board** — Supabase-backed sharing with a 6-char code; collaborators join via the Join tab
- **Live sync** — auto-pushes changes 3s after every edit; auto-pulls every 15s; topbar shows `⟳ Live sync on` and last synced timestamp
- **Save to GitHub** — publishes the app shell (with default demo data) to GitHub Pages; personal board data stays in localStorage only
- **Version stamp** — `v{version} · {DDMMYY HHMMSS}` in the topbar so you can always confirm which version is live

## Live Sync Flow

Once a share code is active:

| Action | When |
|---|---|
| Auto-push your edits | 3s after any change |
| Auto-pull others' changes | Every 15s (only if remote is newer) |
| Manual push | Share modal → Push update |
| Manual pull | Share modal → Pull latest |

Changes from one window appear in all others within ~15s automatically.

## Supabase Setup (required for Share)

1. Create a free project at [supabase.com](https://supabase.com)
2. In the SQL editor, run:
```sql
create table boards (
  code text primary key,
  data jsonb not null,
  updated_at timestamptz default now()
);
alter table boards enable row level security;
create policy "public read"   on boards for select using (true);
create policy "public insert" on boards for insert with check (true);
create policy "public update" on boards for update using (true);
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
| GitHub snapshots | GitHub repo → `snapshots/{id}.json` |
| App code | GitHub repo → `index.html` (default demo data only — not personal data) |

## Default Data

On a fresh load (or after Reset), the board shows a **Product Launch** demo with 17 tasks across 5 groups: Planning, Design, Development, Testing, Launch.

## Local Development

```bash
python3 -m http.server 8080
# open http://localhost:8080
```

## GitHub Save

Click **Save** in the topbar to push the app to GitHub Pages. Prompts for username, repo, file path, and a personal access token (repo scope). Settings stored in localStorage.

## Known conflict: in-app Save vs git push

The Save button pushes via the GitHub Contents API. If you have unpushed local commits this causes a rebase conflict:
```bash
git checkout --theirs index.html
git add index.html
GIT_EDITOR=true git rebase --continue
git push
```

## Deployed

`https://sr9kanth.github.io/friday.com/`
