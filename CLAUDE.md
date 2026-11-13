# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of reusable GitHub Actions, currently one composite action: [setup-pnpm/action.yml](setup-pnpm/action.yml) (pnpm/action-setup → actions/setup-node with pnpm cache → `pnpm install --frozen-lockfile`), documented in [setup-pnpm/README.md](setup-pnpm/README.md). [.github/workflows/test.yml](.github/workflows/test.yml) runs it from the current commit across its input combinations; keep the README inputs table, action.yml and that matrix in sync. Other repos consume it as `andykenward/github-actions/setup-pnpm@<ref>`. There is no build or lint step; the only Node tooling is Changesets for versioning.

## Releases (Changesets)

- The repo is a single private package (`github-actions` in [package.json](package.json)); its version is the version of every action here. [.changeset/config.json](.changeset/config.json) sets `privatePackages.tag: true`, so a release creates a git tag rather than publishing to npm.
- Any change consumers would notice needs a changeset: `pnpm changeset` writes a file in `.changeset/`. Past practice in [CHANGELOG.md](CHANGELOG.md): dependency bumps of actions inside `setup-pnpm` are **patch**; Node.js major changes and upgrades to all actions are **major**.
- [.github/workflows/release.yml](.github/workflows/release.yml) runs on push to `main`. It uses the in-repo `$/setup-pnpm` action (self-repository syntax: same commit, no checkout needed), then `changesets/action` either opens the "Version Packages" PR or runs `pnpm release` (`changeset publish`) to tag.

## Conventions

- [.github/workflows/zizmor.yml](.github/workflows/zizmor.yml) audits `.github` and `setup-pnpm` on PRs and pushes to `main`, and uploads the results to code scanning. Workflows set `permissions: {}` at the top and grant only what each job needs, and checkouts use `persist-credentials: false`.
- Pin every third-party action to a full commit SHA with a trailing `#vX.Y.Z` comment (e.g. `uses: actions/checkout@<sha> #v7.0.1`). Dependabot updates these.
- YAML files start with a `# yaml-language-server: $schema=...` line pointing at the matching schemastore schema.
- Node 24 is the target everywhere: `engines.node` and `devEngines.runtime.version` (same range), the `setup-pnpm` default `node-version` and the release workflow. Keep them in sync.
- pnpm is pinned by `packageManager` in package.json.
- The dev container uses the `ghcr.io/andykenward/devcontainer-dev` image, pinned by tag and digest in [.devcontainer/Dockerfile](.devcontainer/Dockerfile) so Dependabot bumps it. mise ([mise.toml](mise.toml)) installs Node and pnpm from package.json, and [mise.lock](mise.lock) pins their versions and checksums. After any change to `packageManager` or the Node range, including Dependabot bumps, run `mise lock` and commit `mise.lock` in the same PR.
- Commit messages follow Conventional Commits (`chore:`, `chore(deps):`, …).

## Dependency updates

[.github/dependabot.yml](.github/dependabot.yml) covers github-actions (`/` and `/setup-pnpm`; `/` scans only `.github/workflows`), npm, and the dev container image in `.devcontainer/Dockerfile`. Every ecosystem is weekly with a 7-day cooldown.

[pnpm-workspace.yaml](pnpm-workspace.yaml) settings that matter:
- `minimumReleaseAge: 10080` (7 days). pnpm refuses to install newer versions, so the Dependabot cooldown must stay at 7 days or more.
- `pmOnFail: ignore` keeps `pnpm-lock.yaml` a single YAML document so GitHub's dependency graph can parse it (dependabot-core#15904). Don't remove it.
