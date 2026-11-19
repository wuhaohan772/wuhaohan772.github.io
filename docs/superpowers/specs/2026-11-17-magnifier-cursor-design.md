# Magnifier cursor — design (approved 2026-11-17)

## Intent
Replace the 14px outline-dot cursor with a large, physically accurate magnifying glass. "Physically accurate" means what you see *through* the lens: real magnification and real lens distortion of the live page.

## Goals
- Lens ~140px diameter, handle angled down-right; hotspot stays at lens center.
- Lens shows a live, crisp copy of the page at ~2x central magnification.
- Distortion from a ray-traced thin plano-convex lens held above the page: steady center, rim stretch, faint chromatic fringe sized for crown glass (Abbe ~59).
- Glass reads as a real object: dark bezel, Fresnel-bright rim, specular highlight, soft cast shadow.
- Hover over links/buttons: bezel + handle fill solid (replaces the dot's fill state).

## Non-goals / anti-goals
- No WebGL / page snapshot (blurry text, re-capture on scroll/toggles).
- No new dependencies.
- Touch / coarse-pointer visitors keep the native cursor; no change for them.
- The clone must never be clickable, focusable, or announced by screen readers.

## Approaches considered
1. **Live DOM clone + SVG displacement filter** (picked) — crisp text, dark mode free via CSS vars, cheap per frame; needs re-clone on content changes.
2. WebGL lens shader over a rasterized snapshot — most freedom for optics, but blurry text and costly re-capture.
3. Uniform CSS scale only — cheap, but not physically accurate.

## Decided how
- Displacement map computed once on load (canvas → data URL → `feImage` → `feDisplacementMap`), per-channel scales for dispersion, recombined.
- Clone of `.callig-band` + `.split`, ids stripped, `aria-hidden`, `inert`; positioned from cursor + scroll.
- Re-clone on language toggle and email-button text change (MutationObserver on the source).
- Only `src/pages/index.astro` changes.

## Left to builder
Exact optic constants (focal length, lens height), bezel styling, handle geometry.
