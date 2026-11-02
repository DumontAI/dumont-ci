# dumont-ci

Reusable GitHub Actions workflows shared by all Dumont product repos. One copy of each scanner, called from every repo. Do not copy the shell into a product repo.

## Workflows

| Workflow | Use |
|---|---|
| `dependency-scan.yml` | Vulnerability scan of lockfiles with osv-scanner. Detects pnpm, bun, npm, and requirements files with no per-repo config. |
| `secret-scan.yml` | gitleaks over the PR diff. Uses the free binary, not the licensed GitHub Action. |
| `knip.yml` | Unused files, exports, and dependencies for JS/TS repos. Report only unless `strict: true`. |

All three default to report mode. Nothing blocks a merge until its baseline is clean and `fail-on` or `strict` is raised deliberately.

## Callers

Pin the tag, not `main`, so a change here cannot silently change every product's CI:

```yaml
# .github/workflows/dependency-scan.yml
name: Dependency scan
on:
  schedule:
    - cron: "17 4 * * *"
  pull_request:
    paths:
      - "pnpm-lock.yaml"
  workflow_dispatch:
jobs:
  scan:
    uses: DumontAI/dumont-ci/.github/workflows/dependency-scan.yml@v1
    with:
      fail-on: none
```

```yaml
# .github/workflows/secret-scan.yml
name: Secret scan
on:
  pull_request:
    types: [opened, synchronize, reopened]
jobs:
  scan:
    uses: DumontAI/dumont-ci/.github/workflows/secret-scan.yml@v1
```

```yaml
# .github/workflows/knip.yml
name: Knip
on:
  schedule:
    - cron: "41 4 * * 1"
  workflow_dispatch:
jobs:
  knip:
    uses: DumontAI/dumont-ci/.github/workflows/knip.yml@v1
    with:
      strict: false
```

Bun repo: pass `install-command: bun install --frozen-lockfile`. To skip install, pass `install-command: none`.
Self-hosted runner: pass `runs-on: <single label>`.

## Required org setting

The calling repo must be allowed to use workflows from this repo: Settings > Actions > General > Access, or the org-level equivalent, set to "repositories in the organization".

## Rolling out a gate

1. Run in report mode and read the artifact.
2. Triage the baseline. Fix, or ignore with a written reason.
3. Raise `fail-on` to `high` (osv) or `strict: true` (knip) only after that.
4. Never gate a whole monorepo on day one. For knip, gate changed packages.

## Versions

Pinned in the workflow inputs: osv-scanner `v2.7.0`, gitleaks `8.30.1`, knip `6.44.1`. Bump all three here and tag `v2` when needed; callers do not change.

## Local check

`selftest.yml` calls all three workflows on this repo on every push to `main`. The `fixture/` package exists to give osv-scanner a lockfile and knip something to report.
