# Archaic Java design system

Use this system for Archaic Java websites, web applications, and documentation interfaces. Preserve an existing product's explicit design contract unless a redesign is requested. The system defines a visual identity, not a frontend architecture.

## Direction

Draw from technical manuals of the 1950s and 1960s and the typographic discipline of the [1976 NASA Graphics Standards Manual](https://www.nasa.gov/wp-content/uploads/2015/01/nasa_graphics_manual_nhb_1430-2_jan_1976.pdf), especially sections 5.3–5.6. The reference is historical context, not a required runtime dependency or an instruction to reproduce NASA branding.

Visible structure, economical means, and careful typography give the system its character. Use clean paper-like surfaces, aligned information, restrained printing colours, and horizontal rules. Avoid distressed page textures, decorative shadows, gradients, rounded card collections, and decorative animation. Print texture may belong to an illustration.

Documentation can be spacious and editorial; working applications can be compact. Preserve shared type roles, colours, rules, controls, and alignment across both. Do not copy the specimen's large introduction into a screen whose purpose is to perform work.

## Typography

**Sans serif establishes structure; serif explains; monospace specifies.** Use the browser's generic `sans-serif`, `serif`, and `monospace` families directly. Do not download fonts or substitute named typefaces or `system-ui`; the user's browser defaults are intentional.

| Role | Family and treatment |
| --- | --- |
| Page titles and headings | Sans serif, moderate weight (600 in the specimen), sentence case |
| Wordmark | Sans serif, moderately bold, restrained negative tracking; “archaic java” |
| Navigation, controls, labels, table data | Sans serif; clear and economical |
| Explanatory prose, documentation, introductions | Serif; comfortable line length and line height |
| Code, commands, paths, identifiers | Monospace |
| Section numbers and component letters | Upright sans serif; optional blue |
| Context labels and figure captions | Small sans serif; capitals reserved for brief context labels |

Use size, weight, whitespace, and placement to establish hierarchy. Avoid relying on very light weights or tightly tracked letters. Start with 1–1.125rem body text and approximately 1.5–1.6 line height; keep operational text comfortably readable. The accepted specimen uses 1.125rem prose, 0.875rem controls, and 1.15 heading line height. Large titles have modest tracking (−0.025em).

Allow text to wrap and controls to grow with browser-default font metrics. Use relative units and test text enlargement; identical pixels across operating systems are not the goal.

## Ink and paper

| Token | Value | Purpose |
| --- | --- | --- |
| `--paper` | `#F7F5EE` | Warm, clean background |
| `--ink` | `#242922` | Main text and strong rules |
| `--muted` | `#5D6259` | Secondary text |
| `--accent` | `#285B78` | Links, primary actions, focus, selected emphasis |
| `--rule` | `#B8BBB0` | Subtle separators |
| `--tint` | `#EDF0EB` | Code headings and notes |
| `--error` | `#963C2E` | Error emphasis |
| `--success` | `#376143` | Success emphasis |

Keep blue scarce enough to identify interaction. Status colours supplement explicit text; colour alone must not carry meaning. Thin separators are not sufficient boundaries for interactive controls: use the darker control borders in the foundation.

The accepted system is a light theme. Do not infer a dark palette by inversion; if dark mode is requested, design and check it as an extension.

## Composition and components

- Use a consistent title block: short context label when useful, sans-serif heading, optional serif explanation, and a rule.
- Align related information; keep related controls close. Separate major regions through whitespace and heavier rules (3px in the specimen), with thin rules (1px) within regions.
- Prefer tables and lists for operational data. Choose density around the task, not an arbitrary card grid.
- Use square-cornered, visibly bordered controls, explicit labels, and underlined text links. Primary actions use blue with paper-coloured text. Provide clear focus, hover, disabled, validation, and completion states.
- Keep forms and notices concise. The specimen demonstrates validation without replacing the user's input, reset behavior, and status announcements.
- Keep motion limited to useful feedback and respect reduced-motion preferences. Do not animate Riot merely to decorate the page.
- Use simple, precise diagrams with numbered figures and HTML captions when they explain something. Number sections only when the numbering aids orientation.

## Riot

**riot** is the character's name; **Riot** is its display capitalization. He is the punk cousin of the original Java mascot, Duke: curious, independent-minded, and a practitioner who examines, explains, and occasionally encounters a problem.

The supplied [identity reference](../assets/design-system/riot-reference.jpeg) is authoritative for his appearance. Preserve his recognizable pointed silhouette, face construction, proportions, and comic character. The [transparent specimen asset](../assets/design-system/riot.png) is a prepared variant of that reference.

**Posture, texture, and facial expression may be altered wherever relevant.** Choose variants to support the meaning of the page. Punk describes his character; do not automatically add stereotypical accessories or replace his established appearance.

Use Riot sparingly beside introductions, selected technical notes, tutorials, or appropriate empty states. Leave everyday controls and dense working areas quiet. The accepted specimen places him once beside “The working principle,” in a thoughtful pose. That pose fits an explanation or “Before you begin” note, rather than a success message or urgent error.

Keep captions outside the artwork as actual text. Provide descriptive alternative text when the illustration carries meaning, or empty alternative text when it is purely decorative. Never make the illustration the only carrier of an instruction or state. Prefer transparent backgrounds and ink/paper tones. Simplify grain and detail for small uses; the full textured figure is not a favicon.

## Bundled implementation

All paths below are relative to the skill root:

| Resource | Use |
| --- | --- |
| `assets/design-system/foundation.css` | Shared tokens, generic typography, links, controls, and focus; a small reusable starting point |
| `assets/design-system/index.html` | Accepted specimen: introduction, repository table, build note, forms, and states |
| `assets/design-system/specimen.css` | Composition and component examples; loaded after the foundation |
| `assets/design-system/specimen.js` | Local example interactions; no backend or framework |
| `assets/design-system/riot-reference.jpeg` | User-supplied identity reference |
| `assets/design-system/riot.png` | Transparent thoughtful pose used in the specimen |

Open `index.html` locally to inspect the visual reference. Filtering, repository details, and form feedback run locally; copying commands depends on browser clipboard permissions and has a manual fallback. Repository entries are illustrative, not live integrations.

Copy or adapt the foundation and only the relevant specimen components. The foundation uses global element selectors: scope or translate them when integrating into an existing interface such as Forgejo. Keep asset references local and portable. Do not copy demonstration content, deployment metadata, or unrelated example interactions into a product. No web fonts, framework, image-generation tool, hosted preview, or remote service is required to use the bundled assets.

When making a new site, use the actual task to choose composition; preserve the visual language rather than repeating the specimen verbatim. When editing this system, keep the reference, foundation, and specimen consistent.

## Verification

Check that referenced assets load, text remains readable at narrow and wide widths and with enlarged browser text, and controls retain keyboard focus and useful labels. Exercise the interactions affected by the change. Compare with the accepted specimen to catch visual drift. Report any unperformed visual checks honestly; a Java compilation is unrelated to a visual-only change.
