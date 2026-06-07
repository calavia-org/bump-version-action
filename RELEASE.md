# Release Guide: Bump Version Action

This document describes how to release new versions of the `bump-version-action` repository.

## What This Repository Is

A **GitHub Composite Action** that bumps version files across different technologies:
- Ansible (`galaxy.yml`)
- Node.js (`package.json`)
- Python (`pyproject.toml`, `setup.py`)
- Rust (`Cargo.toml`)
- Generic (`VERSION`, `version.txt`)

It is consumed by referencing a git tag (e.g., `@v1`, `@v1.2.3`).

## Who Uses This Action

| Consumer | Repository | How It's Used |
|----------|------------|---------------|
| PR Check Workflow | [`calavia-org/workflows-lib`](https://github.com/calavia-org/workflows-lib) | Called inside `pr-check-and-bump.yml` reusable workflow |
| Direct usage | Any org repo | Can be used standalone in any workflow |

## Release Steps

### 1. Merge Changes to `main`

```bash
git checkout main
git pull origin main
```

### 2. Create a Semantic Version Tag

```bash
# For a new feature (minor bump)
git tag -a v1.1.0 -m "Release v1.1.0: add support for Go modules"

# For a fix (patch bump)
git tag -a v1.0.1 -m "Release v1.0.1: fix Python version regex"

# For breaking changes (major bump)
git tag -a v2.0.0 -m "Release v2.0.0: change input names"
```

### 3. Push the Tag

```bash
git push origin v1.1.0
```

### 4. Update the Floating Major Version Tag

```bash
# Delete old v1 tag locally and remotely
git tag -d v1
git push origin :refs/tags/v1

# Recreate v1 pointing to the new release
git tag -a v1 -m "Update v1 to v1.1.0"
git push origin v1
```

This allows consumers to use `@v1` and automatically get non-breaking updates.

## Versioning Strategy

| Version | Meaning | Consumer Impact |
|---------|---------|----------------|
| `v1.0.0` | Initial release | Pin to specific version |
| `v1.1.0` | New feature, backward compatible | Consumers using `@v1` get it automatically |
| `v1.1.1` | Bug fix | Consumers using `@v1` get it automatically |
| `v2.0.0` | Breaking change | Consumers must manually update from `@v1` to `@v2` |

## What Consumers Reference

```yaml
# Option 1: Floating major version (recommended for org-internal actions)
- uses: calavia-org/bump-version-action@v1

# Option 2: Specific version (for reproducibility)
- uses: calavia-org/bump-version-action@v1.1.0

# Option 3: Commit SHA (most reproducible, but harder to maintain)
- uses: calavia-org/bump-version-action@abc1234
```

## Release Checklist

- [ ] All changes tested with a sample repo
- [ ] README.md updated with new features
- [ ] `action.yml` inputs/outputs documented
- [ ] Version tag created (`vX.Y.Z`)
- [ ] Floating major tag updated (`vX`)
- [ ] Release notes published (GitHub Releases)

## Post-Release Verification

1. Check that `calavia-org/bump-version-action@v1` resolves to the new tag
2. Verify a consumer workflow (e.g., in `workflows-lib`) picks up the change
3. Test with a PR in a consumer repo to confirm version bumping still works

## Automation Options

### Option A: Manual Tagging (Current)
Create tags locally and push them.

### Option B: GitHub Actions Auto-Release
Add a workflow that creates tags from conventional commits:

```yaml
name: Release Action

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Bump version and push tag
        id: tag
        uses: mathieudutour/github-tag-action@v6.1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          default_bump: patch

      - name: Update major version tag
        if: steps.tag.outputs.new_tag != ''
        run: |
          MAJOR=$(echo ${{ steps.tag.outputs.new_tag }} | cut -d. -f1)
          git tag -d $MAJOR || true
          git push origin :refs/tags/$MAJOR || true
          git tag -a $MAJOR -m "Update $MAJOR to ${{ steps.tag.outputs.new_tag }}"
          git push origin $MAJOR
```

### Option C: Semantic Release
Use `semantic-release` with conventional commits for fully automated versioning.

---

*For the release process of the reusable workflow that consumes this action, see:*
- [`workflows-lib/RELEASE.md`](https://github.com/calavia-org/workflows-lib/blob/main/RELEASE.md)

*For the release process of the consumer collection, see:*
- [`ansible-collection-setup/RELEASE.md`](https://github.com/calavia-org/ansible-collection-setup/blob/main/RELEASE.md)
