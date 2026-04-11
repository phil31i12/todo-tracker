# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

TaskBoard is a single-file (`index.html`) shared todo tracker for two business partners. It uses Firebase Realtime Database as the backend and is hosted via GitHub Pages at `https://phil31i12.github.io/todo-tracker`.

No build step, no npm, no bundler — just one HTML file with inline CSS and a `<script type="module">` using Firebase via CDN imports.

## Deployment

Changes go live by pushing to `main`. GitHub Pages serves `index.html` directly from the repo root.

```bash
git add index.html
git commit -m "..."
git push
```

## Architecture

Everything lives in `index.html` in three sections:

1. **CSS** (`<style>`) — CSS custom properties in `:root` control the entire color scheme. All theming changes go there.
2. **HTML** — Three screens rendered via `display:none/flex/block` toggling:
   - `#pw-screen` — password entry (shown on first visit, or if `tb_auth` not in localStorage)
   - `#name-screen` — name entry (shown after auth if `tb_name` not in localStorage)
   - `#app` — main todo board
3. **JS** (`<script type="module">`) — Firebase SDK loaded via CDN (`https://www.gstatic.com/firebasejs/11.6.0/...`). All state is in two variables: `allTodos` (object mirroring Firebase) and `currentFilter`/`currentPerson`.

## Firebase

- **Project:** `todo-tracker-b6ed5`
- **Database URL:** `https://todo-tracker-b6ed5-default-rtdb.europe-west1.firebasedatabase.app`
- **Todos** stored at `/todos/{pushId}` with fields: `title`, `done`, `priority` (`high`/`medium`/`low`), `assignee`, `createdBy`, `createdAt`
- **Password hash** stored at `/_config/pwHash` (SHA-256 of the password, fetched at login time — not in source code)

## Auth flow

Password is verified client-side by SHA-256 hashing the input and comparing against the hash fetched from `/_config/pwHash.json` in Firebase. On success, `tb_auth=1` is written to localStorage. Name is stored in `tb_name`.

## Key patterns

- `renderTodos()` is the single render function — called on every Firebase `onValue` update and on filter changes. It rebuilds both the person-filter buttons and the todo list.
- All window-exposed functions (`addTodo`, `toggleTodo`, `deleteTodo`, `startEdit`, `saveTodo`, `cancelEdit`, `setFilter`, `setPerson`) are assigned as `window.X` because they are called from inline HTML `onclick` attributes inside template literals.
- Inline edit rows (`#edit-row-{id}`) are rendered hidden inside each todo item and shown/hidden by `startEdit`/`cancelEdit` without a re-render.
- The `esc()` helper must be used for all user-supplied strings inserted into HTML to prevent XSS.
