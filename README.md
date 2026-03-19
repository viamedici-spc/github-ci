# github-ci
Shared GitHub Actions workflows and actions used across Viamedici projects.  
Consume them via `workflow_call` or by referencing actions under `.github/actions/`.

## Build-NPM workflow
Reusable workflow to build/test and publish npm packages.

### Usage example
```yaml
jobs:
  build-publish-npm:
    permissions:
      contents: read
      id-token: write
    uses: viamedici-spc/github-ci/.github/workflows/build-npm.yml@v2
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
- This repository uses Semantic Versioning and tags with a `v` prefix (example: `v2.0.0`).
- Consumers should pin to a full version tag for reproducibility (example: `@v2.0.0`) or use the major alias (example: `@v2`) for automatic patch/minor updates.
- When releasing, move the major alias tag (`v2`) to the new `v2.x.y` commit so consumers on `@v2` pick up the latest compatible release.
- Also update the internal action reference in `.github/workflows/build-npm.yml` (step: “Checkout and determine package version”) to the same release tag; for testing you can point it to a branch like `feature/v2`.
