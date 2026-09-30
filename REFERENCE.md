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
