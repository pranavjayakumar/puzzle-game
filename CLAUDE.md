# Front Page Puzzle — project context

Game jam prototype for a **newspaper company**. Each day, that day's **front page is the puzzle**.
Players tap Start, the page breaks into blocks, and they drag the blocks back together.
**Fastest time of the day wins.** Each player gets **one play per day**.

## Files
- `index.html` — the whole game (HTML + CSS + JS in one file, no build step, no dependencies except the Oswald font from Google Fonts).
- `paper.jpg` — today's front page (1024×1536, 2:3 ratio). Loaded via `const PAPER="paper.jpg"` in the script.
- `reference/` — screenshots of the reference game whose look we are matching. **Look at these before any UI change.**

Run: open `index.html` in a browser, or `npx serve .` / `python3 -m http.server`. Test in mobile view (e.g. 390×844).

## Original brief (from the team)
> You land on the game page and there is a start button and a help button for new users to understand the game. Once you click start the timer sets off immediately and every second wasted will put you behind thousands of people who are decoding the puzzle on the same day. The help button will have instructions on how to play the game in 1 or 2 sentences. The puzzle pieces for the day fall down on the screen with micro animations and sounds. When the puzzle is made and the game finishes, it shows the leaderboard with time taken by other players who did that puzzle that day and who reached first.

## Non-negotiable direction
- **Match the reference UI closely** (see `reference/`): flat warm beige background with soft vignette, grid of rounded "empty" cells slightly darker than the background, centred label + cream pill timer at top, square cream button top-right, chunky polyomino blocks with a dark-brown bottom edge (3D thickness) and soft shadow, bold condensed ALL-CAPS type in dark taupe, "WELL DONE!" + dark 3D arrow button on win.
- **Mobile-first.** Single column, max-width 480px, centered on desktop.
- **Very simple UI and UX.** It should be obvious what the game is from the first screen. No nav bars, no tabs, no stats panels, no scoring rules, no long how-to. A previous prototype was rejected for exactly this — it was text-heavy, had 5 nav tabs, a masthead, a practice mode, an archive page, and text-card pieces dropped into labelled slots. **Don't drift back toward that.**
- **Micro-interactions and sound everywhere**: button presses, picking up, snapping, correct joins, break-apart, win.
- It's a prototype — feel and polish matter more than completeness.

## Design tokens (in `:root`)
`--bg #E6DDC8` · `--cell #D9CEB6` · `--ink #5A5146` · `--ink-soft #9A8F7E` · `--cream #F6F0E0` / `--cream-edge #D3C7AC` · `--dark #5E5346` / `--dark-edge #3A3028` · `--piece-edge #3E3329`. Font: Oswald 500–700.
3D buttons = solid offset `box-shadow` edge; pressed state moves down and collapses the edge.

## Current flow (implemented, tested at 390×844, 360×640, desktop)
1. **Landing**: date label ("14 Oct") above timer pill "0:00", "?" button top-right. Whole front page sits on a 6×8 grid (gently floating). Text "TODAY'S FRONT PAGE IS YOUR PUZZLE", dark START button, "12,481 READERS SOLVING TODAY" (fake, ticks up).
2. **Start**: timer (count-up stopwatch) starts instantly → board shakes + paper-tear sound → page splits into 7 pieces that pop up, rotate and fall onto random non-overlapping grid positions with staggered landing thuds. Hint becomes "DRAG THE BLOCKS TO REBUILD THE PAGE".
3. **Play**: pointer-event drag (mouse + touch). Lift = scale + bigger shadow + pick sound. Drop snaps to nearest cell if it fits (in bounds, no overlap) → wooden click; if it forms a correct join with a neighbouring piece → chime + sparkles + bounce on both pieces. Invalid drop → slides back with a bump. `navigator.vibrate` haptics on Android.
4. **Win** = every piece has the same offset from its home position (the picture can be assembled anywhere on the grid, like the reference). Shine sweep → pieces replaced by one clean image → "WELL DONE!" overlay with time (m:ss.t) and arrow button.
5. **Leaderboard**: dark "Your time / Your rank" card, "Fastest today of 12,481" list (top 8, or top 5 + "···" + your neighbourhood if rank > 8), your row in dark brown, #1 gets a crown. "Next front page in hh:mm:ss" countdown, "View today's paper" (full-screen image viewer), small "Reset (prototype only)" link.
6. **Once per day**: result saved in `localStorage` under `fpp-YYYY-MM-DD`. Returning shows "YOU'VE SOLVED TODAY'S PAPER", RESULTS button and your time.
7. **Help** ("?"): bottom sheet, tiny looping drag animation, one sentence, GOT IT.

## Code map (`index.html` script)
- `LAYOUT` — 4×6 letter grid defining the 7 pieces (A–G) over the picture. Edit this to change piece shapes/count. Board is `BW×BH = 6×8`, picture `PW×PH = 4×6`, home offset `HOME={x:1,y:1}`.
- `measure()` — computes integer cell pitch `P` from the stage size (36–74px), gap `G`, inset `PI`.
- `buildPiece(p)` — each piece = container with per-cell divs showing a slice of `paper.jpg` via background-position; outer corners rounded; thickness via CSS `filter: drop-shadow`.
- `scatter()` — random non-overlapping placement, rejects layouts with >1 accidental correct join.
- `attachDrag(p)` — pointer drag, snapping, join detection (`offset()`, `adjacentHome()`), win check (`solved()`).
- `sfx` — all sounds synthesised with Web Audio (tone/noise helpers). No audio files.
- `start()`, `win()`, `showResults()` — flow. `rankFor()` / `board10()` — **fake** leaderboard data.

## Known limitations / open items
- Leaderboard and player counts are fake; no backend, no accounts, one-play-per-day is per-browser only. Real version needs: server-side start/finish timestamps (don't trust client timer), daily puzzle config, leaderboard API, player identity (name currently just "You").
- Newspaper text is small on phones; headlines and photos are what make pieces recognisable.
- Difficulty is fixed: 7 pieces. Could vary by day / grow the grid.
- No mute toggle (relies on device volume).
- The sample `paper.jpg` carries a real newspaper's masthead and quotes attributed to real politicians — treat it as internal placeholder content; confirm rights before any public hosting.

## Ideas discussed but not built
Player name entry after finishing, share-your-time card, streaks, haptics on iOS, sound toggle.
