# Review notes

Claude review reads this file (`pr-review.md` or `.github/pr-review.md`) from the
PR's base branch. Say what matters most in the project; skip what a linter
already catches.

## Where bugs hide here

- **The review workflow is security-sensitive.** `reusable-claude-review.yml`
  runs Claude on untrusted PR text, forks included. The model job must keep a
  read-only token and only Read, Grep and Glob; only the model-free `publish`
  job may write. PR code must never be checked out as the workspace or run, and
  nothing from the PR may reach a `run:` script through `${{ }}`.
- **Credentials stay out of the model's reach.** The federation token and its
  cached credential live under `RUNNER_TEMP`; the `settings` deny rules keep
  Claude's file tools out of it, `/proc` and `.git`, and `publish` refuses
  anything credential-shaped. Check that new steps don't write secrets where
  Claude can read them.
- **Every repo calls `reusable-claude-review.yml@main`.** A change here reaches
  all of them at once, so check that existing callers (`claude-review.yml`)
  still work.
- **Skills are instructions to Claude.** Check that a `SKILL.md` has a clear
  `description` saying when to use it, and that its steps are safe to run as
  written.

## Ignore

Wording and formatting in Markdown, unless it changes what a skill tells Claude
to do.
