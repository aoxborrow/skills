# skills

My Claude Code skills, plus a reusable Claude PR review workflow I use across my repos.

## Skills

Skills live in `skills/<name>/SKILL.md`. Install them in Claude Code:

```
/plugin marketplace add aoxborrow/skills
/plugin install skills@aoxborrow
```

## Claude PR review

An adversarial review that finds bugs, tries to disprove each one, and posts what
survives as one PR review. It runs when a PR opens and on each push (reviewing only
new commits), and on any PR, forks included, when the owner or a collaborator comments
`/claude-review`. A push review with nothing new to report posts nothing; a
`claude-review` commit status records that the commit was reviewed.
How it works is at the top of [reusable-claude-review.yml](.github/workflows/reusable-claude-review.yml).

To turn it on in a repo:

1. Copy [claude-review.yml](.github/workflows/claude-review.yml) to the repo's
   `.github/workflows/`.
2. Give Claude a way to sign in, one of:
   - **A Claude subscription** (billed to the plan, not API credits): run
     `claude setup-token` and save the token as the repository secret
     `CLAUDE_CODE_OAUTH_TOKEN`. This wins when both are set.
   - **API billing**: set the repository variables `ANTHROPIC_FEDERATION_RULE_ID`,
     `ANTHROPIC_ORGANIZATION_ID`, `ANTHROPIC_SERVICE_ACCOUNT_ID` and
     `ANTHROPIC_WORKSPACE_ID`, from an Anthropic workload identity federation rule
     that trusts GitHub Actions for `pull_request` and `issue_comment` runs.
3. Optionally add `.github/pr-review.md` saying what matters most in that project,
   like [this repo's](.github/pr-review.md).
