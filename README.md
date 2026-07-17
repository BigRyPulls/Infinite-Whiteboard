# Infinite Whiteboard

A production-quality, infinite canvas whiteboard that runs entirely in the
browser — no accounts, no cloud, no backend database. Draw, type, drop in
images, organize spatially, bookmark locations with **Waypoints**, and save
your board as a portable JSON file. Built with vanilla TypeScript and
Canvas2D for maximum performance with minimal dependencies.

![status](https://img.shields.io/badge/status-ready-39ff9d)

---

## Quick start

```bash
Extract the .zip and run the below from root
npm install
npm run dev
```

Vite prints a **Local** and one or more **Network** URLs. Open the Network
URL from any phone, tablet, or laptop on the same Wi‑Fi/LAN to use the board
from another device — no extra configuration needed.

```
  Infinite Whiteboard — local dev server
  ────────────────────────────────────────────
  Local:    http://localhost:5173
  Network:  http://192.168.1.42:5173

  Any device on the same Wi-Fi/LAN can open the Network URL above.
```

### Production build + LAN hosting

```bash
npm run build     # type-check + bundle to /dist
npm run serve     # serve /dist on 0.0.0.0:4000 (Express)
```

`npm run serve` prints the same kind of Local/Network URL banner. Both the
dev server and the production server bind to `0.0.0.0`, so they're reachable
from other devices on your network by default — not just `localhost`.

There is no internet connectivity required at any point; everything —
assets, fonts, board data — is served from your machine.

---

## Feature overview

- **Infinite canvas** — pan/zoom without limits (1% – 1000% zoom), true world
  coordinates, camera-only navigation (objects never move to accommodate
  scrolling).
- **Tools** — Selection (move/resize/rotate/marquee), Brush (vector strokes +
  point-level eraser), Text (inline editable, monospace by default), Shape
  (rectangle/ellipse/line/arrow), Image (import/drag‑drop/paste).
- **Waypoints** — named camera bookmarks with animated (300ms ease‑in‑out)
  jump-to navigation, sorted alphabetically, collapsible panel.
- **Undo/redo** — 100-entry command history covering create, delete, move,
  resize, rotate, draw, erase, text edits, imports, and reordering.
- **Persistence** — JSON board files (File System Access API when available,
  otherwise a normal download/upload), localStorage autosave every 30s and
  on tab close, crash recovery banner on next launch.
- **Export** — PNG and SVG (viewport or full board extent), plus JSON project
  export.
- **LAN hosting** — both the Vite dev server and the production Express
  server bind to `0.0.0.0` and print your LAN IP automatically.
- **Dark, Matrix-inspired UI** — glassmorphism-lite floating panels, neon
  green/cyan/purple accents, JetBrains Mono / IBM Plex Mono typography, 150–
  250ms ease-out motion only.

---

## Keyboard & input reference

| Action | Input |
|---|---|
| Pan | Space + drag · Middle-mouse drag · Two-finger touch drag |
| Zoom | Mouse wheel · Ctrl + wheel · Trackpad pinch · Two-finger touch pinch |
| Select tool / Pan / Brush / Text / Shape | `V` / `H` / `B` / `T` / `S` |
| Multi-select | Shift + click, or Shift + drag marquee |
| Duplicate while dragging | Alt + drag |
| Edit text | Double-click a text element |
| Undo / Redo | Ctrl+Z / Ctrl+Shift+Z (or Ctrl+Y) |
| Copy / Paste | Ctrl+C / Ctrl+V (also accepts OS clipboard images) |
| Duplicate | Ctrl+D |
| Delete | Delete / Backspace |
| Select all | Ctrl+A |
| Save / Open | Ctrl+S / Ctrl+O |
| Cancel / deselect | Escape |
| Right click | Context menu (copy, paste, delete, duplicate, reorder, create waypoint) |

---

## Architecture

```
src/
  main.ts                 Bootstraps the App against #board-canvas / #ui-root
  app/
    App.ts                Composition root — owns every subsystem & the render loop
    Camera.ts              Viewport transform (x, y, zoom) + eased animation
    CanvasManager.ts        Canvas element, DPR scaling, resize handling
    Renderer.ts             Retained-scene rendering, grid, selection overlays
    ElementPainter.ts       Pure element → Canvas2D paint logic (shared by Renderer + Export)
    Scene.ts                Authoritative element store, layering, dirty flag
    SpatialIndex.ts          Dynamically-growing quadtree for viewport culling
    Selection.ts             Selected-id set + bounds helper
    History.ts               Undo/redo command stack + reusable command classes
    Input.ts                 Pointer/wheel/keyboard → camera nav + tool dispatch
    ImageLoader.ts            Decodes/caches image bitmaps, thumbnails
    WaypointManager.ts        Named camera bookmarks + animated jumps
    StorageManager.ts         JSON save/load, autosave, crash recovery
    ExportManager.ts          PNG / SVG export
  models/
    Board.ts                Serializable board file format
    Element.ts                Element union type + bounds math
    Waypoint.ts                Waypoint type
    CameraState.ts             Camera state + zoom clamping
  tools/
    Tool.ts                  Shared Tool/ToolContext interfaces
    PanTool.ts, SelectionTool.ts, BrushTool.ts, TextTool.ts, ShapeTool.ts, ImageTool.ts
  ui/
    UI.ts                    UI composition root + context menu + recovery banner
    Toolbar.ts, Sidebar.ts, StatusBar.ts, PropertyPanel.ts, ContextMenu.ts, Toast.ts
  styles/
    theme.css                Centralized design tokens (CSS variables)
    layout.css                Structural placement of floating panels
    components.css            Reusable panel/button/input/toast styling
server/
  server.js                 Minimal Express static server for production hosting
scripts/
  print-lan.js               Prints Local/Network URLs before dev/serve starts
```

### Data flow

`App` is the single composition root: it constructs the `Camera`, `Scene`,
`Selection`, `History`, `ImageLoader`, `Renderer`, `WaypointManager`,
`StorageManager`, `ExportManager`, every `Tool`, `Input`, and the `UI`. Tools
never reach into `App` directly — they receive a narrow `ToolContext`
interface, which keeps them independently testable and swappable.

The render loop (`requestAnimationFrame`) only repaints when something
changed: a camera pan/zoom/animation tick, a scene mutation, a selection
change, or a tool requesting a redraw for an in-progress gesture (marquee,
live stroke, drag). Idle boards do zero rendering work per frame.

### Why a quadtree?

`SpatialIndex` is a quadtree whose root doubles in size (re-rooting the
existing tree as one quadrant) whenever an element lands outside current
bounds — so there's no hard limit on how far you can place objects from the
origin. Viewport culling and click hit-testing both query this index instead
of scanning every element, which is what keeps interaction smooth with
thousands of objects on the board.

### Why elements aren't rasterized

Strokes are stored as vector point lists, shapes as parametric primitives,
and text stays live/editable — nothing is baked to pixels until you
explicitly export a PNG. This keeps memory low, keeps everything crisp at
any zoom level, and is what makes the point-level eraser possible (erasing
deletes points from the array; it doesn't paint over pixels).

---

## Performance notes

- **Viewport culling** — `Scene.queryVisible()` asks the quadtree for only
  the elements intersecting the (padded) visible world rect; off-screen
  elements never reach the painter.
- **Render gating** — the app tracks a single `needsRender` flag set by
  camera/scene/selection change events and tool overlay requests; the
  animation loop skips `Renderer.render()` entirely when nothing changed,
  which is the practical Canvas2D equivalent of "dirty rectangles" (true
  partial-canvas compositing isn't worthwhile here since a single
  `ctx.fillRect` + culled redraw of the visible set is already cheap).
- **Point sampling** — the brush tool only appends a new stroke point once
  the pointer has moved a few screen pixels (scaled by zoom), which keeps
  stroke arrays compact without visible faceting.
- **Cheap history** — undo/redo commands capture only the fields that
  changed (a moved element's `x/y`, a resized element's `width/height`,
  etc.), never a full-scene snapshot, so history stays fast regardless of
  board size.
- **Image decoding** — images are decoded once via `createImageBitmap`
  (off the main thread where supported) and cached by source URL; the same
  bitmap is reused for every element referencing that image.
- **HiDPI-aware canvas** — the backing store is sized to
  `CSS size × devicePixelRatio` (capped at 3×) exactly once per resize, so
  drawing always happens in a single consistent coordinate space.

**Target**: 60 FPS with 10,000+ elements, hundreds of images, and thousands
of freehand strokes on mid-range hardware. The live FPS/object-count/memory
readout in the status bar makes it easy to verify this on your own machine
and content.

---

## File format

Boards are saved as a single JSON file:

```json
{
  "formatVersion": 1,
  "metadata": { "name": "Untitled Board", "createdAt": 0, "updatedAt": 0 },
  "camera": { "x": 0, "y": 0, "zoom": 1 },
  "elements": [ /* strokes, text, shapes, images */ ],
  "waypoints": [ /* named camera bookmarks */ ]
}
```

Images are embedded as base64 data URLs directly inside their element, so a
board file is fully self-contained and portable — copy the `.json` file to
another machine and everything (including images) loads with it.

---

## Known limitations

- Recent-files list is metadata-only (name + timestamp) in browsers without
  the File System Access API; reopening still goes through the normal file
  picker.
- SVG export renders shapes/text/strokes/images as SVG primitives but does
  not attempt to reproduce canvas-specific antialiasing/blend behavior
  pixel-for-pixel.
- Real-time multi-user sync is intentionally out of scope for this version;
  the architecture (a single `Scene` + serializable `BoardFile`) is
  structured so a future sync layer could diff/broadcast `Scene` mutations
  without touching rendering or tools.

---

## Tech stack

- TypeScript (strict) + Vite — no framework, no virtual DOM.
- Canvas2D with a retained scene graph and quadtree spatial index.
- Express for the minimal production static file server.
- Zero runtime UI dependencies — every panel, button, and toast is plain
  DOM + CSS variables.
