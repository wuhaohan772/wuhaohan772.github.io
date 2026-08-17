# Raw Mode Toggle Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a click-to-switch, all-monospace "raw" view of the site's content, built for agent copy-paste and grep, alongside the existing styled split layout.

**Architecture:** Two sibling views coexist in the DOM. A new `RawView.astro` component renders a single `<pre>` block of pipe-delimited plain text sourced from the same JSON data files the styled components already use. A new `RawToggle.astro` component toggles a `raw-mode` class on `<html>`, which CSS uses to show/hide the two views. State persists via `localStorage`, read by a blocking inline script in `Base.astro`'s `<head>` to avoid a flash of the wrong view.

**Tech Stack:** Astro (`.astro` components), vanilla JS (inline `<script>`), plain CSS (existing design tokens in `src/styles/global.css`), no new dependencies.

## Global Constraints

- Raw view uses `var(--font-mono)` exclusively — no other font family.
- Raw view uses only existing color tokens (`--color-fg`, `--color-bg`) — no new colors, no accent color.
- No new npm dependencies.
- No changes to styled-view content, layout, or animations.
- Toggle state key: `localStorage.rawMode` — presence (`"1"`) means raw mode is active; absence means styled mode.
- Field order within each raw-view data row is fixed (see Task 1) — later tasks/consumers must not reorder columns.

---

### Task 1: RawView component

**Files:**
- Create: `src/components/RawView.astro`

**Interfaces:**
- Consumes: `src/data/site.json`, `src/data/projects.json`, `src/data/experience.json`, `src/data/education.json` (existing, read-only, same shape already used by `Identity.astro`, `Projects.astro`, `Experience.astro`).
- Produces: a `<pre class="raw-view">` element in the DOM, hidden by default (`display: none` — added in Task 2's CSS, not this task). No script, no exports — later tasks only need to know the class name `.raw-view` and that it renders unconditionally (no visibility logic lives inside this component).

- [ ] **Step 1: Write the component**

```astro
---
import site from "../data/site.json";
import projects from "../data/projects.json";
import experience from "../data/experience.json";
import education from "../data/education.json";

const contacts = [
  `EMAIL: ${site.email}`,
  `GITHUB: ${site.github}`,
  `LINKEDIN: ${site.linkedin}`,
  `RESUME: ${site.resume}`,
];
---

<pre class="raw-view">NAME: {site.name} ({site.nameParts.map((p) => p.cjk).join("")})
ROLE: {site.role}
LOCATION: {site.location}
BIO: {site.bio}
{contacts.join("\n")}

PROJECTS
{projects.map((p) => `${p.year} | ${p.title} | ${p.tech.join(", ")} | ${p.description} | ${p.links.map((l) => l.url).join(", ")}`).join("\n")}

EXPERIENCE
{experience.map((e) => `${e.period} | ${e.org} | ${e.role} | ${e.notes}`).join("\n")}

EDUCATION
{education.map((ed) => `${ed.period} | ${ed.org} | ${ed.degree} | ${ed.notes}`).join("\n")}</pre>

<style>
  .raw-view {
    display: none;
    font-family: var(--font-mono);
    font-size: var(--text-sm);
    line-height: 1.6;
    color: var(--color-fg);
    background: var(--color-bg);
    white-space: pre-wrap;
    word-break: break-word;
    max-width: 80ch;
    margin-inline: auto;
    padding: clamp(1.5rem, 4vw, 3rem);
  }
</style>
```

Field order is fixed per section (do not reorder — later verification and any
future consumer relies on column position):
- Projects: `year | title | tech | description | links`
- Experience: `period | org | role | notes`
- Education: `period | org | degree | notes`

- [ ] **Step 2: Verify markup renders with real data**

Run: `cd /Users/haohanwu/code/wuhaohan772.github.io && npm run dev &` then `sleep 2 && curl -s http://localhost:4321/ | grep -A2 "PROJECTS"`

Expected: output includes a `PROJECTS` line followed by lines starting `2026 | Showdown | Go, Bubble Tea, Claude API | ...`. Kill the dev server after (`kill %1` or `pkill -f "astro dev"`).

- [ ] **Step 3: Commit**

```bash
git add src/components/RawView.astro
git commit -m "feat: add RawView component with flat-text data dump"
```

---

### Task 2: Raw-mode CSS

**Files:**
- Modify: `src/styles/global.css`

**Interfaces:**
- Consumes: class names `.split` (from `src/pages/index.astro`), `.linenav` (from `src/components/LineNav.astro`), `.raw-view` (from Task 1), and the `raw-mode` class this task defines as living on `<html>`.
- Produces: the visibility contract every later task relies on — `html.raw-mode` shows `.raw-view` and hides `.split` + `.linenav`; absence of `raw-mode` is the default (styled) state, which requires no override since `.split`/`.linenav` already default to visible and `.raw-view` already defaults to `display: none` (Task 1).

- [ ] **Step 1: Add the toggle rules**

Append to `src/styles/global.css`:

```css
/* raw mode: flat mono text view, toggled by RawToggle.astro */
html.raw-mode .split {
  display: none;
}

html.raw-mode .linenav {
  display: none;
}

html.raw-mode .raw-view {
  display: block;
}
```

- [ ] **Step 2: Verify the rule is present**

Run: `grep -A3 "html.raw-mode .split" /Users/haohanwu/code/wuhaohan772.github.io/src/styles/global.css`

Expected: shows the three-rule block just added.

- [ ] **Step 3: Commit**

```bash
git add src/styles/global.css
git commit -m "feat: add raw-mode visibility rules"
```

---

### Task 3: RawToggle component + FOUC-avoidance init script

**Files:**
- Create: `src/components/RawToggle.astro`
- Modify: `src/layouts/Base.astro`

**Interfaces:**
- Consumes: `html.raw-mode` class contract from Task 2; `localStorage.rawMode` key (this task defines and owns it — no other file reads/writes it).
- Produces: a `<button class="raw-toggle">` element, fixed top-right, that on click toggles `raw-mode` on `document.documentElement`, flips its own label between `[RAW]` and `[BACK]`, and syncs `localStorage.rawMode`. `Base.astro` gains a blocking inline script (in `<head>`, before `<slot />` renders) that applies `raw-mode` on initial load if `localStorage.rawMode === "1"`, and sets the button's initial label to match.

- [ ] **Step 1: Add the blocking init script to Base.astro**

In `src/layouts/Base.astro`, add this script inside `<head>`, immediately after the existing `<title>{title}</title>` line (must run before body paint, so it stays a plain synchronous inline script, not `type="module"` and not deferred):

```html
    <script is:inline>
      if (localStorage.getItem("rawMode") === "1") {
        document.documentElement.classList.add("raw-mode");
      }
    </script>
```

- [ ] **Step 2: Write RawToggle.astro**

```astro
<button class="raw-toggle" id="raw-toggle" type="button">[RAW]</button>

<script is:inline>
  {
    const btn = document.getElementById("raw-toggle");
    const sync = () => {
      const active = document.documentElement.classList.contains("raw-mode");
      btn.textContent = active ? "[BACK]" : "[RAW]";
    };
    sync();
    btn.addEventListener("click", () => {
      const active = document.documentElement.classList.toggle("raw-mode");
      if (active) {
        localStorage.setItem("rawMode", "1");
      } else {
        localStorage.removeItem("rawMode");
      }
      sync();
    });
  }
</script>

<style>
  .raw-toggle {
    position: fixed;
    top: var(--space-2);
    right: var(--space-2);
    z-index: 30;
    font-family: var(--font-mono);
    font-size: var(--text-sm);
    color: var(--color-fg);
    background: var(--color-bg);
    border: 1px solid var(--color-fg);
    padding: 0.4em 0.8em;
    cursor: pointer;
  }

  .raw-toggle:hover {
    background: var(--color-fg);
    color: var(--color-bg);
  }
</style>
```

Note: `is:inline` on both scripts is required — Astro bundles/module-wraps
plain `<script>` tags by default, which would defer execution past first
paint (defeating the FOUC-avoidance purpose of Step 1) and would sandbox the
two scripts from sharing the synchronous `document.documentElement` state
they both depend on.

- [ ] **Step 3: Commit**

```bash
git add src/components/RawToggle.astro src/layouts/Base.astro
git commit -m "feat: add RawToggle button with localStorage persistence"
```

---

### Task 4: Wire RawView and RawToggle into the page

**Files:**
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `RawView` (Task 1, default export, no props), `RawToggle` (Task 3, default export, no props).
- Produces: final page composition — no further tasks depend on this file.

- [ ] **Step 1: Import and render the new components**

In `src/pages/index.astro`, update the frontmatter imports and template:

```astro
---
import Base from "../layouts/Base.astro";
import Identity from "../components/Identity.astro";
import LineNav from "../components/LineNav.astro";
import Projects from "../components/Projects.astro";
import Experience from "../components/Experience.astro";
import Footer from "../components/Footer.astro";
import RawView from "../components/RawView.astro";
import RawToggle from "../components/RawToggle.astro";
---

<Base>
  <div class="split">
    <Identity />
    <main>
      <Projects />
      <Experience />
      <Footer />
    </main>
  </div>
  <LineNav />
  <RawView />
  <RawToggle />
</Base>
```

(Only the frontmatter imports and the two new lines before `</Base>` change; the existing `<style>` block in this file is untouched.)

- [ ] **Step 2: Verify the build succeeds**

Run: `cd /Users/haohanwu/code/wuhaohan772.github.io && npm run build`

Expected: build completes with no errors, `dist/index.html` contains both `class="split"` and `class="raw-view"`.

- [ ] **Step 3: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: wire RawView and RawToggle into the homepage"
```

---

### Task 5: Manual verification pass

**Files:** none (verification only, no code changes expected — if a check fails, fix in the relevant task's file and re-run this task's checks)

**Interfaces:**
- Consumes: the fully wired page from Task 4.
- Produces: nothing (terminal task).

- [ ] **Step 1: Start the dev server**

Run: `cd /Users/haohanwu/code/wuhaohan772.github.io && npm run dev`

- [ ] **Step 2: Manual browser checks**

Open `http://localhost:4321/` in a browser and confirm, in order:

1. Fresh load (clear `localStorage` first: devtools console `localStorage.clear()`, then reload) — styled split view renders, `[RAW]` button visible top-right.
2. Click `[RAW]` — styled view and line-nav disappear, flat mono `<pre>` view appears with all four sections (NAME/ROLE/.../PROJECTS/EXPERIENCE/EDUCATION populated), button now reads `[BACK]`.
3. Select-all in the raw view, copy, paste into a plain text editor — confirm clean plain text: no HTML, no duplicated ticker text, no stray CJK-only lines, matches the field order from Task 1.
4. Reload the page — raw mode persists immediately (no flash of the styled view first).
5. Click `[BACK]` — styled view returns; reload again — stays styled (localStorage cleared).
6. Resize the browser to a narrow (mobile) width in both modes — no horizontal scrollbar, no visual overlap/break.
7. Toggle OS light/dark theme (if available) while in raw mode — text stays legible (site currently has one light palette only, so this check confirms nothing is hardcoded to break under `prefers-color-scheme: dark`, not that a dark palette exists).

- [ ] **Step 3: Stop the dev server**

Run: `pkill -f "astro dev"` (or Ctrl-C in the terminal running it).

---

## Self-Review Notes

- **Spec coverage:** toggle mechanism (Task 3), everything-flattened content (Task 1), button placement/label (Task 3), persistence (Task 3), styling constraints — mono-only font, existing tokens only (Task 1 + Task 3 styles), layout hookup (Task 4), verification checklist (Task 5, mirrors spec's 7-point list exactly). No gaps found.
- **Placeholder scan:** none — every step has literal code/commands.
- **Type/name consistency:** class `raw-mode` (Task 2 CSS, Task 3 JS, this doc) and `.raw-view` (Task 1 markup, Task 2 CSS) match everywhere; `localStorage.rawMode` key name and `"1"` sentinel value consistent across Task 3 (write) and Task 3 Step 1 (read) — no other task touches it.
