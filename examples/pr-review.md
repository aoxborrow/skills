# Review notes

Optional. Put this file at `pr-review.md` or `.github/pr-review.md` in a repo, and
Claude review reads it from the PR's base branch before reviewing. Say what
matters most in the project; skip anything a linter or typechecker already
catches.

## Where bugs hide here

- **Code shared across runtimes.** `src/shared` runs in the server and the
  browser, so Node-only APIs there break the web build.
- **Writes with real-world effects.** Anything that sends email, charges a
  card or calls a third-party API must run only on an explicit user action,
  never twice on a retry, and report partial failure.
- **Credentials.** API keys must never reach logs, error messages, client
  bundles or exported files.
- **Stored data.** Old records must still load after a schema change; check
  migrations and defaults for missing fields.
- **Money.** Never add amounts in different currencies; watch rounding.

## Ignore

- Formatting, naming and import order.
- Generated files under `dist/` and lockfiles.
