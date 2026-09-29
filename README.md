# Infinite Whiteboard

> **Personal project / W.I.P.** I built this as a simple local infinite whiteboard for my own use, especially for drawing and sketching on a tablet. It works for the workflow I wanted, but it is not a polished or supported public release.

Infinite Whiteboard is a local-first browser whiteboard with an effectively unlimited canvas, simple drawing and organisation tools, and **Waypoints** for jumping back to useful areas of a large board.

## Why I made it

I wanted a lightweight whiteboard I could run locally, open from a tablet or another device on my network, sketch freely, and keep expanding without running out of space. The main feature I wanted beyond a normal drawing canvas was the ability to save named waypoints and quickly return to different areas of a board.

## Features

- Infinite pan-and-zoom canvas
- Freehand drawing and point-level erasing
- Text, shapes, arrows and images
- Selection, move, resize and rotate tools
- Named **Waypoints** for bookmarking locations on the canvas
- Undo / redo
- Local autosave and crash recovery
- Save/open portable JSON board files
- PNG and SVG export
- Local/LAN hosting for use from a tablet, phone or another computer
- No accounts, cloud service or backend database required

## Running it

The repository includes the complete working project snapshot in `Infinite Whiteboard.zip`.

1. Clone or download the repository.
2. Extract `Infinite Whiteboard.zip`.
3. Open a terminal in the extracted project folder.
4. Install dependencies and start the development server:

```bash
npm install
npm run dev
```

Vite will print a local URL and, when available, a network URL. The network URL can be opened from another device on the same LAN/Wi-Fi.

A production build can also be created and served locally:

```bash
npm run build
npm run serve
```

## Controls

| Action | Input |
|---|---|
| Pan | Space + drag, middle-mouse drag, or two-finger touch drag |
| Zoom | Mouse wheel / trackpad pinch / touch pinch |
| Select / Pan / Brush / Text / Shape | `V` / `H` / `B` / `T` / `S` |
| Multi-select | Shift + click or Shift + drag |
| Undo / Redo | Ctrl+Z / Ctrl+Shift+Z |
| Copy / Paste | Ctrl+C / Ctrl+V |
| Duplicate | Ctrl+D |
| Delete | Delete / Backspace |
| Select all | Ctrl+A |
| Save / Open | Ctrl+S / Ctrl+O |
| Cancel / deselect | Escape |

## Board files

Boards are stored as JSON containing the camera state, elements and waypoints. Images can be embedded directly into the board data so a saved board can remain self-contained.

The app also uses browser-local storage for autosave/recovery. It is designed to keep the workflow local rather than depend on an online account or service.

## Tech

- TypeScript
- Vite
- Canvas2D
- Express for optional local production hosting
- Plain DOM/CSS UI

## Status / limitations

This is a **personal W.I.P. / proof-of-concept**, kept public mainly as a useful personal project and reference.

It worked for the workflow it was built for and I still use it when I want a quick infinite whiteboard, but it has not been packaged or tested like a general-purpose public product. Browser behaviour can vary, especially around file access, touch/pen input and large boards.

There is no multi-user collaboration or cloud sync. Those are intentionally outside the scope of what I built this for.
