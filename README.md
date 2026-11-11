# skills

My Claude Code skills, plus a reusable Claude PR review workflow I use across my repos.

## Skills

Skills live in `skills/<name>/SKILL.md`. Install them in Claude Code:

```
/plugin marketplace add aoxborrow/skills
/plugin install aoxborrow-skills@aoxborrow
```

## Claude PR review

An adversarial review that finds bugs, tries to disprove each one, and posts what
survives as one PR review. It runs when a PR opens and on each push (reviewing only
new commits), and on any PR, forks included, when the owner or a collaborator comments
`/claude-review`. Details are at the top of
[the workflow](.github/workflows/claude-review.yml).

To turn it on in a repo:

1. Copy [examples/claude-review.yml](examples/claude-review.yml) to
   `.github/workflows/claude-review.yml`.
2. Set the repository variables `ANTHROPIC_FEDERATION_RULE_ID`,
   `ANTHROPIC_ORGANIZATION_ID`, `ANTHROPIC_SERVICE_ACCOUNT_ID` and
   `ANTHROPIC_WORKSPACE_ID`, from an Anthropic workload identity federation rule
   that trusts GitHub Actions for `pull_request` and `issue_comment` runs.
3. Optionally add `pr-review.md` with what matters most in that project; see
   [examples/pr-review.md](examples/pr-review.md).
