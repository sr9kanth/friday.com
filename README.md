# Friday.com

A Monday.com-style project board that runs entirely as a single HTML file — no framework, no build step, no server required.

## Features

- **Board view** — tasks organized into color-coded groups with status, priority, owner, dates, effort, timeframe, and timeline columns
- **Gantt view** — timeline visualization across all groups with month headers
- **Drag & drop** — reorder tasks or move them across groups
- **Task panel** — click any task to open a side panel with full details, owner picker, and comments
- **Custom statuses** — add your own status values (beyond Done / In progress / Stuck) via the status chip popup; each gets a unique color; persisted per board
- **Custom owners** — add people by initials via the owner picker ("+ New person…"); owners are board-specific and do not bleed across boards
- **Delete task** — trash button at the bottom of the task side panel
- **Delete group** — trash icon on the group header (appears on hover), removes the group and all its tasks
- **Dark mode toggle** — moon/sun button in the topbar; preference saved to localStorage
- **Boards (snapshots)** — save named snapshots, load them, overwrite them, or push them to GitHub; fetch GitHub snapshots to restore on any device
- **Backup JSON** — export full board state as a `.json` file
- **Load JSON** — restore a board from a previously exported `.json` backup
- **Export HTML** — export current board as a standalone `.html` file with all data baked in
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
| Shared snapshots | GitHub repo → `snapshots/{id}.json` |
| App code | GitHub repo → `index.html` (with default demo data, not personal data) |

Personal board data is **never** pushed to the public GitHub Pages URL — only the app shell with default demo tasks is published. Use **Boards → Push to GitHub** to back up individual snapshots.

## Sharing Between Users

Currently sharing requires manually exporting and sending files:
- **Export HTML** — sends a fully self-contained board anyone can open in a browser
- **Backup JSON → Load JSON** — send a JSON file; recipient loads it via Load JSON
- **GitHub snapshots** — push a snapshot to a shared repo; others fetch it via Boards → Fetch

See the planning notes for a future real-time sharing implementation.

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
