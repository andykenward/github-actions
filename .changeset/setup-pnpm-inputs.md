---
"github-actions": minor
---

setup-pnpm: add `cache`, `node-version-file`, `pnpm-version` and `working-directory` inputs and `cache-hit` / `node-version` outputs. `pnpm install` now passes `--frozen-lockfile` explicitly; this is already pnpm's default on GitHub-hosted runners, and now also applies on runners that don't set `CI`.
