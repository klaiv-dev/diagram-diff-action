# Klaiv Diagram Diff Action

Render before/after [Klaiv](https://github.com/klaiv-dev) diagram previews on pull requests.

When a PR changes your model, the action renders the affected diagrams and posts a
comment showing each one **before and after**, so reviewers see the architectural
change visually instead of reading raw YAML.

## Quickstart

Add `.github/workflows/klaiv-diagram-diff.yml` to your Klaiv repo:

```yaml
name: Klaiv Diagram Diff
on:
  pull_request:
    types: [opened, synchronize, reopened]
    paths: ['architecture/**.yaml']
permissions:
  contents: write        # push rendered SVGs to the previews branch
  pull-requests: write   # post the diff comment
jobs:
  diff:
    runs-on: ubuntu-latest
    steps:
      - uses: klaiv-dev/diagram-diff-action@v1
        with:
          token: ${{ github.token }}
```

That's the whole setup. Open a PR that edits a diagram and the comment appears.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `token` | no | — | GitHub token used to read the repo and post the comment. Pass `token: ${{ github.token }}` — the built-in `GITHUB_TOKEN`. |
| `architecture-dir` | no | `architecture` | Root directory of your Klaiv model. Change this if your model lives elsewhere. |
| `previews-branch` | no | `klaiv-previews` | Orphan branch where rendered SVGs are stored so before/after images have a stable URL. Created automatically. |
| `max-diagrams` | no | `5` | Maximum number of diagrams rendered per comment. |
| `footer-url` | no | Klaiv repo | URL used in the comment footer. |

## Requirements

- **`permissions:` block is required.** Without `contents: write` the action can't
  store the rendered SVGs; without `pull-requests: write` it can't post the comment.
  The snippet above sets both.
- **`token: ${{ github.token }}` must be passed explicitly.** The action's default
  does not evaluate the expression for you.
- The `paths:` filter is optional but recommended — it skips the run on PRs that
  don't touch the model. Adjust the path if you set a custom `architecture-dir`.

## What it creates

- A `klaiv-previews` orphan branch holding the rendered SVGs. This is machine-managed
  history; you don't edit it.
- One PR comment per run, updated in place on subsequent pushes.
