# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A deliberately minimal to-do app: plain HTML/CSS/JS, no frameworks, no build step, no package manager, no tests, not a git repo. Keep it that way (the user's explicit requirement is "extremely simple").

- `index.html` — the entire app (inline `<style>` and `<script>`). Run by opening the file in a browser.
- `mockup.html` — a preview page that shows `index.html` inside CSS-drawn iPhone (402×874) and Samsung Galaxy S26 (360×780, assumed from S25) frames via `<iframe src="index.html">`. Must sit next to `index.html`.

## Architecture (index.html)

- State is a single `todos` array of `{ text, done, category, due }` (`due` is `YYYY-MM-DD` or `''`), persisted to `localStorage` under key `todos`. All storage access is wrapped in try/catch so the app still works when storage is blocked. Items saved before categories/due dates existed are normalized on load (`category`/`due` default to `''`), so keep that normalization when changing the shape.
- `render()` is the single redraw path: it rebuilds the category `<datalist>` and filter `<select>` (`refreshCategories()`), then the `<ul>`. Every mutation does `save(); render();`.
- The DOM is built with `createElement`/`textContent`, never `innerHTML`, so user text (e.g. `<b>`) displays literally. Preserve this.
- Inline editing (`startEdit`) swaps an item's `.main` block for an `.editbox` with text, category and date inputs. Enter saves, Escape cancels, and a `focusout` that leaves the box saves. A blank text keeps the old values.
- Dates are handled as local dates (`todayStr()`, `fmtDate()`) to avoid UTC off-by-one; overdue means not done and `due < today`.
- Layout is mobile-first: `body` max-width 520px, flex-wrap form, 40px tap targets, 16px input font (prevents iOS zoom on focus).

## Testing notes

There are no automated tests. Verify by opening `index.html` in a browser. In the Claude desktop browser pane, local files render as static snapshots: `mockup.html`'s iframes appear blank, and the pane isn't focused, so real `blur` events don't fire (dispatch synthetic events instead). Test data added there persists in `localStorage`, so remove it afterwards.
