# mushpi-docs — REFERENCE

On-demand long-tail material for the docs agents. Core map, hard gates, content boundary and verification commands: `AGENTS.md`. Only read the section you need.

## Mintlify tier and what it does not give us

The site runs on Mintlify's **free Starter tier** (Pro is $450/mo annual; a free-Pro application to their OSS program is tracked as an issue). What Starter includes: custom domain, web editor, search, Git sync, API playground, custom CSS/JS, built-in components, SEO/GEO.

What it does **not** include, and therefore changes how we work:

- **No preview deployments.** There is no per-branch preview URL, so "review before publication" has to be solved some other way — a local `mint dev` preview, a pull request, or a manual gate. Do not promise a Mintlify preview in a review workflow.
- **No analytics, no admin APIs, no webhooks, no rollbacks.**
- **Multi-repo is Enterprise-only.** This site is the *only* repository Mintlify can read, so a page cannot reference `mushpi-server/spec/openapi.json` or `mushpi-grow/spec/openapi.yaml` in place — an in-place API reference is not possible on the current tier. If a spec has to appear here, a copy has to live here. Treat that as a blocked dependency rather than inventing a workaround.
- **No PDF export** on Starter; `mint export` produces the offline snapshot instead.

## CLI mechanics

- **`mint dev` binds port 3000, which collides with `mushpi-server`.** Always `mint dev --port 3001`.
- `mint validate` validates `docs.json` and any OpenAPI spec it references; `mint broken-links` checks internal links; `mint a11y`, `mint format`, `mint score` (agent-readiness audit: `llms.txt`, `skill.md`, MCP discoverability) exist too.
- `mint export` produces a self-contained offline zip of the site — the mechanism behind the offline-snapshot issue.
- `mint index --opencode` installs Mintlify's docs-search MCP into OpenCode.
- Node 20.17+ is required (v24.15.0 on the maintainer's machine).
- If `docs.json` is missing, pages resolve to 404 — a dev-server "not running" symptom is usually a wrong working directory.

## What Mintlify ignores automatically

`.mintignore` documents the built-ins: `.git`, `.github`, `.claude`, `.agents`, `.idea`, `node_modules`, `README.md`, `LICENSE.md`, `CHANGELOG.md`, `CONTRIBUTING.md`. Our additions live in that file (drafts). `AGENTS.md`/`REFERENCE.md` are **not** auto-ignored — they are simply never listed in `docs.json`, so they never render as pages. Never add them to the navigation.

## Migrating a page in from the archive

1. Read the source page in `mushpi-docs-deprecated/` (never write to it).
2. Decide the destination path under the site's existing sections — reuse a section, do not invent a new top-level one without the orchestrator.
3. Convert `.md` → `.mdx`: add frontmatter (`title`, `description`), keep the body verbatim where possible, and re-check Mermaid blocks render in the preview.
4. Add the page path to `docs.json` **in the same change**.
5. Re-point cross-repo references to code-spans or absolute URLs, and relative links to site-relative ones. Run `mint broken-links`.
6. Anything deliberately *not* migrated (the clippings, private orchestration material) is a decision to record, not a silent omission.

## MDX pitfalls worth knowing before a conversion

- `{` and `<` are meaningful in MDX — an unescaped brace in prose or a template-looking `<word>` can break the build. Wrap in backticks or escape.
- Frontmatter is YAML; a colon inside a title needs quoting.
- HTML comments render as text in MDX; use `{/* … */}` for a real comment.
- Images live in the repo and are referenced by site-relative path; the archive's photos are untracked scratch, so a page needing a screenshot needs the asset committed here first.

## `.atlas-analysis.json`

An artifact of Mintlify's Atlas generation, which produced the initial page set by reading the organisation's repositories and READMEs. It is a generator's record, not a page and not a fact source. If it becomes stale it is deleted rather than maintained; do not treat its contents as canonical — the owner table in `AGENTS.md` is.

## Voice and structure notes

- Second person, active voice, one idea per sentence — the site is read by a grower with a Pico on the bench, not by a contributor browsing a repository.
- A page opens with what the reader will be able to do, then the steps, then troubleshooting.
- Hardware pages must match `mushpi-grow/HARDWARE.md`; deployment pages must match `mushpi-ops/DEPLOYMENT.md`. Disagreement is a flag to the orchestrator, never a silent edit on the site.
