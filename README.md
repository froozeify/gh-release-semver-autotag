# Froozeify's GH Release Semver Autotag

A GitHub Action that automatically creates or updates semantic versioning shortcut tags whenever a release is published.

For example, if a release is tagged `v1.2.3`, the action will create/update the tags `v1` and `v1.2`.

---

## Features

- **`semver` mode**: creates clean numeric shortcut tags — `1`, `1.2` — stripping any prefix from the release tag
- **`prefix` mode**: creates prefixed shortcut tags — `v1`, `v1.2` — using the release tag's own prefix
- **`both` mode**: creates all four — `1`, `1.2`, `v1`, `v1.2`
- **Prerelease guard**: skips writing tags for prerelease versions by default — detected via semver suffix (e.g. `v1.2.3-beta.1`) **or** the GitHub release's pre-release flag — but still shows what would have been created in the job summary
- **Dry-run**: preview what would happen without touching any tag
- **Job summary**: writes a table of created/updated tags to the workflow summary

---

## Quick start

No `actions/checkout` step is required — the action only uses the GitHub API to manage tags.

```yaml
on:
  release:
    types: [ published ]

permissions:
  contents: write

jobs:
  autotag:
    runs-on: ubuntu-latest
    steps:
      - uses: froozeify/gh-release-semver-autotag@v1
        # Creates/updates 1 and 1.2 shortcut tags on each release.
```

---

## Inputs

| Input                 | Required | Default               | Description                                                                                                        |
|-----------------------|----------|-----------------------|--------------------------------------------------------------------------------------------------------------------|
| `token`               | no       | `${{ github.token }}` | GitHub token with `contents:write` permission.                                                                     |
| `mode`                | no       | `semver`              | Tagging mode — see [Modes](#modes).                                                                               |
| `tag-prefix`          | no       | `v`                   | Fallback prefix used in `prefix`/`both` mode when the release tag itself has no prefix (e.g. `1.2.3` → `v1`, `v1.2`). |
| `prerelease-strategy` | no       | `skip`                | `skip` to ignore prerelease tags, `include` to update shortcut tags for them too.                                 |
| `dry-run`             | no       | `false`               | Set to `true` to log what would happen without creating or updating any tag.                                      |

## Outputs

| Output         | Description                                                                      |
|----------------|----------------------------------------------------------------------------------|
| `created-tags` | Comma-separated list of tags that were created.                                  |
| `updated-tags` | Comma-separated list of tags that were updated.                                  |
| `skipped`      | `true` if the action was skipped due to a prerelease (tags are listed in the summary but not written). |

---

## Modes

### `semver` (default)

Strips any leading non-numeric prefix from the release tag to produce clean numeric shortcut tags.

| Release tag      | Tags created/updated |
|------------------|----------------------|
| `v1.2.3`         | `1`, `1.2`           |
| `1.2.3`          | `1`, `1.2`           |
| `release-1.2.3`  | `1`, `1.2`           |

```yaml
- uses: froozeify/gh-release-semver-autotag@v1
  with:
    mode: semver  # default, can be omitted
```

### `prefix`

Creates shortcut tags using the prefix detected from the release tag. If the tag has no prefix, falls back to the `tag-prefix` input (`v` by default).

| Release tag      | Tags created/updated         |
|------------------|------------------------------|
| `v1.2.3`         | `v1`, `v1.2`                 |
| `1.2.3`          | `v1`, `v1.2` *(fallback)*    |
| `release-1.2.3`  | `release-1`, `release-1.2`   |

```yaml
- uses: froozeify/gh-release-semver-autotag@v1
  with:
    mode: prefix
```

### `both`

Creates both the prefix-stripped and prefixed shortcut tags.

| Release tag      | Tags created/updated              |
|------------------|-----------------------------------|
| `v1.2.3`         | `1`, `1.2`, `v1`, `v1.2`         |
| `1.2.3`          | `1`, `1.2`, `v1`, `v1.2`         |
| `release-1.2.3`  | `1`, `1.2`, `release-1`, `release-1.2` |

```yaml
- uses: froozeify/gh-release-semver-autotag@v1
  with:
    mode: both
```

---

## Full workflow example

```yaml
name: Release

on:
  release:
    types: [ published ]

permissions:
  contents: write

jobs:
  autotag:
    runs-on: ubuntu-latest
    steps:
      - name: Update semver shortcut tags
        id: autotag
        uses: froozeify/gh-release-semver-autotag@v1
        with:
          mode: both
          prerelease-strategy: skip

      - run: |
          echo "Created : ${{ steps.autotag.outputs.created-tags }}"
          echo "Updated : ${{ steps.autotag.outputs.updated-tags }}"
          echo "Skipped : ${{ steps.autotag.outputs.skipped }}"
```
