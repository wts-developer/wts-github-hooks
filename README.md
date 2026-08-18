# wts-github-hooks

Shared GitHub Actions workflows for wts-developer repos. Automation that
more than one repo needs lives here once, as a reusable workflow;
consuming repos add a small caller and get fixes automatically.

> **Data policy:** everything in this repo is deterministic automation
> on GitHub-managed runners, per the Westminster AI Acceptable Use
> Policy: no AI in the loop, no student records, no PII. Workflows may
> touch only the identifiers explicitly written into PRs (for example
> Asana task ids). Report concerns to aiethics@wts.edu.

## Catalog

| Workflow | What it does | Secrets |
|---|---|---|
| [`asana-close.yml`](.github/workflows/asana-close.yml) | When a PR merges, completes every Asana task referenced in its body as `closes <asana task URL>` or `closes <bare task id>`; also sets a Status enum custom field to Complete when the task has one, comments the PR link on the task, and rewrites bare ids in the PR body into clickable Asana links. PRs closed without merging leave tasks open. | `ASANA_PAT` |

## Consuming a workflow

Add a caller to the repo's `.github/workflows/`, for example
`asana-close.yml`:

```yaml
name: Close Asana tasks

on:
  pull_request:
    types: [closed]

permissions:
  pull-requests: write

jobs:
  close:
    uses: wts-developer/wts-github-hooks/.github/workflows/asana-close.yml@main
    secrets:
      ASANA_PAT: ${{ secrets.ASANA_PAT }}
```

The `permissions` block matters: a called workflow can never exceed the
caller's permissions, and `asana-close` needs `pull-requests: write` to
linkify task ids in the merged PR body.

### Secrets

`ASANA_PAT` is an Asana personal access token, ideally from a service
account rather than a personal one. It is managed as a wts-developer
organization secret; when enabling a workflow on a new repo, make sure
the secret's repository access list includes that repo (org Settings,
Secrets and variables, Actions).

### Versioning

`@main` gives consuming repos fixes immediately and is the default.
Pin to a commit SHA (`@<sha>`) instead if a repo needs its automation
frozen; there are no version tags yet.

## Contributing

- Branch, PR, human review; do not push to `main` directly after the
  initial setup.
- Keep workflows self-contained and stdlib-only (the embedded scripts
  run on stock runners with no installs).
- A change here ships to every consuming repo on their next trigger, so
  treat edits like production deploys.
- No em-dashes in copy, comments, or commit messages.
