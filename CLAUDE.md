# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A static HTML webapp that generates G-code toolpaths for a Duet-based bioprinter. No build step, no server required beyond a static file server.

## How to Run

```bash
python3 -m http.server 8080
# open http://localhost:8080
```

## Architecture

`index.html` is the shell — it renders the sidebar, topbar, and `#tool-frame` container. Tools are **not** full HTML pages; they are fragments (`<style>` + markup + `<script>`) stored in `tools/`. When a nav item is clicked, `index.html` fetches the fragment, injects its HTML into `#tool-frame`, and manually re-executes any `<script>` tags.

**Consequences of the injection model:**
- Tool fragments inherit all CSS variables (`--bg`, `--surface`, `--accent`, etc.) defined in `index.html` — never redefine them in a tool.
- Palette is night mode with pastel blue (`--accent`/`--blue`), green (`--green`), orange (`--orange`) plus `--on-accent` for text on accent fills; each has a `-dim` variant. Canvas code should read them via `getComputedStyle` instead of hardcoding hex.
- Every tool must also work when its file is opened directly (no `index.html`): start the fragment with the "Standalone fallback" `<script>` found at the top of `tools/img-to-gcode.html`, which defines the palette only if `--bg` is missing. Keep its values in sync with `:root` in `index.html`.
- Wrap tool scripts in an IIFE and use `addEventListener` — the script is re-executed on every navigation, so top-level `const`/`let` would throw on the second load. `tools/dot-grid.html` is the reference (slider-style fields, auto-generate, localStorage persistence). It has two print modes, Dots and Lines (lines = the `grid-generator.html` G-code, driven by the same grid box); each mode has its own `buildDots()`/`buildLines()` and its own fields via `modes: [...]` in `GROUPS`.
- Tool scripts run after injection, not on `DOMContentLoaded` — don't rely on any page lifecycle events.
- Each tool is self-contained: its `generate()` and `download()` functions live in its own `<script>` block. All tools share the same UI pattern: slider-style `.field` rows (range via `data-min`/`data-max`), auto-generate on every change (no Generate button), Download/Copy buttons and the G-code safety warning.

## Adding a new tool

1. Create `tools/my-tool.html` as a fragment (no `<html>/<head>/<body>` wrappers).
2. Add an entry to the `TOOLS` array in `index.html`:
   ```js
   { label: 'My Tool', file: 'tools/my-tool.html', icon: 'MY_TOOL.jpg' }   // icon is optional (file in the project root)
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

## External dependencies

- `dxf-parser@1.1.2` — loaded from CDN inside `tools/dxf-to-gcode.html` only
- Google Fonts (JetBrains Mono, Geist) — loaded in `index.html` `<head>`
