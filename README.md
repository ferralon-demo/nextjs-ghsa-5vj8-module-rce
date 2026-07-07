# Next.js — module-resolution RCE (GHSA-5vj8-3v2h-h38v)

Case study of a code-injection vulnerability (CWE-94) in the Next.js server: a
catch-all page route passed an attacker-controlled path to `require()` with no
containment, reaching an arbitrary-module load. Reconstructed from real upstream
source at two historical release commits to demonstrate reachability-based static
analysis.

> ## ⚠️ DEMONSTRATION / CASE-STUDY REPOSITORY
> This repository reconstructs **real open-source code at specific historical
> commits** to demonstrate security analysis. It is **NOT for any real or
> production use**. Do **not** deploy, install, or depend on it. The code here is
> a byte-for-byte snapshot of Next.js at the mapped commits; it is deliberately
> pinned to old, vulnerable releases.

## Synopsis

| | |
|---|---|
| **Advisory** | [GHSA-5vj8-3v2h-h38v](https://github.com/advisories/GHSA-5vj8-3v2h-h38v) (npm advisory 1538) |
| **CVE** | none assigned |
| **Weakness** | CWE-94 (code injection via `require()` of a user-controlled path) |
| **Affected** | `next` `>=0.9.9 <5.1.0` — patched in **5.1.0** |
| **Sink** | `requireModule(path)` in `server/require.js` — `require(await resolve(path))` |
| **Ingress** | catch-all page route reaches the sink with the request path as `page` |
| **Archetype** | **A** — reachable first-party sink; the fix **removes** the sink symbol |

### Why SCA misses this
A software-composition scanner sees only "the app depends on `next` — bump
`5.0.0` → `5.1.0`." It cannot say **which symbol** carried the reachable code
path, nor confirm that the fixed release no longer reaches it. Reachability
analysis keys on the specific resolver — `requireModule` / `resolve` — that
carried the attacker-controlled path, and observes that it is **absent** in the
fixed tree.

### Honesty note (read this)
The real 5.1.0 fix **renames** `requireModule` → `requirePage` and adds a
bundles-directory containment guard (`getPagePath`, which throws on `../`). The
successor still calls `require()`, now behind that guard. The static verdict
flips because the **advisory-named sink symbols are genuinely deleted**:
`requireModule` no longer exists and `server/resolve.js` (the unrestricted path
resolver) is removed. That is an honest **structural** signal — the specific
reachable path named by the advisory is gone — **not** a claim that all
`require()` was removed. A reviewer may fairly characterise 5.1.0 as a guarded
refactor; this case keys on the removed symbol, which is the legitimate,
verifiable structural fact.

### Recency caveat
The vulnerable era is **old**: 5.0.0 dates to **2018-02-05** and the fix (5.1.0)
to **2018-03-26**. The project name is highly recognizable; the affected version
is ancient. Recency is a known weakness of this case — it is presented for the
clean symbol-removal shape, not as a current exposure.

## Provenance

Upstream project: **Next.js** — <https://github.com/vercel/next.js>.
This is **not a fork**: a fresh repository with real source **snapshots** at the
commits below. No upstream remote, no upstream git parents. The upstream
`license.md` (MIT) is retained verbatim on every code branch.

| Stage | Branch | Upstream tag | Upstream commit | Release date | Snapshot tree OID |
|---|---|---|---|---|---|
| baseline (last-vulnerable, doubles as PoE) | `baseline` | `5.0.0` | `4697cbc485855d919ac6c2aef9a433d44b3699b6` | 2018-02-05 | `705265a742654c272374f78ac92f1c9d82c1219a` |
| fix | `fix` | `5.1.0` | `e3a9c9502332371b9dbf2483297964b55f110087` | 2018-03-26 | `c9ca66cbe28d889580b4efa2393313e346b3bd68` |

Load-bearing upstream commits in the `5.0.0 → 5.1.0` range:
- `c8da345765e1e96b98a1625acfd70470956505de` — "Make page require faster (#3776)"
  — renames `requireModule` → `requirePage`, adds the `getPagePath` bundles
  containment guard in `server/require.js`.
- `903f15acc42c9e82b6211f7bb7bf684f4846a0ee` — "Make pageExtensions configurable
  (#3787)" — removes `server/resolve.js` (the unrestricted path resolver).

There is **no failed-fix stage**: 5.1.0 is a single, clean fix. There is also no
separate `vulnerability-introduced` stage — the reachable sink predates 5.0.0, so
the `baseline` branch (5.0.0) already carries it and doubles as the PoE branch.

## Branch map

| Branch | State | Expected static verdict |
|---|---|---|
| [`baseline`](https://github.com/ferralon-demo/nextjs-ghsa-5vj8-module-rce/tree/baseline) | Next.js 5.0.0 — reachable `requireModule` sink present; `server/resolve.js` present | **reachable_candidate** (PoE) |
| [`fix`](https://github.com/ferralon-demo/nextjs-ghsa-5vj8-module-rce/tree/fix) | Next.js 5.1.0 — `requireModule` removed; `server/resolve.js` deleted; guarded `requirePage`/`getPagePath` successor | **not_exploitable** (PoNE — sink symbol/path absent) |

## PR map

The transition PR carries the reviewable diff. It is left **open** (never merged;
`main` stays README-only).

| PR | base ← head | Shows |
|---|---|---|
| [#1](https://github.com/ferralon-demo/nextjs-ghsa-5vj8-module-rce/pull/1) | `baseline` ← `fix` | the real Next.js `5.0.0 → 5.1.0` fix diff (sink removal + `resolve.js` deletion) |

## License

Each code branch retains the upstream Next.js `license.md` (MIT) verbatim. The
only file authored for this case study is `README.md` (this index, specialized
per branch).
