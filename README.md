# github-ci
Shared GitHub Actions workflows and actions used across Viamedici projects.  
Consume them via `workflow_call` or by referencing actions under `.github/actions/`.

## Build-Bun workflow
Reusable workflow `build-bun.yml` (since v4) to verify, pack and publish an npm package that uses Bun.

### Usage example
```yaml
permissions: {}

jobs:
  build-publish:
    permissions:
      contents: read
      id-token: write
    uses: viamedici-spc/github-ci/.github/workflows/build-bun.yml@v4
    secrets:
      env-secrets: '{"SOME_TEST_TOKEN": ${{ toJSON(secrets.SOME_TEST_TOKEN) }}}'
```

### What it does
- Job `verify` (`contents: read` only): checkout without persisted credentials, GitVersion, Bun from the
  `packageManager` field of the root `package.json`, `bun install --frozen-lockfile`, the verify script, the extra runs,
  then `bun pm pack` with the computed version and an upload of the tarball.
- Job `publish` (`id-token: write`): downloads the tarball and runs `npm publish <tarball> --tag <label> --access public`
  with npm trusted publishing. It runs no dependency code, so nothing the tests execute can mint the publishing token.
- Publishes only on a push or a manual run (`workflow_dispatch`) of `main`/`master`, of a version tag (`4.2.0`,
  `v4.2.0`) or of a branch starting with `feature/`, `release/`, `hotfix/` or `support/`. Every other branch (e.g.
  Dependabot's), every pull request and every other event is verified only.
- The npm dist-tag is the branch's label (`feature/modern-toolchain` → `modern-toolchain`). npm refuses a dist-tag that
  reads as a version, so a run that would publish under one (`feature/v5` → `v5`) fails right after GitVersion, before
  anything is installed; rename the branch.

### Inputs
- `project-dir` (optional, default empty): relative path to the package directory.
- `verify-script` (optional, default `verify`): `package.json` script that type-checks, lints, tests and builds.
- `extra-runs` (optional): shell commands run after the verify script, e.g. the tests against another React version.
- `skip-publish` (optional, default `false`): verify and pack only.

### Secrets
- `env-secrets` (optional): a JSON object whose values are strings, exposed (masked) as environment variables to the
  verify script and the extra runs. Pass it on a single line, as in the example: the runner masks every line of a
  secret, so a line holding only `{` would mask every brace in the log. `toJSON()` quotes the value. Anything other than
  such an object fails the run.

### Outputs
- `package-name`: name from `package.json`.
- `package-version`: version from the `gitversion` action.

## Pages-Bun workflow
Reusable workflow `pages-bun.yml` (since v4) to build a static app with Bun and deploy it to GitHub Pages.

```yaml
on: [push, pull_request, workflow_dispatch]
permissions: {}
jobs:
  pages:
    permissions:
      contents: read
      pages: write
      id-token: write
    uses: viamedici-spc/github-ci/.github/workflows/pages-bun.yml@v4
    with:
      deploy: ${{ github.event_name == 'workflow_dispatch' && github.ref == 'refs/heads/main' }}
```

- Inputs: `verify-script` (default `verify`), `output-dir` (default `dist`), `deploy` (default `false`).
- The build job runs with `contents: read`; only the deploy job holds `pages: write` and `id-token: write`.

## Versioning & references
- This repository uses Semantic Versioning and tags with a `v` prefix (example: `v4.0.0`).
- v4 adds `build-bun.yml` and `pages-bun.yml` and removes `build-npm.yml`; callers that still build with npm stay on
  `@v3` until they move to `build-bun.yml`. The `gitversion` action no longer persists git credentials and writes its
  configuration to `$RUNNER_TEMP` instead of `./GitVersion.yml`.
- Consumers should pin to a full version tag for reproducibility (example: `@v4.0.0`) or use the major alias (example:
  `@v4`) for automatic patch/minor updates.
- When releasing, move the major alias tag (`v4`) to the new `v4.x.y` commit so consumers on `@v4` pick up the latest
  compatible release.
- Also update the internal action reference in `.github/workflows/build-bun.yml` (step: “Checkout and determine package
  version”) to the same release tag; for testing you can point it to a branch like `feature/v4`.
- Third-party actions are pinned by commit SHA with the version as a comment; Dependabot proposes updates once a month
  (`.github/dependabot.yml`).
