# Next.js — module-resolution RCE (GHSA-5vj8-3v2h-h38v) — `fix`

> ## ⚠️ DEMONSTRATION / CASE-STUDY BRANCH
> Byte-for-byte snapshot of **real** Next.js source at a specific historical
> commit, for security-analysis demonstration only. **NOT for any real or
> production use.** Do not deploy, install, or depend on it.

## What this branch represents

The **fixed** state: Next.js **5.1.0**, the release that patched the
module-resolution RCE.

| | |
|---|---|
| Upstream tag | `5.1.0` |
| Upstream commit | `e3a9c9502332371b9dbf2483297964b55f110087` ("5.1.0") |
| Release date | 2018-03-26 |
| Snapshot tree OID | `c9ca66cbe28d889580b4efa2393313e346b3bd68` |
| Expected static verdict | **not_exploitable** (PoNE — sink symbol/path absent) |

## What changed vs `baseline`

Two real upstream commits in the `5.0.0 → 5.1.0` range close the reachable path:

- `c8da345765e1e96b98a1625acfd70470956505de` — "Make page require faster (#3776)"
  — **renames** `requireModule` → `requirePage` and adds a bundles-directory
  containment guard `getPagePath` (throws on `../` / any path resolving outside
  the bundles directory).
- `903f15acc42c9e82b6211f7bb7bf684f4846a0ee` — "Make pageExtensions configurable
  (#3787)" — **deletes** `server/resolve.js`, the unrestricted path resolver.

At 5.1.0, `server/require.js` exports `requirePage` and `getPagePath`; there is
**no** `requireModule` symbol and **no** `server/resolve.js`.

`ResolveDependencySymbols(["requireModule"])` → **0 matches** here ⇒
**not_exploitable**. The reachable sink named by the advisory is gone.

## Honesty note (structural signal, stated plainly)

The static flip is real because the advisory-named symbols are **genuinely
deleted** — `requireModule` and `server/resolve.js` do not exist at 5.1.0. It is
**not** a claim that all `require()` was removed: the guarded successor
`requirePage` still calls `require()`, now behind `getPagePath` containment:

```js
export function getPagePath (page, {dir, dist}) {
  const pageBundlesPath = join(dir, dist, 'dist', 'bundles', 'pages')
  // ... normalizePagePath throws on ../ ...
  const pagePath = join(pageBundlesPath, page)
  if (parse(pagePath).dir.indexOf(pageBundlesPath) !== 0) {
    throw pageNotFoundError(page)          // outside bundles → rejected
  }
  return pagePath
}
```

So the verdict keys on the **absent `requireModule` symbol** (its reachable path
no longer exists), which is an honest structural fact. A reviewer may fairly
characterise 5.1.0 as a guarded refactor of the same `require()` path; this case
reports the removed reachable symbol, not a whole-class elimination of
`require()`.

## Provenance & license

Not a fork — a real source snapshot; no upstream remote or git parents. Upstream:
<https://github.com/vercel/next.js> @ `e3a9c9502332371b9dbf2483297964b55f110087`.
Upstream `license.md` (MIT) retained verbatim. `README.md` is the only authored
file on this branch; all other files are the upstream tree, unmodified.

The real `baseline → fix` diff is the open PR
[`baseline` ← `fix` (#1)](https://github.com/ferralon-demo/nextjs-ghsa-5vj8-module-rce/pull/1).
Full index: [`main`](https://github.com/ferralon-demo/nextjs-ghsa-5vj8-module-rce/tree/main).
