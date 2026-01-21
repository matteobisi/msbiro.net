# AGENTS

Purpose
-------
This file describes the repository for automated agents and future maintainers/coders. It explains the site technology, where to look for configuration and minimal customizations, and what automated actions (AI) are allowed.

Repository overview
-------------------
- This is a Hugo static site (a blog) using the PaperMod theme.
- Content lives in the `content/` directory; layouts and shortcodes are in `layouts/`.
- The site configuration and build options are defined in `hugo.yaml` at the repository root — consult it for build flags, baseURL, and other settings.
- The only site customization present in this repository is the Mermaid shortcode located at `layouts/shortcodes/mermaid.html`, which enables rendering Mermaid diagrams in posts.

Files and locations of interest
-------------------------------
- `content/` — blog posts and content pages.
- `layouts/` — custom templates and shortcodes. See `layouts/shortcodes/mermaid.html` for Mermaid support.
- `themes/` — PaperMod theme is used; if local theme overrides are needed, place them under `layouts/` or create a theme fork.
- `hugo.yaml` — main site configuration; use it as authoritative source for build-time configuration.

Guidance for agents and future coders
------------------------------------
- Agents may perform technical maintenance tasks: dependency updates, build script fixes, minor theme customizations, and adding or updating content.
- For any non-trivial structural or visual changes (theme forks, new shortcodes, architectural changes), open a branch and a PR for human review.
- When generating or editing content, respect existing content structure (front matter fields, tag taxonomy) and keep files under `content/`.
- Use the Mermaid shortcode for diagram rendering; follow the patterns used in existing posts when possible.

AI usage policy
----------------
- AI assistance is allowed for technical maintenance, content editing, and future customization of pages, theme, and site structure.
- All AI-generated changes must be reviewed by a human before merging; include rationale and any tests/build checks in the PR.

Practical notes
---------------
- To preview locally, run Hugo using the configuration in `hugo.yaml` (ensure Hugo is installed on your machine).
- Keep commits small and focused. Use clear commit messages and open PRs for review to preserve an audit trail.

Contact / Ownership
-------------------
Repository owner: Matteo Bisi (see repository metadata). For questions about architecture or governance, open an issue or PR.