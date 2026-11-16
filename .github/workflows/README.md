# CI/CD and repository rules

What runs on GitHub for this repository and which rules protect `main`, `dev` and release tags. The git hooks that run on your machine are described in the [root README](../../README.md#checks-and-gates).

## On a pull request to `dev` or `main`

All checks start on their own. A pull request can be merged when the checks marked with yes passed and the rules under [Gates on `main` and `dev`](#gates-on-main-and-dev) are met.

| Check                    | Workflow                                                                                                        | Required to merge                                                                                                        |
| ------------------------ | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `verify / verify`        | `verify.yml`, passes only when `verify / base / verify` and `verify / bun` passed                               | yes                                                                                                                      |
| `verify / base / verify` | `verify.yml`, lints Dockerfiles with hadolint, shell scripts with shellcheck and workflow files with actionlint | not as a check, but `verify / verify` fails when it fails                                                                |
| `verify / bun`           | `verify.yml`, Prettier check, ESLint, typecheck, tests with coverage, build                                     | not as a check, but `verify / verify` fails when it fails                                                                |
| `review / review`        | `scan-dependencies.yml`, fails on new dependencies with a known vulnerability of severity high or critical      | yes                                                                                                                      |
| `analyze / analyze`      | `scan-codeql.yml`, CodeQL scan of the JavaScript and TypeScript code                                            | not as a check, but the ruleset on `main` blocks the merge on CodeQL errors or security alerts of severity high or above |
| `CodeQL`                 | Posted by GitHub code scanning with the result of the CodeQL scan                                               | no                                                                                                                       |

## Workflows

The workflow files in this folder only hold the triggers and call their jobs as reusable workflows from [aktin-github-scripts](https://github.com/aktin/aktin-github-scripts/tree/main/.github/workflows) at `@main`.

What each workflow does and when it runs. Schedules are in UTC and run on `main`:

| Workflow                | Pull request to `dev` or `main` | Push to `main` | Schedule      | Other    | What happens                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------------------- | ------------------------------- | -------------- | ------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `verify.yml`            | yes                             | yes            |               |          | Starts the shared `verify-web.yml`, which runs three jobs. `base` checks the Dockerfiles with hadolint, the shell scripts with shellcheck and the workflow files with actionlint. It fails when hadolint reports an error, shellcheck a warning or worse, or actionlint any finding. `bun` installs the dependencies from `bun.lock` and runs `bun run format:check`, `bun run lint`, `bun run typecheck`, `bun run test` and `bun run build`. `verify` passes only when both passed                  |
| `scan-dependencies.yml` | yes                             |                |               |          | Compares the dependencies before and after the pull request and fails when a newly added one has a known vulnerability of severity high or critical. When it fails it posts the findings as a comment in the pull request                                                                                                                                                                                                                                                                             |
| `scan-codeql.yml`       | yes                             | yes            | Mondays 03:00 |          | Scans the JavaScript and TypeScript code with CodeQL for security mistakes and reports the findings in the Security tab                                                                                                                                                                                                                                                                                                                                                                               |
| `scan-scorecard.yml`    |                                 | yes            | Mondays 04:00 |          | Rates the security practices of the repository, for example branch rules, pinned actions, token permissions and code review, publishes the score to OpenSSF and to the Security tab and feeds the badge in the root README. It changes nothing                                                                                                                                                                                                                                                        |
| `release.yml`           |                                 |                |               | tag `v*` | Runs three jobs one after the other. `verify` runs the same checks as `verify.yml`. `build` builds the tagged commit with the version taken from the tag, packs the bundle as `broker-manager-web-<tag>.tgz`, generates the SBOM `sbom-web.cdx.json` from `bun.lock` and attests build provenance and the SBOM at GitHub. `publish` writes the release notes from the commit history with git-cliff and creates the GitHub Release with both files attached. It fails when it finds no file to attach |

The workflow token can only read by default. A job that needs more lists its permissions in the workflow file. CodeQL only runs through `scan-codeql.yml`, the default setup of GitHub is off.

Dependabot is configured in `.github/dependabot.yml`. Once a week it opens one grouped pull request per ecosystem against `dev`, one for the bun dependencies and one for the GitHub Actions. Dependabot alerts and security updates are switched on in the repository settings and report known vulnerabilities in dependencies that are already in use. Security updates open their pull requests against `main`. GitHub does not support security updates for bun yet, so these pull requests fail the `verify / verify` check.

## Gates on `main` and `dev`

Branch ruleset `default` on `main`:

| Rule                     | Meaning                                                                                                                       |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| Pull request required    | No direct pushes. Merge commit only, 1 approval, code owner review, approval after the last push, all review threads resolved |
| Required checks          | `verify / verify` and `review / review` must pass                                                                             |
| Code scanning            | CodeQL must have analysed the head commit without errors or security alerts of severity high or above                         |
| Signed commits           | Every commit on `main` is signed. Merge commits are signed by GitHub                                                          |
| No force push, no delete | History on `main` is append-only                                                                                              |

Branch ruleset `dev` on `dev`: pull request required with squash merge only, 1 approval and resolved review threads, the same required checks, signed commits, no force push and no delete.

Approvals come from people. GitHub Actions cannot create or approve pull requests, auto-merge is off, and no app or bot is on the bypass list. Organisation admins can bypass every rule. Other branches have no rules.

Only collaborators can open pull requests. Workflows on a pull request from a fork wait for an approval when the author is an external contributor. The head branch is deleted after a merge. Secret scanning with push protection is on, so GitHub blocks a push in which it finds a secret.

## Gates on tags `v*`

Tag ruleset `release-tags`: only maintainers and admins can create, move or delete release tags. A tag starts the release workflow, so this decides who can cut a release.

Releases are immutable. Once a release is published, its files cannot be changed or deleted and its tag cannot be moved.
