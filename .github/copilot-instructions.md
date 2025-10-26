<!-- .github/copilot-instructions.md -->
# Repo overview for AI code assistants

This repository is a Hugo static site that uses the PaperMod theme. The guidance below is focused on concrete, discoverable patterns and commands that make an AI coding agent productive when editing, building, or debugging this site.

- Big picture
  - This is a Hugo site. Root config: `hugo.yaml` (site-wide params and flags). Theme is `themes/PaperMod` (PaperMod theme from adityatelange).
  - Source content lives in `content/` (posts, pages). Archetypes are in `archetypes/`.
  - Layouts come from two places: project-level `layouts/` and theme-level `themes/PaperMod/layouts/`. Theme files are authoritative unless overridden in the project `layouts/`.
  - `assets/` and `themes/PaperMod/assets/` contain CSS/JS that are processed by Hugo's asset pipeline (bundling, fingerprinting, minification). The theme expects Hugo asset processing (no Node toolchain required).
  - `public/` is the generated site output (already present in repo). Treat `public/` as build artifact — do not edit it directly.

- Key workflows & commands (sourceable from repo)
  - CI / deploy: see `themes/PaperMod/.github/workflows/gh-pages.yml`. The workflow installs Hugo and runs:
    - `hugo --buildDrafts --gc --baseURL <pages-base-url>` for GitHub Pages builds.
  - Local development (standard Hugo): start a dev server and preview drafts with Hugo's server mode. Typical commands (PowerShell):
    - `hugo server -D`  # preview drafts and run a local dev server
    - `hugo`           # build static files to `public/`
  - If the theme is managed as a git submodule in other forks, the workflow uses `git submodule update --init --recursive` and `git submodule update --remote --merge`. If you change the theme, update submodule usage accordingly.

- Project-specific conventions and patterns
  - Minimum Hugo version: the theme README requires >= v0.146.0. Prefer that Hugo version when replicating CI builds.
  - Search and client-side behavior: PaperMod uses Fuse.js and fastsearch (see `themes/PaperMod/assets/js/fastsearch.js` and `assets/js/fuse.basic.min.js`), so be careful when modifying search indexing templates — changes affect client-side search behavior.
  - Edit links: `hugo.yaml` includes an `editPost.URL` param that points to the GitHub content path. Keep this param consistent if moving content locations.
  - TOC and code highlighting are driven by Hugo markup config in `hugo.yaml` (goldmark unsafe renderer, Chroma highlighting). Respect `markup` config when editing templates that emit HTML.
  - The theme relies on Hugo's asset pipeline (fingerprinting/minification). Avoid manual bundling or committing built CSS/JS in `themes/PaperMod/assets`.

- Integration points & external dependencies
  - External libraries: Fuse.js (`themes/PaperMod/assets/js/fuse.basic.min.js`) and fastsearch are used for on-site search.
  - No Node.js build required for the theme: the theme CSS/JS are processed by Hugo (asset pipeline). Use Hugo CLI rather than npm scripts for production builds.
  - GitHub Pages deployment is via the theme's workflow and uses the GitHub Actions Pages deployer.

- Where to look for examples
  - Site config: `hugo.yaml` — shows params, analytics, search options, and build flags.
  - Theme usage and override points: `themes/PaperMod/layouts/_default/` (index, list, single templates), and `themes/PaperMod/partials/` for common partials (header, footer, toc, post_meta).
  - Client JS: `themes/PaperMod/assets/js/fastsearch.js` and `assets/js/fuse.basic.min.js`.

- Editing guidance for an AI agent
  - Change content in `content/` and templates in `layouts/` or `themes/PaperMod/layouts/` (prefer project-level `layouts/` overrides when customizing behavior for this site).
  - Do not edit `public/` (generated). If a change looks like it only affects `public/`, update the corresponding source in `content/`, `layouts/`, or `assets/` and re-run `hugo`.
  - When modifying search/indexing templates, include a small smoke test: run `hugo` locally and open `public/index.html` (or run `hugo server -D`) to confirm search assets are emitted and search page loads without console errors.

- Quick checklist to verify a change
  - Run `hugo server -D` and verify the dev server serves pages and templates render without template errors.
  - Run `hugo` and confirm `public/` updates. Check that fingerprinted asset filenames (hashes) are generated in `public/assets/`.
  - If the change touches theme submodule use, run `git submodule update --init --recursive` before building.

If any of the above areas are unclear or you want more examples (for example: a sample content front matter pattern from `archetypes/default.md` or a short test that validates the search index), tell me which part to expand and I'll iterate.
