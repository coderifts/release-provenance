# release-provenance

Public release provenance for CodeRifts packages — shipped-artifact metadata (npm gitHead, tag,
file manifest, SHA-256, dist.integrity, signature). **No source**; the app repo stays private.

## Why this repo exists

`@coderifts/release-acceptance` verifies that each published package came from the commit its
release tag names. Six of the seven source repositories are public and answer a plain
`git ls-remote`. One — `coderifts/app` — is private, and it publishes two of the nine artifacts
(`coderifts` and `@coderifts/release-acceptance`).

Measured, with no credentials:

```
coderifts/agent-guard  ->  b11a82c72b46…  HEAD
coderifts/app          ->  remote: Invalid username or token.
```

So the matrix scored 15/15 on a machine with access and 12/15 on a stranger's. That difference was
a property of the reader, not of the release. These records move the answer to a public place.

## What a record contains

`<artifact>/<version>.json`, plus `index.json` listing them all.

| field | what it is |
|---|---|
| `registry.git_head` | the commit npm recorded for the published version |
| `registry.integrity` / `shasum` | what the registry serves, verbatim |
| `registry.signatures` | the registry's signature over the package |
| `source.tag` / `tag_object` / `peeled_commit` | the release tag, its object, and the commit it names |
| `source.annotated` | whether the tag is annotated (a signed release) or lightweight (a movable name) |
| `shipped.files[]` | every path in the published tarball, with `sha256` and `bytes` |

## What it does NOT contain

No file bodies, no diffs, no paths outside the published tarball. Everything here is metadata of an
artifact anyone can already download; you can recompute the file hashes yourself with `npm pack` and
compare them against these records. The moat is the source, and the source is not here.

## Using it

```
git clone https://github.com/coderifts/release-provenance
npx @coderifts/release-acceptance --release --manifest <frozen-set.json>
```

The matrix finds the checkout at `$HOME/release-provenance`, or wherever `CODERIFTS_PROVENANCE_DIR`
points. A missing record is reported as UNKNOWN and blocks the release verdict — it is never read as
agreement.

## Honest scope

These records say a published artifact came from a named commit, and what bytes it shipped. They are
**trusted-executor integrity, RECORDED** — minted by someone with access to the source repositories
and published here. They are not externally witnessed, and this is not PATH B: no third party
observed the minting, and the registry signature covers the package, not this record.
