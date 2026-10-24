# dsh-bot

Two GitHub Actions workflows that turn a repository into one an agent works in.
Everything runs on GitHub-hosted runners in throwaway containers — nothing runs on
your machine, and each run dies with its job.

| Workflow | Trigger | Who can trigger | Permissions |
| --- | --- | --- | --- |
| `dsh.yml` | comment `/dsh <task>` on an issue **or** a pull request | only OWNER / MEMBER / COLLABORATOR | `contents: write`, `issues: write`, `pull-requests: write` |
| `triage.yml` | an issue is opened | anyone | `contents: read`, `issues: write` |

The split is deliberate. `triage.yml` runs on text written by strangers, so its
token physically cannot write code — no amount of prompt injection changes that.
`dsh.yml` can write code, so only trusted people can start it.

## What each one does

**`/dsh` command** — reads the issue or PR, makes the change on a `dsh/...` branch,
opens a pull request, and replies in the thread. On a pull request it checks out
the PR's head branch and commits there instead. Fork pull requests are skipped.

**Issue triage** — labels a newly opened issue from a fixed set and posts a short
comment restating the problem plus at most one clarifying question.

## Setup

1. Add the secret **`DEEPSEEK_API_KEY`** (Settings → Secrets and variables → Actions).
   An API key is required; an account login does not work in CI.
2. Protect the default branch (public repo, or GitHub Pro for private ones):
   require pull requests, disallow force pushes, disallow deletions, enforce on admins.
3. Create the labels triage is allowed to use:

   | label | colour |
   | --- | --- |
   | `bug` | `d73a4a` |
   | `enhancement` | `a2eeef` |
   | `question` | `d876e3` |
   | `documentation` | `0075ca` |
   | `needs-info` | `fbca04` |
   | `good first issue` | `7057ff` |

4. Comment `/dsh reply with just OK` on a test issue to confirm the loop works.

## Using it on other repositories

The workflow only ever acts on the repository it lives in, so each repository
needs its own copy of `.github/workflows/` + `scripts/` and its own
`DEEPSEEK_API_KEY` secret. Copy the four files, run the setup above, done.

## Known unknowns

- **First run cost**: roughly 1–2 minutes per task (npm install dominates).
- **Sandbox**: `dsh` may pick a bubblewrap bash sandbox on Linux; if it fails
  inside the runner, switch the profile to the plain local bash tool.
- **Label drift**: triage is told to use only the labels above. If you delete one,
  update `scripts/run-triage.sh` or the agent will fail to apply it.
