# wts-github-hooks

Shared GitHub Actions workflows for wts-developer repos. Automation that
more than one repo needs lives here once, as a reusable workflow;
consuming repos add a small caller and get fixes automatically.

> **Data policy:** everything in this repo is deterministic automation
> on GitHub-managed runners, per the Westminster AI Acceptable Use
> Policy: no AI in the loop, no student records, no PII. Workflows may
> touch only the identifiers explicitly written into PRs (for example
> Asana task ids, Notion page ids). Report concerns to aiethics@wts.edu.

## Catalog

| Workflow | What it does | Secrets |
|---|---|---|
| [`asana-close.yml`](.github/workflows/asana-close.yml) | When a PR merges, completes every Asana task referenced in its body as `closes <asana task URL>` or `closes <bare task id>`; also sets a Status enum custom field to Complete when the task has one, comments the PR link on the task, and rewrites bare ids in the PR body into clickable Asana links. PRs closed without merging leave tasks open. | `ASANA_PAT` |
| [`notion-close.yml`](.github/workflows/notion-close.yml) | When a PR merges, completes every Notion task referenced in its body as `closes <notion task URL>` (a markdown link works too) or `closes <bare page id>`. Completing means whatever the task's database has: Status set to its Complete (or Done) option, a Completed checkbox ticked, an empty Completed at date set to today; nothing is created. Also comments the PR link on the task and rewrites bare page ids in the PR body into Notion links. PRs closed without merging leave tasks open. | `NOTION_TOKEN` |

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

The Notion caller is the same shape: `notion-close.yml` in place of
`asana-close.yml`, and `NOTION_TOKEN: ${{ secrets.NOTION_TOKEN }}` in
place of the Asana secret. Both can run side by side; each ignores the
other's references.

### Secrets

`ASANA_PAT` is an Asana personal access token, ideally from a service
account rather than a personal one. It is managed as a wts-developer
organization secret; when enabling a workflow on a new repo, make sure
the secret's repository access list includes that repo (org Settings,
Secrets and variables, Actions).

`NOTION_TOKEN` is the token of an internal Notion connection (Notion,
Settings, Connections, Develop or manage integrations). Give it the
"Read content", "Update content", and "Insert comments" capabilities,
and share each task database with it (the database's **•••** menu,
Connections). A task in a database it can't see fails with a "not
shared" error in the run log. Same org-secret arrangement as
`ASANA_PAT`.

### Referencing a Notion task

Use **Copy link** on the task in Notion and paste it after `closes`.
A link copied while the task is open in a database's side peek works
too (the task is read from its `p=` parameter). A bare 32-character
page id (the hex at the end of the link) also works and is turned into
a link on merge.

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
