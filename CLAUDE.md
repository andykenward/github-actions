# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of reusable GitHub Actions, currently one composite action: [setup-pnpm/action.yml](setup-pnpm/action.yml) (pnpm/action-setup → actions/setup-node with pnpm cache → `pnpm install`). Other repos consume it as `andykenward/github-actions/setup-pnpm@<ref>`. There is no build, lint or test step; the only Node tooling is Changesets for versioning.

## Releases (Changesets)

- The repo is a single private package (`github-actions` in [package.json](package.json)); its version is the version of every action here. [.changeset/config.json](.changeset/config.json) sets `privatePackages.tag: true`, so a release creates a git tag rather than publishing to npm.
- Any change consumers would notice needs a changeset: `pnpm changeset` writes a file in `.changeset/`. Past practice in [CHANGELOG.md](CHANGELOG.md): dependency bumps of actions inside `setup-pnpm` are **patch**; Node.js major changes and upgrades to all actions are **major**.
- [.github/workflows/release.yml](.github/workflows/release.yml) runs on push to `main`. It uses the local `./setup-pnpm` action, then `changesets/action` either opens the "Version Packages" PR or runs `pnpm release` (`changeset publish`) to tag.

## Conventions

- [.github/workflows/zizmor.yml](.github/workflows/zizmor.yml) audits `.github` and `setup-pnpm` on PRs and pushes to `main`, and uploads the results to code scanning. Workflows set `permissions: {}` at the top and grant only what each job needs, and checkouts use `persist-credentials: false`.
- Pin every third-party action to a full commit SHA with a trailing `#vX.Y.Z` comment (e.g. `uses: actions/checkout@<sha> #v7.0.1`). Dependabot updates these.
- YAML files start with a `# yaml-language-server: $schema=...` line pointing at the matching schemastore schema.
- Node 24 is the target everywhere: `engines.node`, the `setup-pnpm` default `node-version`, the release workflow and the dev container. Keep them in sync.
- pnpm is pinned by `packageManager` in package.json, and the dev container's node feature `pnpmVersion` should match it.
- Commit messages follow Conventional Commits (`chore:`, `chore(deps):`, …).

## Dependency updates

[.github/dependabot.yml](.github/dependabot.yml) covers github-actions (`/` and `/setup-pnpm`; `/` scans only `.github/workflows`), npm, devcontainers features, and the docker base image in `.devcontainer/Dockerfile`. Every ecosystem is weekly with a 7-day cooldown.

[pnpm-workspace.yaml](pnpm-workspace.yaml) settings that matter:
- `minimumReleaseAge: 10080` (7 days). pnpm refuses to install newer versions, so the Dependabot cooldown must stay at 7 days or more.
- `pmOnFail: ignore` keeps `pnpm-lock.yaml` a single YAML document so GitHub's dependency graph can parse it (dependabot-core#15904). Don't remove it.
