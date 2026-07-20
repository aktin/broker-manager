# Contributing

## Prerequisites

[bun](https://bun.sh) is the package manager and runtime.

```sh
bun install
```

This installs dependencies and sets up the git hooks (via lefthook).

## Development

```sh
bun run dev        # start the web app (apps/web)
bun run test       # unit tests with coverage
bun run typecheck  # vue-tsc
bun run lint       # eslint
bun run format     # prettier
bun run build      # production build
```

## Commits

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org)
and are enforced by commitlint through the `commit-msg` hook. Examples:

- `feat(web): add request overview table`
- `fix: correct execution state transition`
- `chore: bump dependencies`

## Pull requests

Before opening a PR, make sure `bun run lint`, `bun run typecheck`, and
`bun run test` pass. The pre-push hook runs these plus the build.

## Releases

A release is cut by pushing a `vX.Y.Z` tag. That triggers the release
workflow, which generates the changelog and notes, builds and attests the
bundle, and publishes the GitHub Release.
