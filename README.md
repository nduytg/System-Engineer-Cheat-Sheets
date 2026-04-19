# System Engineer Cheat Sheets

This repository curates practical, task-oriented guides for junior system
engineers. All content now lives in Markdown with consistent metadata, headings,
and fenced code blocks to improve readability and collaboration.

## Repository layout

The repository now follows a GitHub Pages-friendly Jekyll content model:
- `_docs/` for evergreen cheat sheets.
- `_posts/` for dated blog entries.
- `_projects/` for platform engineering case studies.
- `index.md` as the homepage with clear paths to Cheat Sheets and Blog.

Legacy topic folders remain in place during the transition and can be gradually moved or aliased into `_docs/`.

Cheat sheets are organized by topic. Each directory contains one or more
`*.md` files.

- `Automation/` – Ansible primers and automation playbooks.
- `Backup/` – Rsync- and script-based backup guides, including rotation scripts.
- `Docker/` – Multi-part Docker tutorials plus networking and storage notes.
- `Firewall/`, `High Availability/`, `Load Balancing/`, `NTP/`, `SSH/`,
  `Tunning kernel/`, `Utils/`, `Web Services/` – Platform-specific references
  for core services.

## Local development

1. [Install pre-commit](https://pre-commit.com/#install) if you do not already
   have it available.
2. Install the hooks for this repository:

   ```bash
   pre-commit install
   ```

3. Run the hooks before pushing changes:

   ```bash
   pre-commit run --all-files
   ```

## Linting

Markdown linting is enforced in two ways:

- A pre-commit hook runs `pymarkdown` to scan all Markdown files locally.
- A GitHub Actions workflow (`Markdown lint`) executes the same check on every
  push and pull request.

Install the linter with `pip install pymarkdownlnt` or via `pre-commit`'s
managed environments. Running the checks keeps the cheat sheets consistently
formatted and ready for publishing.

## CI/CD

GitHub Actions now uses two workflows:

- `Markdown lint` (`.github/workflows/markdown-lint.yml`) for Markdown quality checks on pushes and pull requests.
- `Build and Deploy GitHub Pages` (`.github/workflows/pages-deploy.yml`) to build the Jekyll site on pull requests and deploy to GitHub Pages on pushes to `main`.

## GitHub Pages domain

This repository is configured to use the default GitHub Pages domain pattern (no custom `CNAME`).

- User site repo URL: `https://<username>.github.io/`
- Project site repo URL: `https://<username>.github.io/<repository>/`

For this repository, expect the project-site URL pattern to apply unless it is published from a dedicated user-site repository.
