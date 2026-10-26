# OKLCH Lab: session history

A condensed record of the Claude Code session (2026-10-26, started in `D:\`) that produced this folder. A new session can use it to pick up the work.

## Where things are

| What | Location |
|---|---|
| Current page | `oklch-lab.html` |
| Published artifact (private) | https://claude.ai/artifact/VpDvgrH4SLBV8Ts83MNhPZ. Republish updates from `oklch-lab.html` to this URL. Deep links: `#sliders`, `#knob`, `#vector`. |
| Older standalone knob artifact | https://claude.ai/artifact/JbCrhYPYvpt6RJkC2KKFo8 (frozen; same as `history/3-oklch-knob.html`) |
| Saved iterations | `history/` (numbered in the order they were made, see below) |

## Working agreements (from the user)

- **Keep every iteration until told otherwise.** Before a change that replaces a version, save a copy into `history/`. New UI ideas go in as new numbered tabs on the lab page, not as replacements.
- **Keep versions on one page.** All versions live as tabs in `oklch-lab.html` and share one color state, so switching tabs compares them directly.
- When the user tweaks an existing version (size, layout), edit that tab in place. Earlier forms are kept in `history/` and in the artifact's own version history.
- Chroma is **linear** by default. A future "mode" concept (e.g. a Background mode) may bring back a scale that favors low chroma. The code already has a `SCALES` table for this.

## How the project evolved

1. **Hex word swatches.** Names and words spelled only with a–f (e.g. `#ace`, `#decade`, `#facade`, King Dedede, Dee Dee) shown as color blocks. The page added a palette strip with drag-and-drop, shuffle, meaning tooltips and a light/dark toggle. → `history/1-hex-word-swatches.html`
2. **Background tuner.** The swatches were dropped. The light/dark toggle became HSL sliders (hue, saturation, lightness) that set the page background. Saturation had an exponential scale for fine low-end control. Slider bars show live colors and react to the other two sliders.
3. **HSL vs OKLCH.** The two models were compared side by side. HSL lightness is not perceptual, and its brightness shifts with hue. HSL was dropped, and OKLCH (hue, chroma, lightness) kept. → `history/2-oklch-sliders.html`
4. **Knob.** Hue became a ring with chroma (left) and lightness (right) half-circle arcs around it. The arcs increase bottom to top. Later it got thinner bands, rounded arc ends and a 300px max size, and its values moved into the center disc. → first form in `history/3-oklch-knob.html`
5. **Lab page.** Versions were combined as tabs (1 Sliders, 2 Knob) sharing one color.
6. **Vector (tab 3).** A point on a plane around an origin: angle = hue (0° at top, clockwise), distance = chroma. The plane shows the color field at the current lightness, with a dashed sRGB gamut edge. Three lightness ideas were added together: wheel/pinch on the plane (up or spread = lighter, about 2% per wheel notch; trackpad pinch arrives as ctrl+wheel), a vertical L slider, and a side view (a slice at the current hue, chroma across and lightness up, draggable). → before lightness: `history/4-lab-vector-hue-chroma-only.html`
7. **Linear chroma.** The exponential chroma scale was replaced by a linear 0 to 0.37 scale. Saved positions are migrated, so the color stays the same.
8. **Absolute / Relative chroma.** The user asked why `oklch(92.9% 0.204 111)` yellow looked so much more saturated than other hues. The sRGB gamut is lopsided: at L 92.9%, max chroma is 0.204 for yellow but about 0.034 for blue and about 0.039 for red. Yellow peaks near L 96%, blue near L 45%. Absolute chroma gets clipped for most hues, while yellow keeps it all. Fix: a global **Chroma: Absolute | Relative** switch. Relative = percentage of the maximum chroma that hue can reach on screen at the current lightness (an OKHSL-like idea), so every hue feels equally intense. Switching converts the position so the on-screen color stays the same. → before this: `history/5-lab-absolute-chroma-only.html`
9. **Bug fixed:** the mode buttons share the `.tabs` styling and were being picked up as version tabs, which reset the view to Sliders. Version tabs are now selected by `[role="tab"]`.
10. **Bug fixed (session 2):** all three panels showed on every tab when the file was opened directly, because `.panel { display: flex }` overrode `hidden`. A global `[hidden] { display: none !important; }` rule fixes it. (The artifact wrapper already had that rule.)
11. **Absolute / Relative lightness.** The user felt the colorful part of the lightness range went by too fast. Measured: averaged over hues, the most chroma a screen can show is at L ≈ 64%, but each hue peaks at a different lightness (blue ~50%, red ~63%, yellow ~97%). First idea was "equal visible change per step" (arc length along the gamut edge in L/C). Tested, it mostly stretches the slopes beside the peak, not the peak, and barely changes red or blue, so it was dropped. Built instead: slider travel at each L is weighted by `exp(strength × share × maxChroma(L,H)/peak(H))` (first built as linear `1 + strength × …`, but that flattened out past about 8; the exponential keeps narrowing toward the peak). The colorful band of the current hue gets more room. It's flat for grays and stronger the more chroma is picked (user's choice: the curve follows current chroma). `share` is chroma as a fraction of the hue's peak (relative p/1000, or absolute C/peak), so it doesn't change during a lightness drag. Strength 0–10, default 2, is a taste knob shown only in Relative. Applies to the Sliders L bar, Knob lightness arc, Vector L slider and wheel/pinch. The Vector side view stays linear (it's a map). → before this: `history/6-lab-linear-lightness-only.html`
12. **Vector tab trimmed.** At the user's request the side view (lightness/chroma slice) was removed, and an **L slider: Show | Hide** switch was added inside the Vector panel (`ok-vec-lslider`). With it hidden, scroll/pinch on the plane is the only lightness control there. → before this: `history/7-lab-vector-with-side-view.html`
13. **Cursor hidden while dragging.** The Knob dial and Vector plane get a `dragging` class (`cursor: none`) from pointerdown until release, and range inputs hide the cursor while `:active`.
14. **Modifier locks on the Vector plane.** While dragging the point, Ctrl changes only hue and Alt only chroma (checked per move, so they work mid-drag). Tried and dropped: "follow the mouse but keep one value" first, which jumped on 1D movement; then relative movement (vertical, later up/right for chroma), which let the hidden cursor drift far from the point, and browsers can't move the cursor back. Settled on: both locks follow the pointer, so the cursor stays near the point. Ctrl = hue from the pointer's angle (chroma kept); Alt = chroma from the pointer's projection onto the current hue line (hue kept). No offsets. Lightness controls are unchanged. The Knob needs no locks, since each of its controls sets one value.
15. **Grid and markers.** The Vector panel got a **Grid: Show | Hide** switch (`ok-vec-grid`) for the axes, chroma rings and labels; the dashed gamut edge always shows. The sliders (Sliders tab and Vector L slider) and the Knob got always-on tick markers: hue every 30° (0/90/180/270 strong), chroma at the plane's ring values (`chromaGrid()`), and lightness every 10% (50% strong), placed through `lightToT` so they show the Relative lightness curve. Slider ticks are a CSS gradient layer inset by the thumb radius (11px); knob ticks are dark-under-light radial lines.
16. **One global Grid switch.** The user wanted the show/hide option for every control, so the Vector-only switch became a header **Grid: Show | Hide** switch (`ok-grid`, which falls back to the old `ok-vec-grid`). It hides slider ticks, knob ticks, and on the Vector plane the axes, rings, ring labels, 0/90/180/270° labels, and the origin-to-point line with its origin dot. The point, plane edge and gamut edge always show.
17. **Background switch.** **Background: Color | White | Black** (`ok-bg`), placed top-right on the title row, apart from the color controls (user request). White and Black theme the page as a gray at L 1 or 0, which `applyPage` handles by overriding its inputs, so surfaces, borders and text stay readable. Canvas ink is still chosen from the color's own lightness.
18. **Lightness on the handles.** `drawDot()` draws every round handle (the three on the Knob tab and the Vector point) at a size set by the header **Handle Size** slider (outer diameter 24–64px, default 24, `ok-handle`; Knob handles no longer scale with the dial). A black band (80% opacity) grows inward from the edge, from 0 at L 100% to the whole handle at L 0%, with a white ring just inside it that thins with lightness (2px at L 100%, 0 at L 0%, so the color stays visible longer) and the color in the middle. (Earlier tries: the white ring thickening, then a dark outer ring from 1.5 to 6px. The user asked for this more extreme version.)

## How `oklch-lab.html` works

- **Shared state:** `{ h: hue°, p: chroma position 0..1000, l: lightness % }`, plus `chromaMode` ("absolute" or "relative"). Every control writes through `setState()`, and `render()` updates all tabs plus the page colors.
- **Chroma:** `chromaAt(p, L, H)`. Absolute is `C_MAX * p/1000` (`C_MAX = 0.37`, via `SCALES.linear`). Relative is `maxChroma(L, H) * p/1000`. `maxChroma` binary-searches the sRGB gamut edge.
- **Color math:** OKLCH to linear sRGB uses Björn Ottosson's OKLab matrices. Out-of-gamut colors are mapped by lowering chroma, keeping L and H (`oklchToRgb` returns `{rgb, c, clipped}`). `rgbToOklch` is not in the current file; it existed in the HSL-vs-OKLCH version if needed.
- **Page theming:** the background is the chosen color. Surface and border are offset in OKLCH lightness. Text flips light/dark at L < 0.6.
- **Drawing:** the knob and vector are `<canvas>` drawings. Each has visually hidden `<input type="range">` elements for keyboard use, and draws a focus ring when one is focused. The vector plane and side view cache their color fields: the plane is rebuilt when L changes, the side view when H changes, and when the mode changes (the side view was removed in step 12).
- **Storage:** `localStorage` keys `ok-h`, `ok-c`, `ok-l`, `ok-view`, `ok-chroma-mode`, `ok-scale`, `ok-light-mode`, `ok-light-strength`, `ok-vec-lslider`, `ok-grid` (older `ok-vec-grid` read as a fallback), `ok-bg`, `ok-handle`. Every access is wrapped in try/catch.
- **Not verified in a browser:** checks so far were syntax checks and node runs of the color math. Interactions (dragging, pinch, vertical slider via `writing-mode`) have not been tested by eye.

## Open ideas mentioned but not built

- **Modes** (e.g. Background) that change the chroma scale or default ranges. The `SCALES` table is the hook.
- A third chroma scale option, or per-mode lightness ranges.
- Possibly consolidating or retiring versions, but only when the user says so.
