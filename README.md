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

## Build-NPM workflow
Reusable workflow to build/test and publish npm packages.

### Usage example
```yaml
jobs:
  build-publish-npm:
    permissions:
      contents: read
      id-token: write
    uses: viamedici-spc/github-ci/.github/workflows/build-npm.yml@v3
```

### Inputs
- `node-version` (optional, default `24`): Node.js version for build and publish via `actions/setup-node`.
- `build-verbs` (optional, default `test,build`): Comma-separated list of `npm run` scripts.
- `project-dir` (optional, default empty): Relative path to the package directory.
- `skip-publish` (optional, default `false`): Set to `true` to skip `npm publish`.

### Secrets
- `npmrun-secrets` (optional): Flat JSON object exposed as environment variables while running the configured `npm run` scripts.

### Trusted publishing
- `build-npm.yml` now publishes to npm via GitHub Actions OIDC / npm trusted publishing. No `NPM_TOKEN` secret is required for `npm publish`.
- The caller workflow should grant `id-token: write`. `contents: read` is sufficient for this reusable workflow unless the caller needs more.
- Configure the npm trusted publisher against the consuming repository and its calling workflow file. For `workflow_call`-based publishes, npm validates the caller workflow identity.

### Outputs
- `package-name`: Name from `package.json`.
- `package-version`: Version from `gitversion` action.

## Versioning & references
- This repository uses Semantic Versioning and tags with a `v` prefix (example: `v4.0.0`). v4 adds `build-bun.yml`;
  `build-npm.yml` is unchanged from v3.
- Consumers should pin to a full version tag for reproducibility (example: `@v3.0.0`) or use the major alias (example: `@v3`) for automatic patch/minor updates.
- When releasing, move the major alias tag (`v3`) to the new `v3.x.y` commit so consumers on `@v3` pick up the latest compatible release.
- Also update the internal action reference in `.github/workflows/build-bun.yml` and `build-npm.yml` (step: “Checkout and determine package version”) to the same release tag; for testing you can point it to a branch like `feature/v4`.
- Third-party actions are pinned by commit SHA with the version as a comment; Dependabot proposes updates once a month
  (`.github/dependabot.yml`).
