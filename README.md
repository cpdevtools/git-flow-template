# git-flow Workspace Template

A generic pnpm monorepo template pre-wired for the [cpdevtools/git-flow](https://github.com/cpdevtools/git-flow) release workflow.

## What's included

Everything workspace-level (nothing package-specific):

| File / Dir                                 | Purpose                                                                                    |
| ------------------------------------------ | ------------------------------------------------------------------------------------------ |
| `package.json`                             | Workspace root — `0.0.0-MAIN` version placeholder, wireit orchestration, shared dev deps    |
| `pnpm-workspace.yaml`                      | Declares `packages/*` as workspace packages                                                 |
| `tsconfig.json`                            | Base TypeScript config (packages extend or provide their own)                               |
| `.publish/versions.yml`                    | Maps release-group placeholders (`0.0.0-MAIN`) to current versions — bumped by git-flow     |
| `.publish/deps.yml`                        | Pinned dependency versions enforced by `pnpm check` / `pnpm fix`                            |
| `.publish/registries.yml`                  | Publish registries referenced by each package's `release-artifacts.yml`                     |
| `.publish/registries.example.yml`          | Examples for all supported registry types (npm, nuget, docker)                              |
| `.github/workflows/test.yml`               | Runs tests on every push (non-release branches)                                             |
| `.github/workflows/create-release-pr.yml`  | Opens/updates a release PR on push                                                          |
| `.github/workflows/build-pack-publish.yml` | Builds, packs, and publishes when a release PR merges                                       |
| `.github/workflows/cleanup-scheduled.yml`  | Daily cleanup of old build releases, their tags, and the registry versions they published   |
| `.github/workflows/cleanup-deleted-branch.yml` | Deletes a branch's `release/<branch>` counterpart when the branch itself is deleted (`on: delete`; only fires from the default branch's copy — see git-flow's `actions/cleanup-deleted-branch`) |
| `.github/workflows/deploy-production.yml.example` | Example per-environment deploy workflow (rename to `deploy-{env}.yml` to enable)     |
| `.publish/dev-local.yml.example`           | dev-link map: local checkout paths for `devutil dev-link` (rename to `dev-local.yml` to enable) |
| `.husky/pre-commit`                        | Guards against `file:/devcontainer` paths leaking into the lockfile or manifests            |
| `.npmrc`                                   | Scope → registry mapping + GitHub Packages auth via `GITHUB_TOKEN`                          |
| `.syncpackrc.yml`                          | Version consistency rules (`workspace:*` for local packages, caret ranges, key sort order)  |
| `.prettierrc` / `.prettierignore`          | Formatting config (packages format themselves via `devutil run format`)                     |
| `.gitignore`                               | Ignores build output                                                                        |

## Getting started

1. Copy this template to a new repo.
2. Search & replace the placeholders:
   - `@my-scope` → your npm scope (e.g. `@cpdevtools`)
   - `my-org` → your GitHub org / docker namespace
   - `my-project` → your repo name
   - `My Org` → author name
3. Add packages under `packages/<name>/` with:
   - `"version": "0.0.0-MAIN"` in `package.json` (the release-group placeholder)
   - a `release-artifacts.yml` describing artifacts to publish (npm / docker) and target registries
4. Set the initial version in `.publish/versions.yml`.
5. `pnpm install && pnpm build`

## Deploying

`gitflow deploy` discovers per-environment workflows by convention: `.github/workflows/deploy-{env}.yml`.

To enable deploys:

1. Rename `deploy-production.yml.example` → `deploy-production.yml` (or copy per environment, e.g. `deploy-staging.yml`), updating the workflow name and `environment:` values to match.
2. Create the matching GitHub Environment (Settings → Environments) and configure:
   - `vars.DEPLOY_URL` — URL of the git-flow deploy gateway/service
   - `secrets.DEPLOY_HMAC_SECRET` — HMAC secret shared with the deploy service
   - `vars.DEPLOY_TYPE_DEFAULT` — optional default deploy method (`node`, `compose`, `swarm`)
   - `vars.DEPLOY_ENV` — optional `KEY=VAL` lines passed to every deploy
   - `vars.DEPLOY_ALLOWED_METHODS` — optional comma-separated allowed methods
3. Declare deployable artifacts in the package's `release-artifacts.yml` (docker artifacts with a `deploy:` section).

## Versioning model

- Package versions use the placeholder `0.0.0-MAIN`; git-flow substitutes the real version at release time.
- `.publish/versions.yml` tracks the current version per release group.
- Releases are tagged `{packageName}/v{version}` plus group tags `MAIN/v{version}` and `v{version}`
  (the package/group name comes first — a plain `v{version}` tag would otherwise collide with it as
  a git ref path, blocking both that tag and every `v*` alias).

## Branch model

`main` (no slash) is the **current** major — it publishes ordinary stable releases and stays that
way indefinitely; there is no LTS branch to create at setup time. If a next-major rewrite starts
before the current major is done, it lives on a `feature/<name>` branch: the slash makes it a
git-flow *development line*, so it can only ever publish prereleases
(`{nextMajor}.0.0-feature.<name>.alpha.N`), never a stable release ahead of `main`. Keep it current
by merging `main` into it periodically (not the other way round), and re-verify with a full build
after each merge, not just "no conflicts."

Only **at graduation** — when the feature branch is ready to replace `main` — cut an LTS branch
(`v{oldMajor}`) from `main`'s last commit, then merge the feature branch into `main` and bump
`.publish/versions.yml` to the new major's first stable version. Don't pre-create the LTS branch or
bump `main` ahead of that point; it was tried, caused real churn, and was reverted.

## Scripts

| Script           | What it does                                                       |
| ---------------- | ------------------------------------------------------------------ |
| `pnpm build`     | Build all packages (`devutil run build`)                            |
| `pnpm test`      | Test all packages                                                   |
| `pnpm typecheck` | Typecheck all packages                                              |
| `pnpm lint`      | Lint all packages                                                   |
| `pnpm format`    | syncpack format → prettier → per-package formatting                 |
| `pnpm check`     | Verify dep versions (`.publish/deps.yml`) and syncpack consistency  |
| `pnpm fix`       | Fix dep versions, mismatches, remove local tags, format             |
| `pnpm clean`     | `git clean -dfX` + reinstall                                        |
| `pnpm reset`     | `git reset --hard` + reinstall                                      |