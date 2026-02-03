# github-ci
Shared GitHub Actions workflows and actions used across Viamedici projects.  
Consume them via `workflow_call` or by referencing actions under `.github/actions/`.

## Build-NPM workflow
Reusable workflow to build/test and publish npm packages.

### Usage example
```yaml
jobs:
  build-publish-npm:
    uses: viamedici-spc/github-ci/.github/workflows/build-npm.yml@v2
    with:
      node-version: '18'
    secrets:
      npm_token: ${{ secrets.NPM_TOKEN }}
```

### Inputs
- `node-version` (required): Node.js version for `actions/setup-node`.
- `build-verbs` (optional, default `test,build`): Comma-separated list of `npm run` scripts.
- `project-dir` (optional, default empty): Relative path to the package directory.
- `skip-publish` (optional, default `false`): Set to `true` to skip `npm publish`.

### Secrets
- `npm_token` (required): npm auth token used for publish.

### Outputs
- `package-name`: Name from `package.json`.
- `package-version`: Version from `gitversion` action.

## Versioning & references
- This repository uses Semantic Versioning and tags with a `v` prefix (example: `v2.0.0`).
- Consumers should pin to a full version tag for reproducibility (example: `@v2.0.0`) or use the major alias (example: `@v2`) for automatic patch/minor updates.
- When releasing, move the major alias tag (`v2`) to the new `v2.x.y` commit so consumers on `@v2` pick up the latest compatible release.
- Also update the internal action reference in `.github/workflows/build-npm.yml` (step: “Checkout and determine package version”) to the same release tag; for testing you can point it to a branch like `feature/v2`.
