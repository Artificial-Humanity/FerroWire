# Store research inherited from FerroTrack — what transfers, and what must not

**Written 2026-09-08 by FerroTrack's resident, seeding this repo.** FerroTrack ran a full
backing-store evaluation over 2026-08-28 → 2026-09-05 and settled on redb. ⚠⚠ **That answer
is FerroTrack's and must not be inherited here.** The reasoning was about an issue tracker,
and **the workloads that argued the other way are exactly the ones that left with
FerroWire.** This note exists so the next resident starts from the evidence rather than
from the conclusion.

## ⚠⚠ The reversal, stated plainly

While the message bus was FerroTrack's, two arguments favoured **fjall** over redb and both
were judged not to have won:

1. **The message bus is write-heavy**, which is LSM's shape rather than a copy-on-write
   B+tree's.
2. **Message retention and dead registry entries need expiry**, and fjall has **compaction
   filters** — user logic run during compaction — which is the natural mechanism. redb has
   no expiry primitive at all; sweeps are hand-written.

When those workloads left FerroTrack, its record noted that both fjall advantages weakened,
which *confirmed* redb there. ✅ **The mirror image is the point: they did not weaken, they
moved. Both are now FerroWire's core workloads.** A store evaluation run for FerroWire
starts from a materially different position than the one that chose redb.

⚠ **This is not an argument that FerroWire should use fjall.** It is an argument that
FerroTrack's conclusion carries none of its weight across the boundary, and that anyone
citing "the Ferro line uses redb" is quoting a decision made about a different workload.

## What the workloads actually are here

| workload | shape | hot access | lifetime |
|---|---|---|---|
| **Registry** | small, hot, read-heavy | lookup by address; enumerate live agents | ⚠ entries outlive their agent — a crash never deregisters |
| **Messages** | **write-dominated**, append-shaped | per-recipient ordered drain | ⚠ short — delivered messages must not accumulate |
| **Schedule** | small | ⚠ **range scan by due-time**: "everything due before now" | until fired |

⚠ **The registry's hard problem is liveness, not storage.** Whatever the store, "is this
agent reachable" is never answered by the presence of a row. Connection state answers it.

## Measured field survey (2026-09-05, `cargo tree` + crates.io)

Dependency counts are resolved graphs at minimal feature sets; no build was run.

| crate | last release | recent downloads | crates | note |
|---|---|---:|---:|---|
| **redb** | 2026-08-17 | 4,656,539 | **1** | zero dependencies; COW B+tree; **no expiry primitive** |
| **fjall** | 2026-08-30 | 471,442 | 41 | LSM; **compaction filters**; partitions with cross-partition atomicity; LZ4 |
| **persy** | 2026-06-30 | 136,269 | 22 | ⚠⚠ **MPL-2.0, not permissive**; isolation is `read_committed` |
| sanakirja | 2026-07-06 | 32,892 | 21 | parked on a beta line |
| sled | **2024-10-11** | 2,760,043 | 16 | ⚠ stalled ~2 years on an alpha |
| canopydb | 2025-11-22 | 847 | 51 | very early |
| structsy | **2023-04-29** | 360 | 27 | effectively abandoned |

**The field is not sparse in count — it is sparse at the intersection of maintained,
adopted, and meeting the criteria, where it is about three.**

⚠⚠ **`sled` is the cautionary row and the lesson generalises:** 2.76M recent downloads
against a 2024 release, on an alpha. **Adoption measured without a release date is a number
that lies.**

⚠ **Every viable candidate is single-maintainer** (measured 2026-09-04): redb ~92% one
author; fjall the same shape; `lsm-tree`, the crate under fjall that actually persists
bytes, more so. **This is a property of the category, not a discriminator** — a risk flagged
against one candidate has to be measured against the others before it can move a decision.

## Traps that transfer unchanged

- ⚠⚠ **A store's safety claim describes the layer you CALL, not the layer that PERSISTS.**
  fjall carries `#![deny(unsafe_code)]`; `lsm-tree` beneath it has 31 `unsafe` blocks and no
  such attribute. The same shape caught SurrealDB: choosing its pure-Rust storage engine
  does **not** yield a C-free build, because the C dependency enters through *auth*
  (`aws-lc-sys` → `jsonwebtoken`) and is hard-coded per target family rather than
  feature-gated.
- ⚠⚠ **"Has ACID transactions" is not "evaluates the conditional update INSIDE the
  transaction."** Measure at the call site, **with a control** so a conflict-free pass cannot
  be mistaken for a vacuous one. ⚠ fjall's transactions are **opt-in**, and its own docs say
  the plain write path cannot do read-modify-write reliably — a silent hole, not an error.
- ⚠ **Check the licence before evaluating, not after.** `persy` is MPL-2.0, which would
  forfeit any future derivative work. The one candidate that was skipped was also the one
  whose licence went unchecked — the same omission.
- ⚠ **Full-text search, if FerroWire ever wants it, costs more than the store.** `tantivy`
  measured +89 crates over a no-search stack **and reintroduces a C toolchain** via
  `zstd-sys` in a core crate.

## Whole-product dependency measurements, for calibration

Measured the same day, on a plausible product rather than a store alone:

| stack | crates | native toolchain |
|---|---:|---|
| redb + axum + tokio | 56 | none |
| fjall + axum + tokio | 84 | none |
| + tantivy | 145 / 161 | `cc`, `pkg-config` |

⚠ Note the gap between stores **widens** without a search engine — tantivy's tree overlaps
fjall's and masks its marginal cost. A conclusion measured inside one stack does not survive
into another.

## Standing owner latitude that transfers

✅ **Building a new product on a permissively-licensed base is on the table** (owner,
2026-09-05), rebranded, *"only if the arguments for it are compelling enough."* redb, sled
and fjall are all `MIT OR Apache-2.0`. ⚠ The trigger recorded in FerroTrack: **a capability
that cannot be built above the store's API, named.** None has been named.
⚠⚠ **If it is ever taken, inherit the crash-safety core rather than rewriting it** — that is
what makes deriving different in kind from starting fresh, since the layer where bugs are
silent is the layer you keep.
