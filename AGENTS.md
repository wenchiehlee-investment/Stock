# Repository Guidelines

## Project Structure & Module Organization
This repository is a GitBook-based knowledge base. Content is primarily Markdown organized by topic folders at the repo root (for example `Financial/`, `US_STOCK/`, `chapter/`, `barons/`).

Core files:
- `README.md`: top-level landing content.
- `SUMMARY.md`: GitBook table of contents; update when adding/removing pages.
- `book.json`: GitBook version/plugins.
- `Makefile`: common local build/serve/deploy commands.
- `_book/`: generated static site output (build artifact; do not edit directly).

Keep images near their content in each module’s `images/` directory (for example `chapter/images/`).

## Build, Test, and Development Commands
- `make init`: initializes GitBook structure (`gitbook init`).
- `make serve`: runs local preview server (`gitbook serve`), typically on `http://localhost:4000`.
- `make build`: builds static site into `_book/` using GitBook 2.6 (`gitbook build --gitbook=2.6`).
- `make clean`: removes generated output (`rm -fr _book`).
- `make github`: publishes `_book/` via `ghp-import` (deployment helper).

If `gitbook` is missing, install the required CLI and Node runtime first.

## Coding Style & Naming Conventions
Use Markdown as the source of truth:
- Keep one topic per file with clear heading hierarchy (`#`, `##`, `###`).
- Prefer short sections and descriptive titles.
- Use relative links and local image paths (example: `![PE chart](./images/pe.png)`).
- Preserve existing folder/file naming where possible to avoid broken links.

## Testing Guidelines
There is no automated test suite in this repository. Validate changes by:
- Running `make serve` for visual/content review.
- Running `make build` to confirm the book compiles successfully.
- Spot-checking edited links, images, and `SUMMARY.md` navigation entries.

## Commit & Pull Request Guidelines
Recent history contains very short commit messages (for example `updateg`). For new work, prefer clear imperative messages like:
- `docs(financial): add cash-flow chapter notes`
- `docs(chapter): fix broken image links`

For PRs, include:
- A concise summary of changed folders/files.
- Any navigation updates in `SUMMARY.md`.
- Screenshots when layout or image-heavy pages are updated.
- Confirmation that `make build` succeeds locally.
