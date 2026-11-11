# Review notes

Claude review reads this file (`pr-review.md` or `.github/pr-review.md`) from the
PR's base branch. Say what matters most in the project; skip what a linter
already catches.

## Where bugs hide here

- **The review workflow is security-sensitive.** `pr-review.yml` runs Claude on
  untrusted PR text, forks included. The model job must keep a read-only token
  and only Read, Grep and Glob; only the model-free `publish` job may write. PR
  code must never be checked out as the workspace or run, and nothing from the
  PR may reach a `run:` script through `${{ }}`.
- **Every repo calls `pr-review.yml@main`.** A change here reaches all of them at
  once, so check that existing callers (`claude-review.yml`) still work.
- **Skills are instructions to Claude.** Check that a `SKILL.md` has a clear
  `description` saying when to use it, and that its steps are safe to run as
  written.

## Ignore

Wording and formatting in Markdown, unless it changes what a skill tells Claude
to do.
