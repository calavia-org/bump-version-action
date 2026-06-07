# Bump Version Action

A GitHub Action that bumps the version in a repository based on semantic versioning. Auto-detects the technology from common version files.

## Supported Technologies

- **Ansible**: `galaxy.yml`
- **Node.js**: `package.json`
- **Python**: `pyproject.toml`, `setup.py`
- **Rust**: `Cargo.toml`
- **Generic**: `VERSION`, `version.txt`

## Usage

```yaml
- uses: calavia-org/bump-version-action@v1
  with:
    version-file: auto          # or explicit path like 'galaxy.yml'
    bump-type: patch            # major, minor, or patch
    current-version: '1.2.3'    # optional, will read from file if not provided
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version-file` | Path to version file or `auto` for detection | No | `auto` |
| `bump-type` | Type of bump: `major`, `minor`, `patch` | Yes | - |
| `current-version` | Current version string | No | Read from file |

## Outputs

| Output | Description |
|--------|-------------|
| `new-version` | The new version after bumping |
| `version-file` | The detected or provided version file path |
| `technology` | The detected technology |

## Example

```yaml
jobs:
  bump:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: calavia-org/bump-version-action@v1
        id: bump
        with:
          version-file: auto
          bump-type: minor
      - run: |
          echo "New version: ${{ steps.bump.outputs.new-version }}"
```