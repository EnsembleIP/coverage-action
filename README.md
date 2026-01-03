# Coverage Gate GitHub Action (overall + diff)

This repo provides a reusable **GitHub Action** that:

- **Enforces an overall coverage threshold** from a Cobertura XML report (`coverage.xml`)
- **Optionally enforces a higher “diff coverage” threshold** for files changed in a pull request
- **Optionally publishes results as check-runs** in the PR checks list (so branch protection can block merges)

The underlying logic lives in `scripts/check_coverage.py` and `scripts/check_diff_coverage.py`, wrapped by a composite action (`action.yml`).

## Usage

Add this to your target repository workflow after tests generate `coverage.xml`:

```yaml
name: CI
on:
  pull_request:
    branches: [ main, develop ]
  push:
    branches: [ main, develop ]

permissions:
  contents: read
  checks: write
  pull-requests: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # ... install deps ...
      # ... run tests producing coverage.xml ...

      - name: Enforce coverage
        uses: ensembleip/coverage-action@v1
        with:
          coverage-xml: coverage.xml
          min-coverage: "80"
          min-diff-coverage: "90"   # omit or set to "" to skip diff coverage
          changed-files-glob: "src/**/*.py"
          publish-checks: "true"
```

## Versioning / pinning

`uses: ensembleip/coverage-action@vx` requires that this repository has a **Git tag** named `vx` (or a branch named `vx`).

- For initial testing you can use `@main`.
- For production usage, prefer pinning to a commit SHA, or use a major tag like `@v1` that you keep updated to the latest `v1.x.y`.

## Inputs

- **`coverage-xml`**: Path to the Cobertura XML report (default: `coverage.xml`)
- **`min-coverage`**: Overall coverage threshold (required, 0–100)
- **`min-diff-coverage`**: Diff coverage threshold (optional, 0–100). If empty, diff coverage is skipped.
- **`changed-files-glob`**: Glob used to detect changed files for diff coverage (default: `src/**/*.py`)
- **`publish-checks`**: Publish results into PR checks list (default: `true`)
- **`check-name-overall`**: Check-run name for overall coverage (default: `coverage (overall)`)
- **`check-name-diff`**: Check-run name for diff coverage (default: `coverage (diff)`)
- **`github-token`**: Token used to publish check-runs (default: `${{ github.token }}`)

## Outputs

- **`coverage`**: Overall coverage percentage (min of line and branch coverage)
- **`line_coverage`**: Line coverage percentage
- **`branch_coverage`**: Branch coverage percentage
- **`diff_coverage`**: Diff coverage percentage (if computed)
- **`changed_files_count`**: Number of changed files matched by the glob (PR only)

## Notes

- The action computes changed files using `tj-actions/changed-files` internally (PR only). If your repo structure differs, adjust `changed-files-glob`.
- Your workflow must generate a Cobertura-style XML file (commonly `coverage.xml` from `pytest-cov` with `--cov-report=xml`).

## Local testing (scripts)

You can run the underlying scripts locally:

```bash
python scripts/check_coverage.py coverage.xml 80
python scripts/check_diff_coverage.py coverage.xml changed_files.json 90
```
