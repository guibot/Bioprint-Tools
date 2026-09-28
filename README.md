# Bioprinter Tools

Browser-based G-code generators for extrusion bioprinters. Pick a tool, tweak the parameters, check the toolpath on a preview, and download the G-code. No install and no build step; everything runs in your browser.

<p align="center">
  <a href="https://guibot.github.io/Bioprint-Tools/">
    <img src="Bioprinter_tools.png" alt="Screenshot of Bioprinter Tools: parameters on the left, work-area preview on the right" width="760">
  </a>
</p>

<p align="center">
  <b>Try it live:</b> a fully working version of the app runs in your browser at<br>
  <a href="https://guibot.github.io/Bioprint-Tools/">https://guibot.github.io/Bioprint-Tools/</a>
</p>

<p align="center">
  <img src="heart.png" alt="Toolpath preview of a heart" height="320">
  &nbsp;
  <img src="heart.jpg" alt="A heart drawn with dispensed droplets" height="320">
</p>

> ⚠️ **Always check your G-code with a simulator such as [NC Viewer](https://ncviewer.com/) before printing.**
> We are not responsible for any damage to your printer.

## Tools

| Tool | What it does |
| --- | --- |
| **Dot / Line Grid** | Prints a grid either as discrete **dots** (Z-hop and dwell per dot) or as **lines** (horizontal + vertical strokes, multiple layers). Both modes share the same grid box, position and work area. |
| **Image → G-code** | Turns an image into a toolpath placed on a work-area preview. Modes: **dots**, **crosshatch**, **spiral**, **raster** (serpentine fill at any angle), **contour** (concentric outlines) and **scaffold** (multilayer woodpile with alternating angles). |

All tools share the same behaviour: numeric fields work like sliders (drag across a field to set its value, or click to type), the G-code and preview regenerate automatically on every change, and invalid values show an error instead of producing broken G-code.

### Dot / Line Grid

- **Work area:** set the bed size; the preview shows the grid to scale on the bed, with a *Fit grid* zoom. A warning appears if the grid extends past the bed.
- Parameters are remembered in your browser (`localStorage`); *Reset* restores the defaults.

## Running it

The app loads each tool with `fetch`, so it needs to be served over HTTP; opening `index.html` straight from disk usually will not work.

```bash
python3 -m http.server 8080
# open http://localhost:8080
```

Any static file server works.

Each file in `tools/` is also usable on its own: open it directly in the browser and it applies the same night palette by itself (no server needed, except that the DXF tool loads its parser from a CDN).

## G-code conventions

- `G90` absolute positioning and `M82` absolute extrusion; `E` values are cumulative.
- A Z-hop precedes every travel move.
- `T0` selects the syringe extruder (dot mode).
- Default bed positions and speeds differ per tool. **Review the header, footer, temperatures, speeds and extrusion values against your own machine before running anything.** The generated files include homing (`G28`) and temperature commands that may not suit your setup.

## Project structure

```
index.html          App shell: sidebar, topbar, tool loader, shared theme (CSS variables)
tools/
  dot-grid.html         Dot / Line Grid
  dxf-to-gcode.html     DXF → G-code   (not listed in the sidebar; uses dxf-parser from a CDN)
  img-to-gcode.html     Image → G-code
```

Tools are HTML fragments (`<style>` + markup + `<script>`) that `index.html` injects into the page.

### Adding a tool

1. Create `tools/my-tool.html` as a fragment (no `<html>`, `<head>` or `<body>`).
2. Register it in the `TOOLS` array in `index.html`:
   ```js
   { label: 'My Tool', file: 'tools/my-tool.html' }
   ```
3. Use the shared CSS variables from `index.html` (`--bg`, `--surface`, `--accent`, `--green`, `--orange`, …) instead of hardcoding colors, and wrap the script in an IIFE, since it runs again on every navigation.

`tools/dot-grid.html` is the reference implementation. See `CLAUDE.md` for the full conventions.

## External dependencies

- [`dxf-parser`](https://github.com/gdsestimating/dxf-parser) `1.1.2`, loaded from a CDN by the DXF tool only
- Google Fonts (JetBrains Mono, Geist), loaded by `index.html`
