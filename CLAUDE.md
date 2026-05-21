# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A static HTML webapp that generates G-code toolpaths for a Duet-based bioprinter. No build step, no server required beyond a static file server.

## Running

```bash
python3 -m http.server 8080
# open http://localhost:8080
```

## Architecture

`index.html` is the shell — it renders the sidebar, topbar, and `#tool-frame` container. Tools are **not** full HTML pages; they are fragments (`<style>` + markup + `<script>`) stored in `tools/`. When a nav item is clicked, `index.html` fetches the fragment, injects its HTML into `#tool-frame`, and manually re-executes any `<script>` tags.

**Consequences of the injection model:**
- Tool fragments inherit all CSS variables (`--bg`, `--surface`, `--accent`, etc.) defined in `index.html` — never redefine them in a tool.
- Tool scripts run after injection, not on `DOMContentLoaded` — don't rely on any page lifecycle events.
- Each tool is self-contained: its `generate()` and `download()` functions live in its own `<script>` block.

## Adding a new tool

1. Create `tools/my-tool.html` as a fragment (no `<html>/<head>/<body>` wrappers).
2. Add an entry to the `TOOLS` array in `index.html`:
   ```js
   { label: 'My Tool', file: 'tools/my-tool.html' }
   ```
3. Follow the existing layout pattern: `<div class="tool">` with `.tool-params` (300 px left column) and `.tool-output` (right column).

## Tool layout conventions

Every tool uses a consistent two-column grid:
- Left: `.tool-params` → stacked `.param-group` blocks, each with `.param-group-label` + `.param-row` inputs
- Right: `.tool-output` → action buttons, `.output-meta` stats line, tab switcher (Preview / G-code), canvas preview, and a readonly `<textarea>` for the G-code

The `switchTab(name)` pattern toggles `.active` on both `.tab-btn` elements and the corresponding panel.

## G-code conventions

- `G90` absolute positioning, `M82` absolute extrusion throughout
- `E` values are cumulative totals — never reset mid-print
- Travel speed: `F600`–`F4800` mm/min; print speed: `F20`–`F600` depending on mode
- Z-hop before every travel move
- Default bed origin: X=100–160 mm, Y=100–140 mm (varies per tool)
- `T0` selects the syringe extruder

## Known gaps

`tools/img-to-gcode.html` is listed in `TOOLS` but does not exist yet — the sidebar will show "tool not found" for that entry.

## External dependencies

- `dxf-parser@1.1.2` — loaded from CDN inside `tools/dxf-to-gcode.html` only
- Google Fonts (JetBrains Mono, Geist) — loaded in `index.html` `<head>`
