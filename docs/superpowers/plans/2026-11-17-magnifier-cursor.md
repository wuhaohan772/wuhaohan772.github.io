# Magnifier Cursor Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the 14px cursor dot with a 140px magnifying glass that shows a live, ray-traced, 2x-magnified view of the page.

**Architecture:** A fixed `.mag` element follows the pointer. Inside it, a scale(2) clone of the page (`.callig-band` + `.split`) is warped by an SVG filter whose displacement maps (one per R/G/B wavelength) come from tracing vertical rays through a plano-convex lens at load time.

**Tech Stack:** Astro 5, vanilla TS in an inline `<script>`, SVG filters. No new deps.

**Spec:** `docs/superpowers/specs/2026-11-17-magnifier-cursor-design.md`

## Global Constraints
- Lens radius 70px (140px across); hotspot = lens center.
- Central magnification M0 = 2 (CSS scale), residual distortion via `feDisplacementMap`.
- Optics: plano-convex, convex up, sphere radius Rc = 110px, aperture radius 70px; BK7 n_R=1.5143, n_G=1.5168, n_B=1.5224; page distance h solved so paraxial green M = 2.
- Displacement `scale` = 12 (max |D| ≈ 5px at rim); filter uses `color-interpolation-filters="sRGB"`.
- Only `src/pages/index.astro` changes. Fine pointer only; touch keeps native cursor.
- Clone: ids stripped, `aria-hidden`, `inert`.

## Review Focus
- Scrolled past the sticky band: lens over band must show the band, not content behind it.
- Language toggle / "copied" text: lens updates within a frame.
- Dark mode toggle: lens shows dark theme.
- Pointer leaves window: lens hides.
- Window resize: clone width tracks viewport.

---

### Task 1: Magnifier cursor (single file)

**Files:** Modify `src/pages/index.astro` (cursor markup ~115, script ~118-127, CSS ~413-449; `#dark-toggle .icon-*` selectors → class-only so the id-stripped clone styles right).

**Interfaces:**
- `traceLens(r: number, n: number): number` — page-plane radius sampled by the ray at lens radius r.
- `buildMap(n: number): string` — 140×140 PNG data URL, R/G = x/y displacement.
- `reclone(): void`, `place(): void` (rAF-scheduled).

- [ ] **Step 1:** Replace cursor-dot markup/CSS/JS with `.mag` (glass, view+filter, sheen, rim, handle) and the SVG `<filter id="lens-f">`.
- [ ] **Step 2:** `npm run build` — expected: completes, no TS errors.
- [ ] **Step 3:** Headless Edge check (playwright-core, channel msedge): move mouse to a heading, screenshot; scroll 600px and hover the band; toggle lang and dark. Expected: magnified, warped, synced content in all four.
- [ ] **Step 4:** Run dev server and open it for the user.
