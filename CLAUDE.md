# Golf Swing Assessor

Side-by-side golf swing analyzer. Upload two swing videos and compare them frame-by-frame; later phases add a MediaPipe pose-skeleton overlay and automatic flaw detection.

## Stack
Single-file vanilla HTML/CSS/JS. No build step, no dependencies (Phase 1). MediaPipe Pose (CDN) is planned for Phase 2.

## Run locally
MediaPipe (Phase 2) needs the page served over HTTP, so run a static server rather than opening the file directly:
```bash
cd "Golf swing assessor" && python3 -m http.server 5800
```
Then open http://localhost:5800 . Preview entry: `golf-swing` (port 5800) in the workspace `.claude/launch.json`.

## Structure
Everything lives in `index.html`:
- **`Lane` class** — one per video column; owns its `<video>`, dropzone, file input, drag/drop + click-to-browse, and `loadedmetadata` wiring.
- **Sync controller** — module-level functions (`play`/`pause`/`restart`/`stepBy`/`seekFraction`/`setSpeed`) that act on every loaded lane at once. The longest video is the master timeline; playback stops when all lanes reach their end (no looping).
- **`tick()`** — `requestAnimationFrame` loop that keeps the scrubber + time readout in sync.

Frame stepping assumes `FPS = 30` (`STEP = 1/FPS`). Adjust `FPS` if real per-frame stepping is needed for known footage.

## Phase status
- **Phase 1 — DONE**: side-by-side upload, synchronized play/pause/restart, scrubber, arrow-key frame stepping, 0.25×/0.5×/1× speed.
- **Phase 2 — planned**: per-lane `<canvas>` overlay; MediaPipe Pose skeleton drawn on each frame (wrists, elbows, shoulders, hips, knees), driven by `requestVideoFrameCallback`.
- **Phase 3 — planned**: per-frame landmark buffering; detect address / top / impact / follow-through; flag club past parallel, over-the-top, early extension, sway, chicken wing; flaw list with click-to-seek timestamps + red joint highlight.
