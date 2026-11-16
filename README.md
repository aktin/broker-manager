# broker-manager

[![Verify](https://github.com/aktin/broker-manager/actions/workflows/verify.yml/badge.svg)](https://github.com/aktin/broker-manager/actions/workflows/verify.yml)
[![CodeQL](https://github.com/aktin/broker-manager/actions/workflows/scan-codeql.yml/badge.svg)](https://github.com/aktin/broker-manager/actions/workflows/scan-codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/aktin/broker-manager/badge)](https://securityscorecards.dev/viewer/?uri=github.com/aktin/broker-manager)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/aktin/broker-manager)](https://github.com/aktin/broker-manager/releases)

Management software for the AKTIN query broker. A bun workspaces monorepo with a Vue 3 and TypeScript web app under `apps/web`.

## Getting started

You need [bun](https://bun.sh) on PATH (the version is pinned in `.bun-version`) and the tools that the git hooks call. They are listed under [On your machine](#on-your-machine).

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

Example: `feat(web): add the login form`

Allowed types are `feat fix docs style refactor perf test chore`. The subject starts lowercase, has no trailing period, and every line of the message stays at most 100 characters. Pull requests into `dev` are squash merged, so one conventional subject per PR lands on `dev`. `dev` is merged into `main` with a merge commit.

A release is a tag `vX.Y.Z` on `main`, for example `v1.2.3`. The release workflow takes the version from the tag (`package.json` stays at `0.0.0`) and attaches the web bundle and its CycloneDX SBOM to the GitHub Release. Its jobs are described in [.github/workflows/README.md](.github/workflows/README.md#workflows).

To check that a downloaded web bundle was built by the release workflow, verify its build provenance with the [GitHub CLI](https://cli.github.com):

```sh
gh attestation verify broker-manager-web-<tag>.tgz --repo aktin/broker-manager --signer-repo aktin/aktin-github-scripts

# Example:
# gh attestation verify broker-manager-web-v1.2.3.tgz --repo aktin/broker-manager --signer-repo aktin/aktin-github-scripts
```

`--signer-repo` is needed because the attestation is signed by the shared workflow in aktin-github-scripts, not by a workflow of this repository.

## Checks and gates

A change passes three layers: the git hooks on your machine, the checks on the pull request and the rules on the branch. Anyone can skip the hooks, only organisation admins can bypass the checks and the rules. The hooks are described here. The checks on pull requests, the workflows, Dependabot and the rules on `main`, `dev` and release tags are described in [.github/workflows/README.md](.github/workflows/README.md).

### On your machine

> [!CAUTION]
> Run `bun install` once after cloning, before your first commit. It runs `lefthook install`. Without it no git hook runs on your machine.

`bun install` installs the git hooks. [lefthook](https://lefthook.dev) runs them and takes all of them from [aktin-github-scripts](https://github.com/aktin/aktin-github-scripts#git-hooks), see `lefthook.yml`. They need `python3`, `gitleaks`, `actionlint`, `hadolint` and `shellcheck` on PATH. A missing tool fails the commit. actionlint, hadolint and shellcheck are only called when a workflow file, Dockerfile or shell script is staged.

| When       | What runs                                                                                                                                                                                                                                                    | Blocks     |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- |
| pre-commit | ESLint fixes the staged files, Prettier formats them, gitleaks scans for secrets, actionlint, hadolint and shellcheck lint the staged workflow files, Dockerfiles and shell scripts, the shared scripts reject data files, large files and oversized commits | the commit |
| commit-msg | Conventional Commit header check                                                                                                                                                                                                                             | the commit |
| pre-push   | `bun run typecheck`, `bun run test` (Vitest with an 80% line coverage threshold), `bun run build`                                                                                                                                                            | the push   |

Hooks can be skipped with `--no-verify`, so nothing here is a guarantee. They exist for fast feedback.

## License

Apache-2.0. See [LICENSE](./LICENSE) and [NOTICE](./NOTICE).
