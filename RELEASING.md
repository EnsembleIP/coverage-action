# Releasing `coverage-action`

GitHub Actions references use Git refs. For example, `uses: ensembleip/coverage-action@v1` requires a tag (or branch) named `v1` in this repository.

## Recommended approach (major tag + semver tags)

Create a semver tag for the release, and move the major tag to the same commit:

```bash
git checkout main
git pull

# Create an immutable semver tag for this release
git tag -a v1.0.0 -m "coverage-action v1.0.0"
git push origin v1.0.0

# Create (or move) the floating major tag
git tag -f v1
git push -f origin v1
```

Now downstream repos can use:

- `ensembleip/coverage-action@v1` (tracks latest v1)
- `ensembleip/coverage-action@v1.0.0` (fixed version)
- `ensembleip/coverage-action@<commit SHA>` (most secure pin)


