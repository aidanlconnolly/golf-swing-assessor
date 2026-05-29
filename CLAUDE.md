# Golf Swing Assessor

Side-by-side golf swing analyzer. Upload two swing videos and compare them frame-by-frame; later phases add a MediaPipe pose-skeleton overlay and automatic flaw detection.

## Stack
Single-file vanilla HTML/CSS/JS, no build step. Pose detection uses **MediaPipe Tasks Vision** (`@mediapipe/tasks-vision`) loaded from the jsDelivr CDN via dynamic `import()` inside the module script; the `pose_landmarker_lite` model is fetched from Google's model storage. The script tag is `type="module"`.

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
- **Pose overlay (Phase 2)** — `setupPose()` creates one `PoseLandmarker` (VIDEO mode) per lane sharing a single `FilesetResolver` (kept in `_vision`/`_PoseLandmarker` for reuse). `attachOverlay()` drives detection off `video.requestVideoFrameCallback` during playback, plus `seeked`/`loadeddata` listeners so the skeleton updates while frame-stepping/scrubbing paused. `drawPose()` maps the 33 normalized BlazePose landmarks onto the letterboxed (object-fit: contain) video rect and draws connectors + dots; `KEY_JOINTS` (shoulders/elbows/wrists/hips/knees) are highlighted, and any joints in `lane.flaggedJoints` are drawn red. GPU delegate with CPU fallback. Detection timestamps are forced monotonic per lane.
- **Flaw analysis (Phase 3)** — `analyzeAll()` lazily creates a separate IMAGE-mode `analysisLandmarker`, then for each loaded lane `captureFrames()` scrubs the clip frame-by-frame (`seekTo` awaits `seeked` with a 250ms fallback; capped at 150 frames) and buffers `{t, lm}`. `detectEvents()` finds address/top/impact/follow-through from the wrist-midpoint vertical trajectory + speed. `detectFlaws()` applies pose-only heuristics (past-parallel, over-the-top, early extension, sway, chicken-wing), normalizing thresholds by `bodyScale` (shoulder-center→hip-center distance) and gating on landmark visibility. Lead side comes from the handedness `<select>`. `renderResults()` builds per-lane cards: clickable swing-timeline chips + flaw rows (severity-colored) that seek the lane and set `flaggedJoints` to turn the offending joint red. **These are estimates — the club is not tracked and a single uncalibrated camera angle limits accuracy.**

Frame stepping assumes `FPS = 30` (`STEP = 1/FPS`). Adjust `FPS` if real per-frame stepping is needed for known footage.

## Phase status
- **Phase 1 — DONE**: side-by-side upload, synchronized play/pause/restart, scrubber, arrow-key frame stepping, 0.25×/0.5×/1× speed.
- **Phase 2 — DONE**: per-lane `<canvas>` overlay; MediaPipe Pose skeleton drawn on each frame (key joints highlighted), driven by `requestVideoFrameCallback`; Skeleton on/off toggle + load-status indicator.
- **Phase 3 — DONE**: per-frame landmark buffering; detects address / top / impact / follow-through; flags club past parallel, over-the-top, early extension, sway, chicken wing; flaw list with click-to-seek timestamps + red joint highlight; handedness selector. Heuristic/pose-based estimates (no club tracking).
