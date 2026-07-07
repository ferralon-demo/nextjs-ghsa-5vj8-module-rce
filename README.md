# Next.js — module-resolution RCE (GHSA-5vj8-3v2h-h38v) — `baseline`

> ## ⚠️ DEMONSTRATION / CASE-STUDY BRANCH
> Byte-for-byte snapshot of **real** Next.js source at a specific historical
> commit, for security-analysis demonstration only. **NOT for any real or
> production use.** Do not deploy, install, or depend on it.

## What this branch represents

The **last-vulnerable** state: Next.js **5.0.0**, the final release before the
fix. The reachable code-injection sink predates this release, so `baseline`
already carries it and **doubles as the Proof-of-Exploitability (PoE)** state —
there is no separate `vulnerability-introduced` branch for this case.

| | |
|---|---|
| Upstream tag | `5.0.0` |
| Upstream commit | `4697cbc485855d919ac6c2aef9a433d44b3699b6` ("Release 5.0.0") |
| Release date | 2018-02-05 |
| Snapshot tree OID | `705265a742654c272374f78ac92f1c9d82c1219a` |
| Expected static verdict | **reachable_candidate** (PoE) |

## The reachable sink

`server/require.js` at 5.0.0 is, in full:

```js
import resolve from './resolve'

export default async function requireModule (path) {   // <-- SINK (reachable)
  const f = await resolve(path)                         // unrestricted user-path → any file
  return require(f)                                     // arbitrary require = RCE
}
```

`resolve` is defined in `server/resolve.js` (present at this snapshot) and does
**no** bundles-directory containment. The catch-all page route reaches
`requireModule` with the request path as `page`, so an attacker-controlled path
flows to `require()` → arbitrary module load (CWE-94).

`ResolveDependencySymbols(["requireModule"])` → **1 match** here; the node is a
call-graph vertex reverse-reachable from an `http_route` ingress ⇒
**reachable_candidate**.

## How it differs from `fix`

`fix` (5.1.0) **removes** `requireModule` (renamed to a guarded `requirePage`)
and **deletes** `server/resolve.js`. The real code diff is the open PR
[`baseline` ← `fix` (#1)](https://github.com/ferralon-demo/nextjs-ghsa-5vj8-module-rce/pull/1).

## Provenance & license

Not a fork — a real source snapshot; no upstream remote or git parents. Upstream:
<https://github.com/vercel/next.js> @ `4697cbc485855d919ac6c2aef9a433d44b3699b6`.
Upstream `license.md` (MIT) retained verbatim. `README.md` is the only authored
file on this branch; all other files are the upstream tree, unmodified.

Full index: [`main`](https://github.com/ferralon-demo/nextjs-ghsa-5vj8-module-rce/tree/main).
