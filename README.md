# Friday.com

**v1.0** · `090426 235730`

A Monday.com-style project board that runs entirely as a single HTML file — no framework, no build step, no server required.

## Features

- **Board view** — tasks organized into color-coded groups with status, priority, owner, dates, effort, timeframe, and timeline columns
- **Gantt view** — timeline visualization across all groups with month headers
- **Drag & drop** — reorder tasks or move them across groups
- **Task panel** — click any task to open a side panel with full details, owner picker, and comments
- **Custom statuses** — add your own status values (beyond Done / In progress / Stuck) via the status chip popup; each gets a unique color; persisted per board
- **Custom owners** — add people by initials (up to 4 chars) via the owner picker ("+ New person…"); owners are board-specific and do not bleed across boards
- **Delete task** — trash button at the bottom of the task side panel
- **Delete group** — trash icon on the group header (appears on hover), removes the group and all its tasks
- **Dark mode toggle** — moon/sun button in the topbar; preference saved to localStorage
- **Boards (snapshots)** — save named snapshots, load them, overwrite them, or push them to GitHub; fetch GitHub snapshots to restore on any device
- **Backup JSON** — export full board state as a `.json` file
- **Load JSON** — restore a board from a previously exported `.json` backup
- **Export HTML** — export current board as a standalone `.html` file with all data baked in
- **Share board** — generate a 6-char code and push board state to Supabase; collaborators enter the code to load the board; push updates to sync changes
- **Save to GitHub** — push the app shell (with default demo data) to GitHub Pages; your personal board data lives only in localStorage and GitHub snapshots
- **Reset** — clears localStorage and reloads with the default Product Launch demo data

## Default Data

On a fresh load (or after Reset), the board shows a sample **Product Launch** project with 17 tasks across 5 groups: Planning, Design, Development, Testing, and Launch.

## Persistence Model

| What | Where |
|---|---|
| Current board state | `localStorage` → `fridayBoardState` |
| Board snapshots | `localStorage` → `boardSnapshots` |
| Theme preference | `localStorage` → `fridayTheme` |
| GitHub token & settings | `localStorage` → `ghToken`, `ghOwner`, `ghRepo`, `ghFile` |
| Active share code | `localStorage` → `fridayShareCode`, `fridayShareUpdatedAt` |
| Shared boards | Supabase → `boards` table (`code`, `data`, `updated_at`) |
| GitHub snapshots | GitHub repo → `snapshots/{id}.json` |
| App code | GitHub repo → `index.html` (with default demo data, not personal data) |

Personal board data is **never** pushed to the public GitHub Pages URL — only the app shell with default demo tasks is published.

## Sharing Between Users (Phase 1 — Supabase codes)

1. Click **Share** in the topbar
2. On the **Share tab**, click **Generate code** — a 6-char code (e.g. `AB3X7K`) is created and the board is pushed to Supabase
3. Send the code to collaborators
4. They click **Share → Join tab**, enter the code, and click **Join board**
5. After making changes, either party clicks **Push update** to sync the latest state

Anyone with the code can load and overwrite the shared board. Real-time auto-sync is planned for Phase 2.

## Supabase Setup (required for Share feature)

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
3. In `index.html`, replace the two placeholder constants with your project URL and anon key:
```js
const SUPABASE_URL = 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key';
```

The anon key is safe to be public — security comes from Supabase Row Level Security policies.

## GitHub Save (optional)

Click **Save** in the topbar to push the app to GitHub Pages. You'll be prompted for:
- GitHub username
- Repository name
- File path (e.g. `index.html`)
- Personal access token (with `repo` scope)

Settings are stored in `localStorage`. The token is never sent anywhere except the GitHub API.

## Local Development

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

## Known conflicts: in-app Save vs git push

The Save button pushes `buildPublicHtml()` directly to GitHub via the Contents API. If you have unpushed local commits, this causes a rebase conflict. Resolution:
```bash
git checkout --theirs index.html
git add index.html
GIT_EDITOR=true git rebase --continue
git push
```
