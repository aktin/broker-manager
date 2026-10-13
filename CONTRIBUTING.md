# Contributing

## Prerequisites

[bun](https://bun.sh) is the package manager and runtime. The version is pinned in `.bun-version`.

```sh
bun install
```

This installs dependencies and sets up the git hooks (via lefthook).
The shared git hooks are Python scripts and need `python3` on PATH. They are
pulled from the repository configured under `remotes` in `lefthook.yml`, change
`git_url` and `ref` there to use another source.

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

Messages follow [Conventional Commits](https://www.conventionalcommits.org),
enforced by the shared `commit-msg` hook from
[aktin-github-scripts](https://github.com/aktin/aktin-github-scripts).

## Pull requests

Make sure `bun run lint`, `bun run typecheck`, and `bun run test` pass. The
pre-push hook runs those plus the build.

## Releases

Push a `vX.Y.Z` tag. The release workflow builds the bundle, generates the
release notes, produces an SBOM and attestations, and publishes the
GitHub Release.
