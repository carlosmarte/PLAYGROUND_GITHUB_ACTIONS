# npm-publish

A manually-triggered GitHub Actions workflow that bumps a package's version,
runs its lifecycle scripts, publishes to the npm registry, and creates a
matching GitHub Release.

## Install

GitHub only runs workflows stored under `.github/workflows/`. Copy (or symlink)
the workflow there:

```sh
mkdir -p .github/workflows
cp npm-publish/publish.yml .github/workflows/npm-publish.yml
```

## Required secrets

| Secret         | Purpose                                                        |
| -------------- | ------------------------------------------------------------- |
| `NPM_TOKEN`    | npm **automation** token used to authenticate `npm publish`.  |
| `GITHUB_TOKEN` | Provided automatically by Actions; used by `gh` for releases. |

## What it does

1. **Input** — `version` choice: `patch` / `minor` / `major`.
2. Checkout the codebase (full history, for tagging).
3. Install the GitHub CLI (`gh`).
4. Install Node.js (v20).
5. Verify `gh` / Node / npm versions.
6. Create `.npmrc` with the registry + auth token.
7. `npm install`.
8. `npm list` (non-fatal).
9. `npm test` — only if a `test` script exists.
10. `npm run build` — only if a `build` script exists.
11. `npm version <level>` (no git tag yet).
12. `npm publish` using the token, with `--ignore-scripts`.
13. Tag the release and `gh release create` — **`.npmrc` is removed before any
    `git add`, so it is never staged or committed.**

## Security notes

- The `.npmrc` containing the auth token is generated at runtime and deleted
  before the commit/tag step. It is also listed in the repo `.gitignore`.
- `npm publish` runs with `--ignore-scripts` to avoid executing arbitrary
  pre/post-publish scripts from dependencies.
