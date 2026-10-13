# broker-manager

[![CI](https://github.com/aktin/broker-manager/actions/workflows/ci.yml/badge.svg)](https://github.com/aktin/broker-manager/actions/workflows/ci.yml)
[![CodeQL](https://github.com/aktin/broker-manager/actions/workflows/codeql.yml/badge.svg)](https://github.com/aktin/broker-manager/actions/workflows/codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/aktin/broker-manager/badge)](https://securityscorecards.dev/viewer/?uri=github.com/aktin/broker-manager)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/aktin/broker-manager)](https://github.com/aktin/broker-manager/releases)

Management software for the AKTIN query broker. A bun workspaces monorepo with a Vue 3 and TypeScript web app under `apps/web`.

## Getting started

You need [bun](https://bun.sh) on PATH (the version is pinned in `.bun-version`) and `python3` for the shared git hooks.

```sh
bun install        # deps + git hooks
bun run dev        # start the web app (apps/web)
bun run test       # unit tests with coverage
bun run typecheck  # vue-tsc
bun run lint       # eslint
bun run format     # prettier
bun run build      # production build
```

The coverage report lands in `apps/web/coverage/index.html`.

To add a backend or a shared package, create `apps/api` or `packages/shared` with its own `package.json`, extend `tsconfig.base.json`, and the root workspace globs pick it up. Root scripts (`build`, `test`, `typecheck`) fan out to every workspace.

## Commits and releases

Commits follow [Conventional Commits](https://www.conventionalcommits.org), enforced by the commit-msg hook:

```
<type>(<optional scope>): <subject>
```

Allowed types are `feat fix docs style refactor perf test chore`. The subject starts lowercase, has no trailing period, and the header stays at most 100 characters. Pull requests are squash merged, so one conventional subject per PR lands on `main`.

A release is a tag `vX.Y.Z` on `main`. The release workflow builds the tagged commit with the version taken from the tag (`package.json` stays at `0.0.0`), writes the release notes from the commit history with git-cliff, adds a CycloneDX SBOM and build attestations, and attaches the web bundle to the GitHub Release.

## Checks and gates

Three layers. Each layer trusts the one before it a little less.

### On your machine

lefthook runs these after `bun install`:

| When       | What runs                                                                                                                                               | Blocks     |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| pre-commit | ESLint fixes the staged files, Prettier formats them, gitleaks scans for secrets, the shared hooks reject data files, large files and oversized commits | the commit |
| commit-msg | Conventional Commit header check                                                                                                                        | the commit |
| pre-push   | `bun run typecheck`, `bun run test` (Vitest with an 80% line coverage threshold), `bun run build`                                                       | the push   |

Hooks can be skipped with `--no-verify`, so nothing here is a guarantee. They exist for fast feedback.

### On a pull request to `main`

| Check             | Workflow                                                                                   | Required to merge                                                                                              |
| ----------------- | ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| `verify / verify` | `ci.yml` calls `verify.yml`: Prettier check, ESLint, typecheck, tests with coverage, build | yes                                                                                                            |
| `review`          | `dependency-review.yml`, fails on new dependencies with high or critical CVEs              | yes                                                                                                            |
| `analyze`         | `codeql.yml`, CodeQL code scanning                                                         | not as a check, but the ruleset blocks the merge on CodeQL errors or security alerts of severity high or above |

### After a merge and on tags

On every push to `main`, `verify / verify` and `analyze` run again on the merged commit and Scorecard scores the repository and publishes the result to OpenSSF. CodeQL and Scorecard also run weekly.

On a tag `vX.Y.Z`, `release.yml` runs verify, builds the bundle, writes the SBOM and attestations and creates the GitHub Release.

### Gates on `main`

Branch ruleset `default`:

| Rule                     | Meaning                                                                                                                       |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| Pull request required    | No direct pushes. Squash merge only, 1 approval, code owner review, approval after the last push, all review threads resolved |
| Required checks          | `verify / verify` and `review` must pass                                                                                      |
| Code scanning            | CodeQL must have analysed the head commit without errors or security alerts of severity high or above                         |
| Signed commits           | Every commit on `main` is signed. Squash merges are signed by GitHub                                                          |
| No force push, no delete | History on `main` is append-only                                                                                              |

Approvals come from people. GitHub Actions cannot create or approve pull requests, auto-merge is off, and no app or bot is on the bypass list. Organisation admins can bypass every rule. Other branches have no rules.

### Gates on tags `v*`

Tag ruleset `release-tags`: only maintainers and admins can create, move or delete release tags. A tag starts the release workflow, so this decides who can cut a release.

## License

Apache-2.0. See [LICENSE](./LICENSE) and [NOTICE](./NOTICE).
