# checksum-signing

## Feature exercised

pnpm 12 RC 2 enforces signature/checksum verification during install:
every package entry in `pnpm-lock.yaml` includes an `integrity` field
(sha512 hash) validated against registry-published signatures. This
probe checks that Mend's UA reads `resolution.integrity` fields
correctly and resolves both scoped and unscoped packages under the
new trust-policy settings (`verify-store-integrity`, `globalShims`).

## Pattern categories

`install_command`, `checksum_signing`

## Source release

pnpm 12 RC 2

## Packages

| Package | Type | Version | Notes |
|---|---|---|---|
| `zod` | unscoped direct | 3.22.4 | sha512 integrity in lockfile |
| `@sindresorhus/is` | scoped direct | 5.1.0 | sha512 integrity in lockfile |

## pnpm config

- `.npmrc`: `verify-store-integrity=true`, `public-hoist-pattern[]=*`
- `package.json` pnpm block: `trustPublicHoisting: true`
  (reflects pnpm 12 globalShims trust policy)
- Lockfile format: v9.0 (pnpm 9+)

## Expected dependency tree

- `root.name`: `checksum-signing`
- `root.dependencies`: `["zod", "@sindresorhus/is"]`
- `packages` map has exactly two entries:
  - `zod` at version `3.22.4`, source `registry`, no child deps
  - `@sindresorhus/is` at version `5.1.0`, source `registry`, no
    child deps
- Source type is `registry` for both: `resolution.integrity` sha512
  hashes confirm registry origin; no `git`/`local`/`url` indicators.

## Mend failure modes this probe targets

1. **sha512 vs sha1 mismatch** — UA may expect sha1 for SHA1
   enrichment; sha512-only entries could be misread as
   "unresolvable", causing packages to be silently dropped.
2. **Scoped package scope-stripping** — `@sindresorhus/is` must
   appear with its full scope in the tree; stripping `@sindresorhus/`
   is a known failure mode for snapshot-keyed v9 lockfiles.
3. **`.npmrc` misparse** — `public-hoist-pattern[]=*` must not be
   treated as a package entry.
4. **Offline-mode fallback** — if the UA pre-step runs with `--offline`
   it will fail to fetch signature data and may fall back to the
   filesystem collector, losing hierarchy. This probe has no
   `--offline` flag; any hierarchy loss is a UA bug.

## Mend config

Bucket A — default-emit. `js-pnpm` has no dynamic version detection
from the manifest. `.whitesource` pins `pnpm: "12.0.0-rc.2"` and
`node: "20.11.1"` so `install-tool` provisions exactly the toolchain
this probe was built against.

`configMode` is `"AUTO"` — no `whitesource.config` is present in
this probe root.
