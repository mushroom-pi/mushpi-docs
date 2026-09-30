# mushpi-docs — REFERENCE

On-demand long-tail material for the docs agents. Core map, hard gates, content boundary and verification commands: `AGENTS.md`. Only read the section you need.

## Mintlify tier and what it does not give us

The site runs on Mintlify's **free Starter tier** (Pro is $450/mo annual; a free-Pro application to their OSS program is tracked as an issue). What Starter includes: custom domain, web editor, search, Git sync, API playground, custom CSS/JS, built-in components, SEO/GEO.

What it does **not** include, and therefore changes how we work:

- **No preview deployments.** There is no per-branch preview URL, so "review before publication" has to be solved some other way — a local `mint dev` preview, a pull request, or a manual gate. Do not promise a Mintlify preview in a review workflow.
- **No analytics, no admin APIs, no webhooks, no rollbacks.**
- **Multi-repo is Enterprise-only** — this site is the only repository Mintlify can read, so a page cannot link to a file in a sibling repository. A spec, however, **can be referenced by URL**: `docs.json` points the API Reference groups at the raw GitHub URLs of the server's and the Pico's specifications, so nothing is vendored here. Two consequences to remember: the spec is read from the **pushed** branch, so `mint validate` and `mint dev` judge what is pushed rather than what is in a sibling working tree; and a spec change does **not** trigger a docs rebuild (the deploy trigger is tracked as `mushpi-ops#6`).
- **No PDF export** on Starter; `mint export` produces the offline snapshot instead.

## CLI mechanics

- **`mint dev` binds port 3000, which collides with `mushpi-server`.** Always `mint dev --port 3001`.
- `mint validate` validates `docs.json` and any OpenAPI spec it references; `mint broken-links` checks internal links; `mint a11y`, `mint format`, `mint score` (agent-readiness audit: `llms.txt`, `skill.md`, MCP discoverability) exist too.
- `mint export` produces a self-contained offline zip of the site — the mechanism behind the offline-snapshot issue.
- `mint index --opencode` installs Mintlify's docs-search MCP into OpenCode.
- Node 20.17+ is required (v24.15.0 on the maintainer's machine).
- If `docs.json` is missing, pages resolve to 404 — a dev-server "not running" symptom is usually a wrong working directory.
- **Overlays**: an OpenAPI overlay is a separate JSON/YAML file that transforms a specification without editing it, applied **after parsing and before validation**. It is the tool for bending a spec maintained elsewhere — renaming paths, replacing server URLs, hiding endpoints — and it is referenced from `docs.json` next to the spec. **A target that matches nothing fails silently**, not loudly: malformed overlay syntax fails `mint validate`, but a mistyped JSONPath target passes validation and simply changes nothing. Check a new overlay's targets against the fetched specification rather than trusting a green validate. The server API's conditional bearer declaration lives this way, in `overlays/server-authentication.yaml`, deliberately **not** in the committed specification — the client regenerates its API layer from that spec, so a document-wide security requirement there could change generated call signatures and break the app's build to improve a docs page.

## Branding and theme mechanics

The site's colours are the app's colours, measured rather than eyeballed. Every gotcha here cost a build to find.

- **The `colors` slot names are counter-intuitive.** `colors.primary` is emphasis in **light** mode, `colors.light` is emphasis in **dark** mode, and `colors.dark` is filled buttons and hover states in **both**. Do not "correct" them into something sensible-looking.
- **`#2F7349` is a docs-only derived colour.** It is the client's `brand.leaf` (`#3C8D5A`) darkened ~15% so light-mode links clear WCAG AA on white (4.08:1 → 5.72:1). It does **not** exist in `mushpi-client/src/theme/tokens.ts` — a future token sync must never round-trip it back into the app. Dark-mode emphasis is `brand.accent` `#C66F2F` verbatim (4.91:1 on `#0f1720`); `background.color.dark` is `brand.bg` `#0f1720` with `decoration: gradient` echoing the app's body fade to `brand.bgDeep` `#071013`.
- **`appearance.default` is `dark`**, matching the app's dark-only identity, but the toggle is deliberately still available — so every branding change is checked in **both** modes, not just the default one.
- **`logo.light`/`logo.dark` mean page modes** — the opposite of the `colors.*` meaning above — and Mintlify's own schema prose describes them backwards. The rendered classes (`dark:hidden` / `hidden dark:block`) are authoritative. Practically: `logo/dark.svg` is the client lockup verbatim, `logo/light.svg` is the same canvas with the circuit strokes darkened to `#0e7490` (5.36:1 on white; the original `#7dd3fc` is 1.67:1 and vanishes on a light navbar). Same canvas in both, or the header jumps when the mode toggles.
- **The logo is sized in the root `style.css`, and on desktop it drags the whole header with it.** Mintlify caps the navbar logo at `h-7` (28px), which renders the lockup's wordmark at ~11px — sub-legible, and because the `name` text is `sr-only` behind the image, the wordmark is the *only* visible brand text. Our file sets `2.5rem` as the base (40px logo, ~16.1px wordmark; the mobile toolbar keeps ~180px of slack at 375px) and `@media (min-width: 1024px)` raises it to `6.5rem` — a 104px logo, 215.7px wide, **41.8px** wordmark, since wordmark = `height × (38.8056 / 96.486992)`. A 104px logo cannot fit a 64px bar, so the header is grown to match: `#navbar .h-16 { height: auto; min-height: calc(6.5rem + 1.5rem) }` (the bar has **zero** vertical padding — the slack around the logo is flex centring), making the desktop sticky header **176px** (bar 128 + nav-tabs 48) instead of 112px. That is the accepted cost, not a bug. `6.5rem` is coupled across **three** places in the file — the logo height, the bar `min-height`, and the header variable below — so changing the size means changing all three.
- **Two header-offset variables, and they are not interchangeable.** `--mintlify-slot-header-height` is referenced only as `var(--mintlify-slot-header-height, 7rem)` and Mintlify **never defines it**, so `style.css` must set it (it drives the left sidebar's `top`/`height`); on desktop it is `calc(6.5rem + 1.5rem + 3rem)` = 11rem. `--scroll-mt` is the opposite: **Mintlify's JS sets it at runtime** to header height + 40px, so it must **not** be set in CSS — and it is the reason anchor jumps still land correctly under the taller header.
- **The right-hand "On this page" TOC is the fragile part of the logo sizing.** Its `top-[9.5rem]` and `h-[calc(100vh-9.5rem)]` are hardcoded and do not read the header variable, so `style.css` re-derives them with attribute selectors (`[class*="top-[9.5rem]"]`, `[class*="h-[calc(100vh-9.5rem)]"]`, each matching exactly one element) from `--mintlify-slot-header-height` + 2.5rem. **This override fails silently** — the same failure mode as a mistyped overlay target — if a Mintlify update renames those utility classes: the symptom is the TOC sliding up beneath the header, and the check is to grep the rendered HTML for `top-[9.5rem]`. The banner variant of those rules (`top-[12rem]` / `h-[calc(100vh-15rem)]`) is deliberately untouched and untested because the site has no banner; enabling one needs the same adjustment.
- **Mintlify applies any `.css` file in the content directory site-wide** (the dev server logs `added style.css`), so this file is the lever for every global override — never inline styles in a page. Selector craft that has already proven safe here: `img.nav-logo` (Mintlify's own class, present on exactly the two navbar slots and nothing else, so it cannot catch a content image) and `#navbar .h-16` (the only `h-16` inside the navbar). `!important` is load-bearing: the logo size comes from the Tailwind `.h-7` utility, which carries none. Any change is re-checked in **both** modes — the two logo variants must stay identical in size, or the header shifts when the mode toggles.
- **The favicon is never referenced as an SVG.** Mintlify resizes `favicon.svg` into a raster chain under `/favicons/*` — `favicon-16x16.png`, `favicon-32x32.png`, `apple-touch-icon.png`, `favicon.ico` — and links *those*. So "the favicon won't show" is normally the browser's per-origin favicon cache (which survives ordinary reloads), not the configuration. Before changing anything, evidence it: `curl -sI localhost:3001/favicon.svg` (expect `200 image/svg+xml`) and grep the emitted `<link rel="icon">` tags; then confirm with a hard reload or a private window.
- **A filled button's label colour is not in our control** — it resolves through the c15t consent library's `--c15t-surface` token, so it is only observable in a live browser. The numbers that matter if we ever swap in a fallback: white on `colors.dark` `#C66F2F` is 3.67:1 (below AA), the app's own dark label `#0f1720` on it is 4.91:1.
- **One residual risk to re-check after any palette change:** `colors.primary` `#2F7349` on the dark background `#0f1720` is only 3.15:1. Dark-mode emphasis consistently uses `--primary-light` (the orange) in the CSS we inspected, so nothing currently trips this — but an element that picks up `--primary` in dark mode would be low-contrast.

## What Mintlify ignores automatically

`.mintignore` documents the built-ins: `.git`, `.github`, `.claude`, `.agents`, `.idea`, `node_modules`, `README.md`, `LICENSE.md`, `CHANGELOG.md`, `CONTRIBUTING.md`. Our additions live in that file (drafts). `AGENTS.md`/`REFERENCE.md` are **not** auto-ignored — they are simply never listed in `docs.json`, so they never render as pages. Never add them to the navigation.

## Diagram theme

Project-specific colours and emojis for Mermaid diagrams. This is the **single source of truth** for the project's palette; the `mermaid-diagram-conventions` skill supplies the generic rules (shape semantics, layout, edge labelling, legend pattern, narrative, verification checklist) and this table overlays them. Apply the skill first, then this theme.

One colour family per architectural layer via `classDef`. Within a family, use the **lighter** fill for planner/analysis roles and the **darker** fill for builder/execution roles.

| Layer | Light | Dark | Notes |
|-------|-------|------|-------|
| Users | `#8b5cf6` | — | Human operators (purple) |
| Pico firmware / Grow chain | `#6366f1` | `#3b82f6` | MicroPython layer (blue) |
| Server chain | `#f59e0b` | `#d97706` | NestJS backend (amber) |
| Client chain | `#10b981` | `#059669` | React frontend (green) |
| Database | `#f97316` | — | SQLite storage (orange) |
| Hardware / External | `#ef4444` | — | Physical environment, external systems (red) |
| Cron / Async tasks | `#a855f7` | — | Background jobs (purple) |
| Config | `#6b7280` | — | Configuration, infrastructure (gray) |
| Docs | `#14b8a6` | — | Documentation artifacts (teal) |
| Orchestrator | `#0ea5e9` | — | Coordination / gateway role (sky) |
| Mock | `#6366f1` | — | `mushpi-mock` uses the grow colour family (blue) |
| Product owner / Tracker | `#ec4899` | — | Backlog and board ownership — the non-technical product/governance layer (rose) |

For wiring and schematic diagrams, colour-code edges by signal type and include the inline legend (the skill's Section 8 legend pattern).

| Signal | Colour | Notes |
|--------|--------|-------|
| Power (5V / VBUS) | `red` | |
| Ground (GND) | `black` | |
| Data / GPIO | `blue` | |
| Relay output | `green` | |
| Not-installed component | `grey`, dashed (`-..->`) | Add a "(NOT INSTALLED)" note |

| Entity | Emoji | Entity | Emoji |
|--------|-------|--------|-------|
| User | 👤 | Pico / Grow | 🌱 |
| Server | 🖥️ | Client | 🖼️ |
| Database | 🗄️ | Environment | 🌿 |
| Planner (any) | 🔍 | Builder (any) | 🛠️ |
| Orchestrator | 🤖 | Electronics | ⚡ |
| Docs | 📝 | Mock | 🛠️ |
| HTTP layer | 🌐 | Swagger | 📋 |
| Pico Units | 📡 | Readings | 📊 |
| Batches | 📦 | Recipes | 📝 |
| Control | 🎮 | Dashboard | 📈 |
| Settings | ⚙️ | Cron | ⚡ |
| Config | ⚙️ | Monitoring | 🏥 |
| Sensors | 🌡️ | Relay | ⚙️ |
| Humidifier | 💧 | Fan | 🌀 |
| Heating Mat | 🔥 | DHT11 | 🌡️ |
| USB Power | ⚡ | Force-Provision | 🔘 |
| Product owner / Tracker | 🗂️ | | |

The C4 component pages invent sub-layer classes (`http`, `monitoring`, `entry`, `state`, `router`, `query`, `chart`, `form`, `exec`) whose colours are not in this table, and a few collide with the layers above — teal is Docs, rose is the product owner. They are left as they are rather than re-themed; if a diagram is redrawn, bring its classes back onto this palette.

Note that a `subgraph` id must never match a `node` id: it parses cleanly and then silently drops the node.

## Migrating a page in from the archive

Applies only while the pre-Mintlify migration source still exists — it is deleted when the migration completes, and this section goes with it.

1. Read the source page in `mushpi-docs-deprecated/` (never write to it).
2. Decide the destination path under the site's existing sections — reuse a section, do not invent a new top-level one without the orchestrator.
3. Convert `.md` → `.mdx`: add frontmatter (`title`, `description`), keep the body verbatim where possible, and re-check Mermaid blocks render in the preview.
4. Add the page path to `docs.json` **in the same change**.
5. Re-point cross-repo references to code-spans or absolute URLs, and relative links to site-relative ones. Run `mint broken-links`.
6. Anything deliberately *not* migrated is a decision to record, not a silent omission.

## MDX pitfalls worth knowing before a conversion

- `{` and `<` are meaningful in MDX — an unescaped brace in prose or a template-looking `<word>` can break the build. Wrap in backticks or escape.
- Frontmatter is YAML; a colon inside a title needs quoting.
- HTML comments render as text in MDX; use `{/* … */}` for a real comment.
- Images live in the repo and are referenced by site-relative path; a page needing a screenshot needs the asset committed here first.
- Mermaid labels reject backslash-escaped quotes — `\"word\"` terminates the label early and fails the parse. Use the entity form `#quot;word#quot;` instead, and check each diagram in a rendered preview.

## `.atlas-analysis.json`

An artifact of Mintlify's Atlas generation, which produced the initial page set by reading the organisation's repositories and READMEs. It is a generator's record, not a page and not a fact source. It is now stale — it snapshots a navigation the site no longer has — so delete it rather than maintain it; the owner table in `AGENTS.md` is the canonical source.

## Voice and structure notes

- Second person, active voice, one idea per sentence — the site is read by a grower with a Pico on the bench, not by a contributor browsing a repository.
- A page opens with what the reader will be able to do, then the steps, then troubleshooting.
- Hardware pages must match `mushpi-grow/HARDWARE.md`. **The deployment pages are themselves the operator guide** — `mushpi-ops/DEPLOYMENT.md` was reduced to release-repo notes pointing here, so a change to a deployment procedure lands on the site and the release repo follows, never the other way round. Disagreement anywhere is a flag to the orchestrator, never a silent edit on the site.
