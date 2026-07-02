# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This is a single static HTML landing page for "AI Workspace" (智能工作空间), a marketing site with no backend, build system, or package manager. The entire site is one self-contained file:

- `index.html` — the complete page: markup, all CSS (in a `<style>` block), no JavaScript. Content is in Simplified Chinese (`lang="zh-CN"`).
- `README.md` — one-line project title, no further documentation.

There is no `package.json`, build tooling, linter, test suite, or CI workflow in the repository. Nothing needs to be installed or compiled to work on this project.

## Development workflow

Because the site is a single static HTML file with inline CSS and no external dependencies, changes are made directly in `index.html`.

- **Preview locally**: open `index.html` directly in a browser, or serve it with any static file server, e.g. `python3 -m http.server` from the repo root, then visit `http://localhost:8000`.
- **No build/lint/test commands exist.** Verify changes by opening the page in a browser and visually checking the affected section.

## Structure of `index.html`

The page is a single scrolling layout composed of these sections, in order, each identifiable by class name:

1. `.navbar` — fixed top nav with logo, nav links (`#features`, `#pricing`, `#docs`, `#about`), and a CTA button.
2. `.hero` — headline, subtitle, and primary/secondary CTA buttons.
3. `.features` (`id="features"`) — a 3-column `.features-grid` of `.feature-card` items (icon, title, description).
4. `.cta-section` — secondary call-to-action banner.
5. `.footer` — multi-column footer (brand blurb + link columns for 产品/资源/公司) and a copyright bottom bar.

Styling conventions to preserve when editing:
- Dark theme: background `#0f1419`, card backgrounds use low-opacity white (`rgba(255,255,255,0.03–0.08)`).
- Accent gradient `#667eea → #764ba2` is reused across the logo icon, feature icons, primary button, and hero headline text (via `background-clip: text`).
- A single `@media (max-width: 968px)` breakpoint handles all responsive adjustments (nav links hidden, grids collapse to 1–2 columns, padding reduced). Add new responsive rules inside this same media query rather than introducing new breakpoints.
- Nav anchors (`#features`, `#pricing`, `#docs`, `#about`, `#start`) currently only `#features` has a matching section id; other links are placeholders.
