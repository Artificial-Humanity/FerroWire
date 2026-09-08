# FerroWire — rules of record

**The agent communication platform** (owner, 2026-09-07/08): a registry agents enter when
they start, a router that carries messages between them, and a wake-up mechanism.

> **Provenance, and it matters for how you read this file.** Drafted 2026-09-08 by
> **FerroTrack's resident**, at the owner's direction, because FerroWire's requirements were
> decided while the communication product was still inside FerroTrack and that agent held
> the record. **It is a transfer, not a set of native rulings**, and the owner has ratified
> none of it as FerroWire's.
>
> ⚠⚠ **Every decision below was made about this product, but made in FerroTrack's context.**
> Each carries its date. **The next resident should treat them as inherited and put anything
> load-bearing back to the owner** rather than assume this file speaks for FerroWire.
> The sibling convention is deliberate: a ruling from elsewhere enters a repo on its own
> merits, individually, or not at all — and this workspace has measured that rules cross
> between sibling repos *past* written guards, so the risk is real rather than theoretical.
>
> ⚠ **This repo has no resident and no roster.** The owner assigns one.

## What the owner has decided

1. **FerroWire is a separate product** (owner, 2026-09-07). It was requirements 2, 3 and 4
   of FerroTrack's product brief — inter-agent communications routing on a registry, the
   wake-up mechanism, and task scheduling — and the owner moved them out so each product
   does one thing. FerroTrack keeps issue management and its referee.
2. **Inter-brand by design, and this is the point rather than a feature.** Codex, Gemini,
   Grok and **locally hosted models** must all address one another through it. ⚠ The gap the
   owner named: Claude Code has messaging between its own agents and **no registry**.
3. **Protocol: MCP** (owner, 2026-09-05, extended 2026-09-07 to the whole Ferro line on the
   grounds that these are agents-first systems). ⚠ Stated as a direction with details to
   settle later — the default a Ferro project argues its way out of, not a finished spec.
   ⚠⚠ **See the open question in §Open, which is FerroWire's alone**: MCP is agent-as-client
   calling tools, and **wake-up and delivery run the other way.**
4. **Liveness is CONNECTION-BASED** (owner, 2026-09-04). An agent is live if and only if it
   holds an open channel — so "live" means exactly "deliverable right now", and it imposes
   nothing on clients beyond staying connected. ⚠ That last point carried the decision:
   heartbeating is an obligation every *other vendor* would have to implement, which is
   expensive for a protocol whose purpose is that they adopt it.
5. **Delivery is AT-LEAST-ONCE, with explicit acknowledgement and a TTL** (owner,
   2026-09-04). Retained until the recipient acknowledges; expired after a configured age.
   ⚠⚠ **Agents must therefore be idempotent, which is a contract obligation on every
   client** — it belongs in the protocol documentation, not in a design note.
6. **Addresses are FerroWire's to define: normalized, case-insensitive, naming an AGENT
   rather than a session** (owner, 2026-09-04), so an address survives a restart. One rule
   for every vendor's client instead of each inventing its own.
7. **Wake-up is an OPTIONAL COMPANION SPAWNER; the core never starts a process** (owner,
   2026-09-04). A separate, opt-in component holds the per-vendor launch commands and
   whatever grants spawning needs. ⚠ The reason is a bar the sibling products share: an
   install must not require broad filesystem grants. **The split is the mitigation — keep
   spawn logic out of the core.**
8. **Licence: Apache-2.0**, the house default for every new repo (owner, 2026-08-01).

## Standing rules

1. ⚠ **This is a PUBLIC repo.** Nothing lab-internal enters it: no hostnames or ports, no
   credential locations, no internal service or session names, no tracker issue numbers.
   Establish disclosure context *before* material arrives from a lab session.
2. **`north-star.md` carries the why**, under exactly that filename — the workspace
   convention. ⚠ **Its §1 Vision is the owner's to write or ratify**; the draft here is
   marked unratified and must stay that way until the owner acts on it.
3. **No review lane, no process invented.** Work is owner-directed until the owner says
   otherwise. Ask rather than inventing one.

## Open — none of these has an answer yet

1. ⚠⚠ **Where the registry lives.** FerroTrack's brief said one registry serves both
   products; the split reopened it. The registry has two halves that divide cleanly:
   **identity** (an address names an agent — FerroTrack needs this for an issue's author and
   assignee) and **presence** (is this agent connected — FerroWire's alone). Three answers,
   none chosen: FerroWire owns the whole registry and FerroTrack asks it; a shared library
   holds identity and each product embeds it; or each keeps its own and accepts drift.
   ⚠ **A constraint narrows this**: FerroTrack uses redb, which takes a file lock and
   permits one writer, so two programs cannot share one store file.
2. ⚠⚠ **Whether MCP can carry the push direction.** Register, send and acknowledge are
   natural tool calls. Wake-up and delivery are not. MCP defines server→client
   notifications; **whether they suit a general delivery channel must be verified against
   the current spec, not assumed.** The answer decides whether one protocol serves or two.
   ✅ If they do fit, liveness gains a precise definition for free: "holds an open MCP
   session" is an already-specified form of "holds an open channel".
3. ⚠⚠ **The registry has no identity story, and it is the largest hole.** As specified,
   anything that can reach the port may register under any address and receive that agent's
   messages. **"May claim an address" and "may cause a process to launch" are a potent pair.**
   ⚠ Authentication retrofits worst into a protocol whose clients you do not control — which
   is the position decision 2 puts FerroWire in deliberately. FerroStep's acceptance list
   has a usable shape: *bind an existing authenticated identity carrying a readable role —
   bind, don't mint.*
4. **Whether scheduling needs a subsystem at all.** The owner described a scheduled task as
   *"another wake-up message"*. Taken literally it is a message with a `deliver_after` time,
   and the scheduler is a scan of what is due — no cron parser, no timer service. ⚠ The one
   thing that would break that reading is **recurrence** ("every weekday at 09:00"), which a
   single timestamp cannot express. That is an owner question.
5. **Protocol versioning from the first release.** Other people write the clients, so the
   release that *adds* a version breaks every client that has none.
6. ⚠ **Connection-based liveness has a false-death case the spawner turns into a bug.** An
   agent that is running but briefly disconnected reads as dead and gets launched again.
   Single-flight per agent stops ten queued messages causing ten spawns; it does **not** stop
   one spawn duplicating a live process.
7. **The backing store is undecided.** ⚠ Do not inherit FerroTrack's answer — its workloads
   left with this product, and the reasoning that chose redb was about an issue tracker.
   `notes/inherited-store-research.md` records what transfers and what does not.
