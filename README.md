# Friday.com

A Monday.com-style project board that runs entirely as a single HTML file — no framework, no build step, no server required.

## Features

- **Board view** — tasks organized into color-coded groups with status, priority, owner, dates, effort, and timeframe columns
- **Gantt view** — timeline visualization across all groups
- **Drag & drop** — reorder tasks or move them across groups
- **Task panel** — click any task to open a side panel with full details, owner picker, and comments
- **Custom owners** — add new people by initials via the owner picker ("+ New person…"); persists to localStorage
- **Dark mode toggle** — moon/sun button in the topbar; preference saved to localStorage
- **Boards** — save named snapshots of the board and restore them later
- **Backup JSON** — export full board state as a `.json` file
- **Load JSON** — restore a board from a previously exported `.json` backup
- **Export HTML** — export the current board as a standalone `.html` file (bakes in all data)
- **Save to GitHub** — push the board HTML directly to a GitHub repo via the Contents API (requires a personal access token)
- **Reset** — clears localStorage and reloads with the default Product Launch demo data

## Default Data

On a fresh load (or after Reset), the board shows a sample **Product Launch** project with 17 tasks across 5 groups: Planning, Design, Development, Testing, and Launch.

## Persistence

All board state is saved to `localStorage` under the key `fridayBoardState`. The baked-in HTML is only used when no localStorage state exists (e.g. first load or after Reset).

## GitHub Save (optional)

Click **Save** in the topbar to push the current board HTML to GitHub Pages. You'll be prompted for:
- GitHub username
- Repository name
- File path (e.g. `index.html`)
- Personal access token (with `repo` scope)

Settings are stored in `localStorage`. The token is never sent anywhere except the GitHub API.

## Local Development

No build step needed — just open `index.html` in a browser, or serve it with any static server:

```bash
python3 -m http.server 8080
```
