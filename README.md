# broker-manager

[![CI](https://github.com/aktin/broker-manager/actions/workflows/ci.yml/badge.svg)](https://github.com/aktin/broker-manager/actions/workflows/ci.yml)
[![CodeQL](https://github.com/aktin/broker-manager/actions/workflows/codeql.yml/badge.svg)](https://github.com/aktin/broker-manager/actions/workflows/codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/aktin/broker-manager/badge)](https://securityscorecards.dev/viewer/?uri=github.com/aktin/broker-manager)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/aktin/broker-manager)](https://github.com/aktin/broker-manager/releases)

# Management software for the AKTIN query broker

## Stack

bun workspaces monorepo, Vue 3 + TypeScript (Vite), Vitest, ESLint (flat) + Prettier, lefthook with shared hooks from aktin-github-scripts, tag-driven releases via git-cliff, and a hardened GitHub Actions pipeline (CI, CodeQL, Scorecard, dependency review, SBOM, build attestations).

## Getting started

Prerequisites: [bun](https://bun.sh) on PATH (the version is pinned in `.bun-version`). The shared git hooks are Python scripts and need `python3` on PATH.

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

Commits must follow [Conventional Commits](https://www.conventionalcommits.org), enforced by the commit-msg hook:

```
<type>(<optional scope>): <subject>
```

Allowed types are `feat fix docs style refactor perf test chore`, the subject starts lowercase and has no trailing period, the header stays at most 100 characters. Cut a release by pushing a `vX.Y.Z` tag.

## Repository settings

Generating a repo from a template copies files and branches only, never settings, so these one-time steps stay manual. Do them after `bun run init`.

| Setting                           | Where and what to do                                                                                                                 | Why                                                                                                                      |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| Branch ruleset on `main`          | Settings → Rules → Rulesets. New branch ruleset targeting `main`: require a pull request, require the `CI` check, block force pushes | Keeps unreviewed or broken code off `main`                                                                               |
| Tag ruleset on `v*`               | Same screen, new tag ruleset targeting `v*`, restrict creation to maintainers                                                        | Releases are tag-driven, without the rule anyone with push access can cut one                                            |
| Workflow permissions              | Settings → Actions → General → Read and write                                                                                        | The release workflow creates the GitHub Release and uploads the attestations                                             |
| Fork PR approval                  | Settings → Actions → General, require approval for outside collaborators                                                             | Keeps untrusted fork code from running workflows before a maintainer has looked                                          |
| Merge settings                    | Settings → General → Pull Requests. Allow squash merging only, default the commit message to the PR title, auto-delete head branches | One conventional subject per PR lands on `main`, which is what git-cliff reads for the release notes                     |
| Code scanning                     | Runs from `codeql.yml`, keep "Default setup" off in Settings → Code security                                                         | Default setup conflicts with the advanced workflow, private repos need GitHub Advanced Security for the alerts to appear |
| Secret scanning + push protection | Settings → Code security, on by default for public repos                                                                             | Blocks committed credentials                                                                                             |
| Dependabot alerts                 | Settings → Code security, enable Dependabot alerts and security updates                                                              | `dependabot.yml` only schedules version updates, the vulnerability alerts and auto-fix PRs are separate toggles          |
| Template toggle                   | Leave "Template repository" off if this repo was made from the template                                                              | Prevents re-templating                                                                                                   |
| Housekeeping                      | Settings → General. Set description and topics, untick Wikis and Projects if unused                                                  | Discoverability, fewer places for stale content                                                                          |

## How the infra works

| Component                | What it does                                                                                                                                                                                                                                                      | Why                                                                                             |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| bun monorepo             | `apps/*` + `packages/*`, one lockfile, shared tooling at the root                                                                                                                                                                                                 | Add a backend (`apps/api`) or shared code (`packages/shared`) later without reworking the setup |
| ESLint (flat) + Prettier | ESLint owns correctness, Prettier owns formatting                                                                                                                                                                                                                 | No rule conflicts, one formatter of record                                                      |
| Vitest + coverage        | Unit tests with an 80% line gate (`thresholds`)                                                                                                                                                                                                                   | Regressions surface locally and in CI                                                           |
| lefthook + shared hooks  | pre-commit formats and lints staged files, runs gitleaks when installed and the shared hooks from aktin-github-scripts (data files, large files, commit size), commit-msg enforces Conventional Commits via the shared hook, pre-push runs typecheck, test, build | Catch problems locally, keep history parseable for the changelog                                |
| CI                       | `ci.yml` calls the reusable `verify.yml` on push and PR, cancels superseded runs, least-privilege token                                                                                                                                                           | Fast, safe feedback                                                                             |
| Security scans           | CodeQL, dependency-review, OpenSSF Scorecard, secret scanning, all surfaced in the Security tab                                                                                                                                                                   | Catch code and supply-chain issues early                                                        |
| Releases                 | Tag `vX.Y.Z` builds the tagged commit with the version taken from the tag (`package.json` stays at 0.0.0), git-cliff release notes, CycloneDX SBOM, SLSA attestations, assets attached to the Release                                                             | Reproducible, auditable releases from commit history                                            |
| Dependabot               | Weekly grouped npm and github-actions updates                                                                                                                                                                                                                     | Stay patched, grouping avoids partial bumps that break paired steps such as CodeQL              |
| SHA-pinned actions       | Every action is pinned to a full commit SHA with a version comment                                                                                                                                                                                                | A moved tag cannot swap the code, Dependabot keeps the SHAs current                             |
| Pinned bun               | Runtime version fixed in `.bun-version`                                                                                                                                                                                                                           | Same runtime locally and in CI                                                                  |

## Project structure

```
apps/web/          Vue 3 + Vite app (the default workspace)
packages/          shared packages (add as needed)
.github/workflows/ CI, CodeQL, Scorecard, dependency-review, release
scripts/init.ts    one-time template initializer (removed after first run)
```

## Adding a backend or shared package

Create `apps/api` or `packages/shared` with its own `package.json`, extend `tsconfig.base.json`, and the root workspace globs pick it up. Root scripts (`build`, `test`, `typecheck`) fan out to every workspace.

## License

Apache-2.0. See [LICENSE](./LICENSE) and [NOTICE](./NOTICE).
