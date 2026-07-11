# Context

Glossary for this site's design language.

- **Editorial index** — the site's core layout idiom: work presented as chronological list rows with metadata columns (number, year, title), not card grids. Drawn from the reference sites.
- **Landing card** — the first viewport: a large bordered box with name and basic info centered inside; the border itself is the focal device.
- **Top bar** — contacts and downloads only (Email, GitHub, LinkedIn, Resume); section navigation does not live here.
- **Side rail** — fixed left dot-plus-label menu; appears only after the landing card scrolls away; the filled accent dot marks the active section (scroll-spy).
- **Reversible reveal** — sections fade up when scrolled into view and fade back out when they leave, in both scroll directions (l-i-l.de-inspired).
- **Project row** — one entry in the Projects index: year, title, one-to-two-sentence description, tech list, external links.
- **Entry** — one item in Experience or Education: period, org, role/degree, one-line note.
- **Design tokens** — CSS variables in `src/styles/global.css`; the only place colors, type scale, and spacing are defined. Generated from `design-system/haohan-wu-personal-site/MASTER.md` ("Exaggerated Minimalism").
- **Placeholder content** — all text marked PLACEHOLDER in `src/data/*.json` awaits real content; structure is final, words are not.
