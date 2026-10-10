# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A tiny static daily-planner site made of two self-contained HTML files — no build step, no package manager, no test framework, no git repo (this directory is not currently version-controlled).

- `index.html` — the main to-do / planner app
- `about.html` — a placeholder "about me" profile page, linked from `index.html`'s top nav
- `index.backup.html` — a snapshot of a previous version of `index.html`, kept for reference only; it is not linked from anywhere and should not be edited to "fix" the live app

## Running / testing

There is no build or dev server. To view changes:

- Open the file directly in a browser (double-click, or `start index.html` from PowerShell), or
- Serve the folder with any static file server if you need `fetch`-style semantics (not currently used).

There is no linter or test suite — verify changes by opening the page in a browser and exercising the UI (add/check/delete a todo, navigate the calendar, toggle theme, reload to confirm `localStorage` persistence).

## Architecture

Each HTML file is fully self-contained: inline `<style>` block + inline `<script>` IIFE at the bottom of `<body>`. There are no shared `.js`/`.css` files and no imports between the two pages — **`index.html` and `about.html` duplicate their CSS custom-property theme tokens and their dark-mode toggle script verbatim.** Any change to the color tokens (`:root` variables) or the `theme-v1` toggle logic must be made in both files or they will visually diverge.

### Shared design tokens

Both pages define the same CSS variable set on `:root` (light defaults), overridden under `@media (prefers-color-scheme: dark)` guarded by `:root:not([data-theme="light"])`, and again under `:root[data-theme="dark"]` for the explicit toggle. The visual language is a stamped/analog planner look (double borders, monospace labels, a red "stamp" accent) inspired by the Motmot ("모트모트") 텐미닛 플래너 format.

### `index.html` state model

Everything lives in a single IIFE at the bottom of the file, driven by three `localStorage` keys:

- `todos-v1` — array of `{ id, text, done, date, time, priority }`. `date` is an ISO `YYYY-MM-DD` string; every todo belongs to exactly one date. `time` is an optional `HH:MM`. `priority` marks it as one of that date's TOP-3 items.
- `notes-v1` — `{ [isoDate]: string }`, a one-line memo per date.
- `theme-v1` — `"dark" | "light"` (absent = follow system preference).

On load, saved todos are migrated forward with `Object.assign({time:"", priority:false}, t, {date: t.date || todayStr})` so older/missing fields don't break rendering. If no saved data exists at all, three hardcoded seed todos are inserted (see `loadTodos()`/the block right after it).

`render()` is the single entry point called after every mutation (`toggleTodo`, `removeTodo`, `updateTodoTime`, `addTodo`, `togglePriority`, memo edits, calendar navigation). It re-derives everything from `todos`/`notes`/`selectedDate` and re-renders four regions in order: the 00:00–24:00 timetable ruler (`renderTimetable`), the TOP-3 priority box (`renderTop3`, always exactly 3 slots, capped via `togglePriority`'s count check), the scrollable checklist (`renderChecklist`), and the always-visible (non-popup) month calendar (`renderCalendar`). There is no virtual DOM — each render fully rebuilds the relevant container's `innerHTML`/children.

The calendar is deliberately inline/static, not a popup — day cells use real `mouseenter`/`mouseleave` listeners to drive `showPreviewForDate()`, which renders that date's schedule into `.calendar-preview` via `escapeHTML()`-sanitized `innerHTML`. Clicking a day cell sets `selectedDate` and calls `render()`; it does not "close" anything.

### `about.html`

Static profile page with bracketed Korean placeholder text (e.g. `[이름을 입력하세요]`) that has not been filled in — don't assume the names/content are real. It only carries the shared theme-toggle script; it has no todo logic.

## 작업 범위 규칙

- 사용자가 요청한 작업 범위를 벗어나는 추가 작업(리팩토링, 기능 추가, 코드 정리 등)을 임의로 판단해서 진행하지 말 것. 요청받은 것만 정확히 수행한다.
- 범위를 벗어나는 개선이 필요하다고 판단되면, 먼저 제안만 하고 사용자의 승인을 받은 뒤에 진행한다.

## Notes for future changes

- Keep `index.html` single-file: inline `<style>`/`<script>`, no external JS libraries (this was an explicit original requirement).
- If adding fields to a todo object, update the `Object.assign` migration defaults in `loadTodos()`'s mapping step so existing saved data doesn't break.
- `~/.codex/config.toml` and `~/.gemini/` exist on this machine as user-level agent configs; if you want their settings imported into Claude Code, run `/import` rather than having Claude read/port them manually.
