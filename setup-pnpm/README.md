# setup-pnpm

Composite action that sets up Node and pnpm, caches the pnpm store keyed on `pnpm-lock.yaml`, then runs `pnpm install --frozen-lockfile`.

## Usage

Pin to a full commit SHA, with the release tag as a comment so Dependabot can update it:

```yaml
steps:
  - uses: actions/checkout@<sha> # v7.0.1
    with:
      persist-credentials: false
  - uses: andykenward/github-actions/setup-pnpm@<sha> # v4.0.0
```

The pnpm version comes from the `packageManager` field in `package.json`. Use `pnpm-version` only for projects without that field; `pnpm/action-setup` fails if both are set and differ.

### Release and publish jobs

Turn off caching so a cache entry written by a less trusted workflow can't end up in a released build:

```yaml
  - uses: andykenward/github-actions/setup-pnpm@<sha> # v4.0.0
    with:
      cache: false
```

## Inputs

| Input               | Default                                    | Description                                                                                 |
| ------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------- |
| `node-version`      | `24.x` (unless `node-version-file` is set) | Version of Node.                                                                            |
| `node-version-file` |                                            | File to read the Node version from (`.nvmrc`, `.node-version`, `package.json`), relative to `working-directory`. |
| `pnpm-version`      |                                            | Version of pnpm, for projects without `packageManager` in `package.json`.                   |
| `cache`             | `true`                                     | Cache the pnpm store.                                                                       |
| `working-directory` | `.`                                        | Directory containing `package.json` and `pnpm-lock.yaml`, relative to the repository root.  |

## Outputs

| Output         | Description                                      |
| -------------- | ------------------------------------------------ |
| `cache-hit`    | Whether the pnpm store was restored from the cache. |
| `node-version` | The installed Node version.                      |
