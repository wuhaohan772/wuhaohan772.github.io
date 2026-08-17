# Raw mode toggle — design

## Purpose

Add a click-to-switch "raw" view to the personal site: a barebones, all-monospace,
brutalist rendering of the same content, intended for fast copy-paste into LLM
agents or `grep`/`Ctrl-F` scanning. Complements the existing styled split-layout
view, doesn't replace it.

## Non-goals

- No separate URL/route for the raw view (in-page toggle only, per user choice).
- No server-side rendering variant / no-JS fallback beyond avoiding flash-of-wrong-mode.
- No changes to the styled view's content, layout, or animations.

## Architecture

Two sibling views live in the DOM at all times; CSS + a `raw-mode` class on
`<html>` decide which one is visible. No client-side routing, no content
duplication logic beyond a second Astro component reading the same data files.

```
<html class="raw-mode?">
  <body>
    <div class="split">...</div>      <!-- existing styled view -->
    <RawView />                        <!-- new flat-text view -->
    <RawToggle />                      <!-- new toggle button -->
    <LineNav />                        <!-- hidden when raw-mode active -->
  </body>
</html>
```

## Components

**`src/components/RawView.astro`** (new)
Reads `site.json`, `projects.json`, `experience.json`, `education.json` directly
(same data sources as the styled components — no duplication of content, only
of presentation). Renders a single `<pre class="raw-view">` block, plain text,
line-based:

```
NAME: Haohan Wu (吴皓汉)
ROLE: Software Engineer
LOCATION: Placeholder City
BIO: Currently studying at PLACEHOLDER. Interested in PLACEHOLDER-FIELD.
EMAIL: hwu27@stanford.edu
GITHUB: https://github.com/wuhaohan772
LINKEDIN: https://www.linkedin.com/in/PLACEHOLDER
RESUME: /resume.pdf

PROJECTS
2026 | Showdown | Go, Bubble Tea, Claude API | Heads-up poker in the terminal against an LLM opponent that talks trash, remembers the match, and reacts when it loses. | https://github.com/wuhaohan772/showdown
2025 | Placeholder Project Two | TypeScript, React, PostgreSQL | One to two sentences: what it does, why it exists, what was hard about it. Concrete numbers beat adjectives. | https://github.com/wuhaohan772, https://example.com
2025 | Placeholder Project Three | Python, FastAPI | A tool/library/experiment. What problem it solves and for whom. | https://github.com/wuhaohan772
2024 | Placeholder Project Four | C++ | Older but still worth showing — coursework project, hackathon win, or research code. |

EXPERIENCE
2025 — now | Placeholder Company | Software Engineering Intern | One line on the team and what you shipped. Numbers if possible.
2024 | Placeholder Lab | Research Assistant | One line on the project and your contribution.
2023 | Placeholder Org | Teaching Assistant | Course name, what you ran.

EDUCATION
2022 — 2026 | Placeholder University | B.S. Computer Science | Relevant coursework: systems, algorithms, ML. GPA if strong.
2026 — | Placeholder University | M.S. (if applicable) | Focus area.
```

Field order per row: fixed, so a line's Nth `|`-delimited column always means the
same thing within its section (year/period, org/title, tech/role, description,
links). Links field: comma-joined URLs, empty string if none (keeps column count
constant for anyone parsing/grepping by `|`).

**`src/components/RawToggle.astro`** (new)
Fixed-position button, top-right, `[RAW]` / `[BACK]` label toggle. Inline
`<script>`:
- On click: toggle `raw-mode` class on `document.documentElement`, flip own
  label, write choice to `localStorage.rawMode` (`"1"` / removed).
- A second inline script in `Base.astro` `<head>` (or very top of `<body>`,
  before first paint) reads `localStorage.rawMode` and applies the class
  immediately — avoids a flash of the wrong view on load.

## Styling

New scoped rules (in `RawView.astro` and/or a small addition to
`global.css`):
- `.raw-view { display: none; }` — hidden by default.
- `html.raw-mode .raw-view { display: block; }`
- `html.raw-mode .split { display: none; }`
- `html.raw-mode` also hides `LineNav` (its dot-menu targets `.split` sections
  and is meaningless once that layout is gone).
- `.raw-view` text: `font-family: var(--font-mono)` only, existing `--color-fg`
  / `--color-bg` tokens (so light/dark still respected), no accent color, no
  borders/shadows beyond the toggle button's own plain outline, `white-space:
  pre-wrap` so long description lines wrap instead of overflowing, a sane
  `max-width` + margin so it's readable on wide screens without becoming a
  design element itself.
- Toggle button styled minimally: mono font, 1px solid border, no hover
  animation beyond a plain color invert — stays legible in both states.

## Data flow

Build-time only: Astro imports the same four JSON files already used by
`Identity`/`Projects`/`Experience`. `RawView.astro` performs its own `.map()`
over each array to emit the pipe-delimited lines — no new data files, no
runtime fetch.

## Persistence

`localStorage.rawMode` — presence means raw mode is active. Read once on page
load (blocking inline script, before first paint) and on toggle click. No
expiry, no cross-device sync (out of scope).

## Testing / verification

Manual, in-browser (this is a static personal site with no existing test
suite):
1. Load page fresh (no localStorage) → styled view renders, `[RAW]` button
   visible.
2. Click `[RAW]` → styled view hides, flat mono text view shows, all four
   data sections present, button now reads `[BACK]`.
3. Select-all + copy the raw view → paste into a plain text editor → confirm
   clean plain text, no stray markup/duplicated ticker text/CJK-only lines.
4. Reload page → raw mode persists (localStorage read on load, no flash of
   styled view).
5. Click `[BACK]` → styled view returns, localStorage cleared, reload stays
   styled.
6. Resize to mobile width in both modes → no horizontal scroll, no layout
   break.
7. Toggle dark/light OS theme in raw mode → text stays legible (uses existing
   color tokens).
