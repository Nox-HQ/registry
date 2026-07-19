# nox plugin registry

The index the [nox](https://github.com/nox-hq/nox) CLI resolves plugins from,
plus the tooling that maintains it.

```
index.json                 what `nox plugin search` / `nox plugin install` read
cmd/registry-sync          reconcile the index against plugin GitHub releases
cmd/marketplace-build      render the public marketplace site from the index
```

## Why this is a separate repository

It used to live in the nox repo at `registry-scaffold/index.json`. That put core
in the business of cataloguing seven other repositories: core needed a GitHub
token, knew every plugin's release cadence, and failed CI when an unrelated
repository published. Plugin availability was also coupled to core's default
branch, since the index was served from
`raw.githubusercontent.com/nox-hq/nox/main/...`.

nox now only *consumes* the published index over HTTP. It does not import this
module, so the dependency runs one way.

## The index is curated, not a mirror

A GitHub release is **not** automatically offered. Listing a version here is
what makes it installable, and that is deliberate: it is the point at which a
human decides a build should be handed to users. Several historical versions
are intentionally absent.

`registry-sync -check` therefore reports only when the **newest** release of a
listed plugin is missing — the actionable case. `-all` audits everything.

## Digests are never invented

Every artifact digest and size comes from that release's own `checksums.txt`
and the GitHub API. An artifact whose checksum cannot be resolved is reported
and **omitted**: a placeholder in a security registry looks like verification
metadata and is not.

## Working on it

```bash
go run ./cmd/registry-sync -check     # is the index current?
go run ./cmd/registry-sync -write     # add missing releases
go run ./cmd/registry-sync -check -all
go run ./cmd/marketplace-build --index index.json --output dist
```

`GITHUB_TOKEN` is needed in practice: unauthenticated GitHub API access is
60 requests/hour, which a full pass exceeds.

## Removing a plugin

Delete its entry from `index.json`. `registry-sync` walks plugins already
listed, so it will not resurrect one — the index stays the source of truth for
what is *offered*, independent of what has a release.
