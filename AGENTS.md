# Mushroom Pi — mushpi-docs (published Mintlify site)

The **public documentation site** for the Mushroom Pi three-tier system, built with Mintlify. Pages are MDX with YAML frontmatter; navigation, theme and branding live in `docs.json`. Mintlify syncs this repository through its GitHub app and deploys on every push to `main`.

Three regressions this repo must not cause: publishing content that is not public, breaking the site's navigation by adding a page that `docs.json` does not list, and letting a page drift from the fact it describes. "Done" is defined under Verification.

## Map

| Path | What exists there |
|---|---|
| `index.mdx` | Landing page |
| `introduction.mdx` · `quickstart.mdx` · `system-overview.mdx` | Getting Started: what Mushroom Pi is, the first-run path, the three-tier shape |
| `hardware/` | Building the units: `components.mdx`, `wiring.mdx`, `pico-firmware.mdx` |
| `deployment/` | Deploying the hub: `sd-card-provisioning.mdx`, `self-hosting.mdx`, `environment.mdx`, `first-boot.mdx`, `pico-registration.mdx`, `updates-and-backups.mdx`, `host-wifi-recovery.mdx` |
| `dashboard/` | Using Mushroom Pi — the grower guide: `index.mdx`, `pico-units.mdx`, `recipes.mdx`, `batches.mdx`, `monitoring.mdx`, `finishing-a-batch.mdx`, `server.mdx`, `settings.mdx`, `troubleshooting.mdx` |
| `configuration/` | `hysteresis-control.mdx` (the firmware control loop), `remote-access.mdx` |
| `development/` | Contributors: `mock-pico.mdx`, `contributing.mdx` |
| `api/` | API Reference section — `overview.mdx` only. The endpoint pages are **generated** by Mintlify from the two OpenAPI specifications, referenced by raw URL in `docs.json`: the server spec and the Pico spec, each in its own group. There is no hand-written endpoint page |
| `docs.json` | **The site's index**: navigation tabs/groups, theme colours, logo, navbar, and any OpenAPI spec references. A page not listed here is not published |
| `logo/` · `favicon.svg` | Brand assets |
| `.mintignore` | Files Mintlify must not render |
| `.atlas-analysis.json` | Atlas code-generation artifact — see `REFERENCE.md` |
| `AGENTS.md` · `REFERENCE.md` | Instruction files. **Not pages** — never add them to `docs.json` |

## Where the content comes from

The site is the **destination** of a migration. The pre-Mintlify tree still holds material that has not moved yet.

| Tree | Role |
|---|---|
| `mushpi-docs-deprecated/` (local-only, no remote) | The **pre-Mintlify migration source** — C4 architecture, hardware, the user guide, presentations, `versioning.md`, `diagram-theme.md`. Read-only while it exists, and **deleted when the migration completes**: content *moves* here → site, nothing is archived. Denied in the docs agents' `edit` maps |
| workspace-root `docs/` (in `mushpi-orchestration`) | **Private** orchestration documentation — topology, model routing and history, tooling history. Never published here |

Migrating a page is a **move, not a rewrite**: place the already-written content, convert `.md` → `.mdx`, add its path to `docs.json`. If the source needs new prose to make sense on the site, that is `mushpi-docs`'s call, not the bulk agent's.

## Canonical facts — name the owner, never copy the fact

| Fact | Owner |
|---|---|
| Server API shape | `mushpi-server/spec/openapi.json` (generated on pre-commit) |
| Pico REST API shape | `mushpi-grow/spec/openapi.yaml` (hand-authored) |
| Hardware BOM, pin map, power data | `mushpi-grow/HARDWARE.md` (electronics agent owns it — never edit) |
| Packaging, release and deployment decisions | `mushpi-ops/MAINTAINING.md` (internals) · `mushpi-ops/DEPLOYMENT.md` (operator-facing) |
| Version and release semantics | `releases.mdx` for what a release number means to a reader; the **normative** rules live in `mushpi-ops/MAINTAINING.md` §Version & Release Semantics |
| Feature status, phase, priority, effort | The GitHub Project `mushroom-pi` #1 — read via `mushpi-product-owner` |
| Diagram palette and emoji map | `mushpi-docs-deprecated/diagram-theme.md` |
| Mermaid shape/legend conventions | The `mermaid-diagram-conventions` skill |
| Per-repo coding rules | Each sub-repo's own `AGENTS.md` |

## Hard gates

1. **New gotcha → `REFERENCE.md`**, never this core.
2. **Structural change → the index row in the same change.** A new, moved, renamed or deleted page updates its `docs.json` navigation entry (and any table above) in the same commit — as a row, not prose.

## Content boundary — public only

Never publish: any `AGENTS.md`/`REFERENCE.md`; anything from the workspace-root `docs/` folder (model routing and model history are private by policy); the archive's raw shop-page clippings (`mushpi-docs-deprecated/hardware/components/clippings/` — excluded from both licenses, copyright-risky, and not documentation); and any credential, token, local path or private hostname. When in doubt, ask the orchestrator rather than guessing — the site is public the moment it is pushed.

## Authoring rules

- **MDX with YAML frontmatter** (`title:`, `description:`) — set both; Mintlify uses them for nav, search and previews. Sentence case for headings; bold for UI elements; code formatting for files, commands, paths and identifiers.
- **Portable-first.** Plain Markdown and tables are the default. Mintlify-specific components (`<CardGroup>`, `<Accordion>`, `<Steps>`) belong in landing and navigation-style pages, not spread through every page — vendor lock-in is mitigated by keeping the prose re-renderable.
- **Cross-repo references are code-spans or absolute URLs, never relative links.** The sub-repos and this site are separate repositories; `../../mushpi-server/...` resolves nowhere on a published site. Same rule as the archive.
- **Mermaid:** one `mermaid` fenced block per diagram; Mintlify renders it natively, so a diagram must be checked *in the preview*, not only parsed. Load the `mermaid-diagram-conventions` skill and use the project theme before touching any diagram.
- **Never restate feature status.** Status lives in the tracker; a page that needs it links to the tracker.
- **Link discipline:** relative links between site pages only; verify with `mint broken-links` before reporting done.

## Verification ("done" for this repo)

1. `mint validate` — passes (and validates any OpenAPI spec referenced from `docs.json`).
2. `mint broken-links` — no broken internal link.
3. A visual check in `mint dev --port 3001` for any page whose diagram, table or component you touched. **Never the default port**: `mint dev` binds 3000, which collides with `mushpi-server`.
4. Every structural change has its `docs.json` navigation entry and its map row above.
5. Facts come from the owner files in the table — never invented; docs↔code disagreement is flagged, never silently reconciled.

## Instruction files are off-limits to the docs agents

`AGENTS.md` and `REFERENCE.md` here are denied in the docs agents' `edit` permission maps and are owned by the orchestrator. Report a needed change instead of making it. Pages, `docs.json` and site assets are the docs agents' territory; `mushpi-docs-bulk` places supplied or migrated content, `mushpi-docs` writes prose.
