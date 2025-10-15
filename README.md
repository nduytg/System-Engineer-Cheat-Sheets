# System Engineer Cheat Sheets

This repository curates practical, task-oriented guides for junior system engineers. Each cheat sheet is now written in Markdown so it is easier to browse, review and collaborate on changes.

## Repository layout

Cheat sheets are grouped by topic. Each directory contains one or more `*.md` files.

- `Automation/` – Ansible primers and automation playbooks.
- `Backup/` – Rsync- and script-based backup guides.
- `Docker/` – Multi-part Docker tutorials, including networking and storage notes.
- `Firewall/`, `High Availability/`, `Load Balancing/`, `NTP/`, `SSH/`, `Tunning kernel/`, `Utils/`, `Web Services/` – Additional platform-specific references.

The legacy `.txt` files have been superseded by Markdown equivalents with consistent headings and metadata.

## Local development

1. [Install pre-commit](https://pre-commit.com/#install) if you do not already have it available.
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
- A GitHub Actions workflow (`Markdown lint`) executes the same check on every push and pull request.

Following these checks keeps the cheat sheets consistently formatted and ready for publishing.
