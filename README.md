# Solfai — launch page

A scroll-driven companion page for [Solfai](https://solfai-v2.onrender.com). Scrolling
plays a 110-frame film in which sound waves converge, break into particles, and assemble
into a glass treble clef.

## What's here

```
index.html            the whole page — markup, styles, and the scroll engine
frames/desktop/       110 WebP frames, 1920×1080
frames/mobile/        110 WebP frames, 960×540
frames/manifest.json  source metadata and extraction settings
server.js             tiny static server for local preview only
```

## Run it locally

The page must be served over HTTP — opening `index.html` directly will not load the frames.

```bash
node server.js
```

Then open <http://localhost:8080>.

## How the scroll engine works

Scroll position maps to a frame index through a **dwell remap**: a lookup table
redistributes progress so the sequence slows near each chapter centre and runs at normal
pace between them. Chapter centres and dwell centres are the same six values, so the film
settles exactly where there is something to read.

Frames load in two passes — the six chapter frames plus the first and last load before the
loader lifts, then the remainder streams in batches. A nearest-loaded-frame fallback means
the canvas never flashes blank while a frame is still in flight.

| Chapter | Progress | Frame | Rehearsal mark |
|---|---|---|---|
| The tool | 0.045 | 5 | A |
| Origin | 0.200 | 21 | B |
| The gap | 0.380 | 41 | C |
| What you get | 0.565 | 62 | D |
| Two ways in | 0.720 | 78 | E |
| Try it | 0.925 | 102 | F |

## Motion

The sequence is scroll-driven, so nothing ever autoplays — the viewer advances every frame
themselves. `prefers-reduced-motion` therefore does **not** remove the film. It drops the
frame easing and the entrance transforms instead, and a toggle in the header lets anyone
override the choice either way. The preference persists in `localStorage`.

## Design

The interface borrows the language of a marked-up choral score: rehearsal letters, measure
numbers, movable-do syllables on the progress rail, and staff hairlines that continue the
ones visible in the footage. The palette is sampled from the film — void brown `#0A0705`,
amber `#C98A44`, brass `#E6C083`, score paper `#F2EDE4` — with one functional accent used
only for the trouble-spot feature.

Type is Archivo (variable, width axis animates with scroll) and IBM Plex Mono.

## Regenerating frames

Frames come from a graded master. To rebuild at a different count or size:

```bash
ffmpeg -i glass-clef-hero-graded.mp4 \
  -vf "fps=10.9084,scale=1920:1080:flags=lanczos" \
  -c:v libwebp -quality 68 -compression_level 6 -an -frames:v 110 \
  frames/desktop/frame-%04d.webp
```

`fps` is `FRAME_COUNT / duration`. If you change the count, update `FRAME_COUNT` in
`index.html` to match.
