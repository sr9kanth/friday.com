# CLAUDE.md — Friday.com

## Project overview

**Current version:** `v1.5.1` · `100426 092828` (DDMMYY HHMMSS) — shown in footer.

Single-file vanilla HTML/CSS/JS project board (Monday.com-style). Everything lives in `index.html` — no framework, no build step, no dependencies.

## Architecture

- **`index.html`** — the entire app: styles, markup, and scripts in one file
- No modules, no imports, no bundler
- All state lives in plain JS globals (`TASKS`, `STREAMS`, `KNOWN_OWNERS`, `AV_COLORS`, `STATUS_COLORS`, `nextId`)
- `localStorage` key: `fridayBoardState` — serialized on every mutation via `saveLiveState()`
- Theme preference key: `fridayTheme` — `'dark'` or `'light'`
- Supabase JS loaded via CDN (`@supabase/supabase-js@2`) in `<head>`

## Key globals

| Variable | Description |
|---|---|
| `TASKS` | Array of task objects |
| `STREAMS` | `{ groupName: colorHex }` — defines groups |
| `KNOWN_OWNERS` | Array of owner initials — derived from task `ld[]` arrays on load (board-specific) |
| `AV_COLORS` | `{ initials: colorHex }` — avatar colors |
| `STATUS_COLORS` | `{ statusName: colorHex }` — custom + built-in status colors |
| `nextId` | Auto-increment counter for task IDs |
| `currentView` | `'main'` or `'gantt'` |
| `collapsed` | `{ groupName: bool }` — collapsed state |
| `panelTaskId` | Currently open side panel task id (or null) |
| `dragTaskId` | Task id being dragged (or null) |
| `shareCode` | Active Supabase share code (or null); persisted in `localStorage` → `fridayShareCode` |
| `shareUpdatedAt` | Human-readable timestamp of last push; persisted in `localStorage` → `fridayShareUpdatedAt` |

## Task object shape

```js
{
  id: Number,
  st: String,        // group name (stream)
  nm: String,        // task name
  ld: [String],      // owner initials array (up to 4 chars each)
  ef: String,        // effort (e.g. "2d")
  tf: String,        // timeframe (e.g. "1 week")
  s: String,         // start date "YYYY-MM-DD"
  e: String,         // end date "YYYY-MM-DD"
  status: String,    // any key from STATUS_COLORS, or ""
  pri: String,       // "High" | "Medium" | "Low" | ""
  comments: [{ author: String, text: String, ts: String }]
}
```

## Important patterns

### Version stamp
- `APP_VERSION` and `APP_UPDATED` constants at the very top of the `<script>` tag
- `APP_UPDATED` format: `DDMMYY HHMMSS` — update manually on every meaningful release
- Rendered into `#appVersion` span in the topbar via `DOMContentLoaded`
- Run `date "+%d%m%y %H%M%S"` to get the current timestamp

### Adding new state that must persist
1. Add to `saveLiveState()` serialization object
2. Add to the IIFE at the top of the script that reads from localStorage (use replace, not merge)
3. Add to `buildExportHtml()` regex replacements so Export HTML bakes it in
4. Add to `buildPublicHtml()` with default values so GitHub Save doesn't expose personal data
5. Add to `saveCurrentBoard()` and `updateSnapshot()` so snapshots carry it
6. Add to `loadSnapshot()` so loading restores it

### Rendering
- `renderView()` → `renderMain()` or `renderGantt()`
- `renderMain()` pre-seeds groups from `Object.keys(STREAMS)` before iterating tasks (so empty groups still render)
- Same pattern in `renderGantt()`
- Status chips use `statusChipStyle(st)` for inline styles — do NOT use hardcoded CSS classes like `c-done`
- Avatar initials display full text (up to 4 chars); font-size 8px

### Modals
- `showInputModal(title, placeholder, callback)` — generic single-input modal
- `confirmInputModal()` saves `const cb = _inputModalCb` BEFORE calling `closeInputModal()` (which nulls the ref), then calls `cb(val)`. Do not change this order.

### Status system
- `STATUS_COLORS = { name: hexColor }` — built-in: Done, In progress, Stuck; user can add more via popup
- `statusChipStyle(st)` — returns inline style string for any status
- `openStatusPopup(e, id)` — shows all STATUS_COLORS + clear + "＋ Add status…"
- Adding a custom status calls `addCustomStatus(id)` → `showInputModal` → assigns color from palette → calls `setStatus`
- `buildFilters()` dynamically includes all STATUS_COLORS keys in the status dropdown

### Owner system
- `KNOWN_OWNERS` is derived from task `ld[]` arrays on every page load — never accumulated globally
- Adding a new person via "+ New person…" also assigns them to the task immediately
- `AV_COLORS` maps initials → hex color; persisted in localStorage and snapshots

### Share system (Supabase)
- `SUPABASE_URL` / `SUPABASE_ANON_KEY` — constants at top of script; anon key is safe to be public
- `getSb()` — lazy-initialises the Supabase client; returns null if URL is still placeholder
- `generateShareCode()` — creates a random 6-char code, inserts board snapshot into `boards` table
- `pushSharedBoard()` — upserts current board state to existing code
- `joinBoard()` — fetches by code, calls `loadBoardSnapshot()`
- `stopSharing()` — clears local `shareCode` / `shareUpdatedAt` from localStorage
- `boardSnapshot()` — returns `{ TASKS, STREAMS, KNOWN_OWNERS, AV_COLORS, STATUS_COLORS, nextId }`
- `loadBoardSnapshot(d)` — replaces all live globals from a snapshot object, then saves + re-renders
- All async share functions wrapped in try/catch — errors shown in modal status line, never silent

**Supabase table schema:**
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

### GitHub Save vs Export HTML
- **`buildExportHtml()`** — bakes current TASKS/STREAMS/owners/statuses into the HTML (used by Export HTML button)
- **`buildPublicHtml()`** — bakes DEFAULT Product Launch data into the HTML (used by Save to GitHub button)
- Neither replaces `SUPABASE_URL` / `SUPABASE_ANON_KEY` — credentials are preserved in both exports
- This separation means the public GitHub Pages URL never exposes personal board data

### GitHub Snapshots
- `saveSnapshotToGithub(id)` — pushes `snapshots/{id}.json` to the repo
- `fetchSnapshotsFromGithub()` — lists `snapshots/` dir, imports any not already in localStorage (by id)
- Snapshots include: TASKS, STREAMS, KNOWN_OWNERS, AV_COLORS, STATUS_COLORS

### Reset
- Clears `localStorage.removeItem('fridayBoardState')` then `location.reload()`
- On reload with no localStorage, the baked-in `TASKS` and `STREAMS` are used (Product Launch demo)

### Delete
- `deleteTask(id)` — splices from TASKS, saves, re-renders
- `deleteGroup(stream)` — confirms, filters TASKS, deletes from STREAMS, saves, re-renders
- Group delete button is hidden until hover (`.grp-label:hover .grp-del { opacity: 1 }`)

## Default demo data

17 tasks across 5 groups: Planning (3), Design (4), Development (5), Testing (3), Launch (2).
Owners: AL (`#a25ddc`), JM (`#00c875`), SR (`#fdab3d`). nextId starts at 18.

## Deployment

GitHub Pages via the in-app Save button, or manually:
```bash
git add index.html && git commit -m "..." && git push
```

Deployed at: `https://sr9kanth.github.io/friday.com/`

## Known conflicts: in-app Save vs git push

The Save button pushes `buildPublicHtml()` directly to GitHub via the Contents API. If you have unpushed local commits, this causes a rebase conflict. Resolution:
```bash
git checkout --theirs index.html
git add index.html
GIT_EDITOR=true git rebase --continue
git push
```
