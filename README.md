# Mushroom Pi — Documentation

This repository is the **published Mushroom Pi documentation site**, built with [Mintlify](https://mintlify.com) and rendered from the Markdown/MDX in this repo. Mintlify syncs it through its GitHub app and deploys on every push to `main`.

A reader finds here: a grower's guide to running a grow, the build and self-hosting instructions, the contributor architecture, and a generated API reference for the server and the Pico.

## Where to read it

The rendered site is the Mushroom Pi project's documentation site. (It is not this `README.md` — Mintlify does not render this file, and it is not a page on the site.)

## Working on it locally

Pages are MDX with YAML frontmatter; navigation, theme and branding live in `docs.json`.

```bash
mint dev --port 3001   # preview — never the default port: 3000 collides with mushpi-server
```

Before a change is done, it must pass:

```bash
mint validate        # validates the site, including the referenced OpenAPI specs
mint broken-links    # no broken internal link
```

## How the content is organised

| Area | What lives there |
| --- | --- |
| Getting Started | What Mushroom Pi is, the first-run path, the three-tier shape |
| Build & Self-Host | Building the units and deploying the hub |
| Using Mushroom Pi | The grower guide to the dashboard |
| Configuration | The firmware control loop and remote access |
| Contributing | Architecture (C4) and Development |
| License | Which licence covers which material |
| Releases | What a release number means to a reader |
| API Reference | The server and Pico endpoints, generated from their OpenAPI specs |

## Writing for the site

The conventions live in `AGENTS.md` — page anatomy, the navigation gate (`docs.json`), and the public-content boundary — so they are not restated here. That file also carries agent-facing instructions; it is not a page and is never added to `docs.json`.

## API Reference

The endpoint pages are **generated** by Mintlify from the server's and the Pico's OpenAPI specifications, referenced by URL from `docs.json`. Nothing about them is written by hand: a change to an endpoint belongs in the relevant specification, not in this repository.

## Licensing

The site publishes material under several licences: documentation prose and diagrams under **CC BY-SA 4.0**, and hardware material under **CERN-OHL-S 2.0** — both scoped in this repository. The application code and firmware described here carry their own licences in their own repositories. See the site's License page for the detailed statement, and `LICENSE-docs` / `LICENSE-hardware` in this repository for the texts.
