# North Star — FerroWire

> The map, not the implementation: the *why*, and the standing decisions the build serves.
> When a branch or a detour raises "wait, what are we doing again?", this answers it.
>
> ⚠ **This file owns nothing but §1.** Everything else is held in [`AGENTS.md`](AGENTS.md)
> or in the private working notes, and the owning file is named at each point. If they
> disagree, the owning file wins and this one is the defect.
>
> ⚠⚠ **§1 IS THE OWNER'S AND IS UNRATIFIED.** It was assembled 2026-09-08 by **FerroTrack's
> resident** from the owner's own words in conversation on 2026-09-04 — organised, not
> invented — and it is **awaiting the owner's rewrite or ratification.** Keep that order for
> any future edit to §1. ⚠ **A wrinkle the next resident should know**: the owner spoke
> those words about FerroTrack, *before* the 2026-09-07 split. They describe the registry
> and the router as the heart of the product and the issue tracker as not — which is why
> they read as FerroWire's vision now. **That reading is the drafter's, and it is exactly
> the kind of inference the owner should confirm rather than inherit.**

---

## 1. The Vision — *assembled from the owner's words, UNRATIFIED*

**The registry and the communication router are the heart of it.**

The gap FerroWire exists to close: **combining locally hosted agents with frontier-vendor
agents is becoming common, and nothing lets them find each other.** Claude Code has
messaging between its own agents but **no registry**. Nothing addresses the mixed case — a
local model and a frontier-vendor agent in one workflow, needing to be addressable,
routable and wakeable by the same mechanism.

**It is built for us and shared with the world.** Both, genuinely. In the owner's words:
*"I intend to both use the product and share it… valuable things should, ideally, be shared
with others if it's realistic."* There is **no charge, no commercial product and no service
entity** around it. We build it for ourselves and accommodate potential others.

⚠ **Being built for us is not a licence for it to stay rough.** *"Just because it's built
for me, it does not mean it remains in an unpolished state."* That is a quality standard,
and it is why the protocol decisions in `AGENTS.md` are treated as contracts rather than as
internal conveniences — other people's agents will code against them.

**Intended for small teams and solo developers working with multi-agent workflows.**
⚠⚠ **A target, not a wall — explicitly not a rule that FerroWire never becomes anything
else** (owner, 2026-09-04, declining to state a "must never become" constraint on the
grounds that inferred rules of that shape harden into walls). Read it as aim. Never cite it
to rule something out.

⚠ **A correction the owner made while stating this, worth carrying:** an earlier framing —
that the owner was the customer — *"became both a rule and a wall. It was never the
intent."* If this document is ever used to narrow the audience, it is being misread.

---

## 2. What is ours, and what is rented

**Ours** — the registry, the router, delivery semantics, wake-up, the address format, the
scheduler, and whatever the companion spawner turns out to be. Nearly the whole product.

**Rented** — a backing store, not yet chosen (`AGENTS.md` §Open 7), and whatever an MCP
server implementation costs.

---

## 3. The One Organizing Principle

> **The protocol is the product, because the clients are other people's.**

FerroWire's users are agents written by vendors this project does not control. That single
fact decides most arguments: it is why the interface is a network protocol rather than a
Rust API, why liveness imposes nothing on clients beyond staying connected, why
idempotency is documented as a client contract, and why the query surface can be widened
but never narrowed.

---

## 4. Load-Bearing Constraints

Each is held elsewhere; this section points.

- **Public repo — nothing lab-internal.** `AGENTS.md` §Standing rules 1.
- **Apache-2.0**, aligning with §1's not-commercial, share-freely posture.
- **The interface is a network protocol, not a Rust API** — agents in other vendors'
  runtimes cannot link a crate.
- **Installation must not require broad filesystem grants** — the reason spawning lives in
  an optional companion rather than in the core.
- **Delivery semantics are a client contract**, not an implementation detail: at-least-once
  means every client must be idempotent.

---

## 5. The Real Bottleneck

**A decision, not a measurement.** Unlike its siblings, FerroWire's blocker is not an
unverified premise but an unanswered question: ⚠⚠ **whether MCP can carry the push
direction** (`AGENTS.md` §Open 2). Wake-up and delivery run against MCP's agent-as-client
grain, and the answer decides whether one protocol serves or two — which shapes the
registry, the spawner and the client contract behind it.

⚠ Second in line, and cheaper to answer than to retrofit: **the registry's identity story**
(§Open 3). Authentication retrofits worst into a protocol whose clients you do not control.

---

## One-Breath Summary

A registry that agents enter when they start, a router that carries messages between them,
and a way to wake one when work arrives — across vendors, so a local model and a
frontier-vendor agent can address each other by name. Built for us, polished as though for
the world, and given away.
