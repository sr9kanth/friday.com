# CLAUDE.md — Friday.com

## Project overview

Single-file vanilla HTML/CSS/JS project board (Monday.com-style). Everything lives in `index.html` — no framework, no build step, no dependencies.

## Architecture

- **`index.html`** — the entire app: styles, markup, and scripts in one file
- No modules, no imports, no bundler
- All state lives in plain JS globals (`TASKS`, `STREAMS`, `KNOWN_OWNERS`, `AV_COLORS`, `nextId`)
- `localStorage` key: `fridayBoardState` — serialized on every mutation via `saveLiveState()`
- Theme preference key: `fridayTheme` — `'dark'` or `'light'`

## Key globals

| Variable | Description |
|---|---|
| `TASKS` | Array of task objects |
| `STREAMS` | `{ groupName: colorHex }` — defines groups |
| `KNOWN_OWNERS` | Array of owner initials strings |
| `AV_COLORS` | `{ initials: colorHex }` — avatar colors |
| `nextId` | Auto-increment counter for task IDs |
| `currentView` | `'main'` or `'gantt'` |
| `collapsed` | `{ groupName: bool }` — collapsed state |
| `panelTaskId` | Currently open side panel task id (or null) |
| `dragTaskId` | Task id being dragged (or null) |

## Task object shape

```js
{
  id: Number,
  st: String,        // group name (stream)
  nm: String,        // task name
  ld: [String],      // owner initials array
  ef: String,        // effort (e.g. "2d")
  tf: String,        // timeframe (e.g. "1 week")
  s: String,         // start date "YYYY-MM-DD"
  e: String,         // end date "YYYY-MM-DD"
  status: String,    // "Done" | "In progress" | ""
  pri: String,       // "High" | "Medium" | "Low" | ""
  comments: [{ author: String, text: String, ts: String }]
}
```

## Important patterns

### Adding new state that must persist
1. Add to the `saveLiveState()` serialization object
2. Add to the IIFE at the top of the script that reads from localStorage
3. Add to `buildExportHtml()` regex replacements so Export HTML bakes it in

### Rendering
- `renderView()` — dispatches to `renderMain()` or `renderGantt()`
- `renderMain()` pre-seeds groups from `Object.keys(STREAMS)` before iterating tasks (so empty groups still render)
- Same pattern in `renderGantt()`

### Modals
- `showInputModal(title, placeholder, callback)` — generic single-input modal
- `confirmInputModal()` saves `const cb = _inputModalCb` BEFORE calling `closeInputModal()` (which nulls the ref), then calls `cb(val)`. Do not change this order.

### GitHub Save
- `saveToGithub()` checks for token first; opens settings modal if missing
- `buildExportHtml()` uses regex replace on `document.documentElement.outerHTML` to bake in current state
- **Warning**: if the user clicks Save in-app while the repo has unpushed local changes, it will cause rebase conflicts. Always `git pull --rebase` before pushing after a Save.

### Reset
- Clears `localStorage.removeItem('fridayBoardState')` then `location.reload()`
- On reload with no localStorage, the baked-in `TASKS` and `STREAMS` in the script are used (the Product Launch demo data)

## Default demo data

17 tasks across 5 groups: Planning (3), Design (4), Development (5), Testing (3), Launch (2).
Owners: AL (`#a25ddc`), JM (`#00c875`), SR (`#fdab3d`).

## Deployment

GitHub Pages via the in-app Save button, or manually:
```bash
git add index.html && git commit -m "..." && git push
```

Deployed at: `https://sr9kanth.github.io/friday.com/`
