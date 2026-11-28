# Context

Glossary for this site's design language.

- **Split layout** — the site's core structure (from prototype variant B): identity fixed left, content scrolls right. Projects are numbered entries (big solid numeral left of a large title), not boxes.
- **Identity column** — sticky full-height left pane (38vw): role line, name with inline 吴皓汉, tagline, bio, contacts; ticker strip along its bottom edge. Never leaves the screen.
- **The line** — the 2px divider between identity column and content pane; the site's structural axis.
- **Line menu** — dot-plus-label section menu that straddles the line at vertical center; filled accent dot marks the active section (scroll-spy).
- **Reversible reveal** — sections fade up when scrolled into view and fade back out when they leave, in both scroll directions (l-i-l.de-inspired).
- **Project row** — one entry in the Projects index: year, title, one-to-two-sentence description, tech list, external links.
- **Entry** — one item in Experience or Education: period, org, role/degree, one-line note.
- **Design tokens** — CSS variables in `src/styles/global.css`; the only place colors, type scale, and spacing are defined. Generated from `design-system/haohan-wu-personal-site/MASTER.md` ("Exaggerated Minimalism").
- **Placeholder content** — all text marked PLACEHOLDER in `src/data/*.json` awaits real content; structure is final, words are not.
- **ASCII view** — third toolbar button (`>_`) switches it on and off. Only the title changes: "Haohan Wu" is carved out of a fine, moving field of ASCII glyphs. The field leaves glyphs out where letters fall (cell coverage sampled from a canvas redraw of the title) and fades them at the edges, while the real `<h1>` stays in place, unpainted. The field drifts and flickers about 14 frames a second, only while the title is on screen, and holds still under reduced motion. The rest of the page keeps its normal look but is set in DM Mono, the same font as the glyphs. Saved in `localStorage.ascii`.
