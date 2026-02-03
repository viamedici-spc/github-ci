# Repository Guidelines

## Project Structure & Module Organization
- Root contains documentation and GitHub CI assets. Currently the only tracked file is `README.md`.
- Add reusable workflows under `.github/workflows/` and composite or JavaScript actions under `.github/actions/` (recommended locations for this repo’s purpose).

## Versioning & References
- Use SemVer with `v`-prefixed tags (example: `v2.0.0`) and keep the major alias (`v2`) moved to the latest `v2.x.y` release.
- The reusable workflow `build-npm.yml` references the internal action `.github/actions/gitversion` by ref; update that ref when releasing or testing (example: `@feature/v2` for tests, `@v2` for releases).

## Coding Style & Naming Conventions
- Use 2-space indentation for YAML files and keep workflow/action IDs in `kebab-case` (example: `deploy-preview`).
- Use clear, action-oriented names for workflows (example: `ci-pr.yml`) and actions (example: `setup-toolchain`).
- If you introduce scripting (bash, node, etc.), add a formatter or linter and note it here.

## GitVersion Configuration Notes
- GitVersion v6+ is required by the current GitTools Actions; the config in `.github/actions/gitversion/action.yml` follows the v6 schema.
- Feature branch labels come from a named regex group (`BranchName`) so versions reflect the branch name.

## Security & Configuration Tips
- Avoid hard-coding secrets; use GitHub Actions secrets or environment variables.
- Prefer least-privilege permissions in workflows (`permissions:` blocks) and document any elevated needs.

## Agent-Specific Instructions
- When proposing changes, keep the repo lightweight and document new commands or conventions in this file.
