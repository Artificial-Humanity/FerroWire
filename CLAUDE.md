# FerroWire

**[AGENTS.md](AGENTS.md) is this repo's rules of record. Read it before you do anything else.**
It is not loaded for you — only this file is — so nothing else will put it in front of you.

⚠⚠ **This repo has no resident agent and no roster yet, and `ferrostep agent-env` WILL
LIE TO YOU ABOUT THAT.** Run here today it reports you as **Cyndi, FerroStep's developer**,
and exits 0. There is no error to notice.

**Why:** roster discovery prefers `<dir>/FerroStep/config.yaml` over a bare `config.yaml`
at each level walking up. One level above this repo sits the **FerroStep repository**,
whose directory satisfies that convention — so it wins, and the workspace-root
`config.yaml` that exists to be the shared parent layer is shadowed. Reproduced in a clean
fixture 2026-09-08 against `ferrostep` built from `622d71e`; reported to FerroStep.

⚠⚠ **The guard, sharpened by FerroStep's resident 2026-09-08 — it is not merely "distrust
the answer": `agent-env` here emits THAT AGENT'S identity, so a resident following the
documented commit procedure signs as them.** So:

> **Treat a resolved roster whose `AGENT_ROSTER` points into the FerroStep repository as a
> REFUSAL, not a result.** Stop and ask the owner. Do not commit with what it returned.

⚠ Filed upstream as FerroStep #353, **confirmed and deliberately not yet fixed**: the two
honest repairs — refuse on ambiguity, or layer both rosters instead of choosing — differ in
whether every `agent-env` in the workspace starts failing at once, and that is the owner's
call. ⚠ The blast radius is wider than this repo: eight directories under the workspace have
no roster of their own, and the shadowed workspace file also carries the declared
**credential source**, which is therefore unreachable too.

⚠ **The sibling repos are not this repo.** FerroStep and FerroTrack each have their own
`AGENTS.md`, and a convention that holds in one of them can be reversed in another. Re-read
this file's rules on arrival rather than carrying habits across the boundary.

**Keep this file short.** It exists to route; the rules live in `AGENTS.md`, the reasoning
in the private working notes (`notes/` is a symlink into the umbrella `Notes` repo and is
not published), and the *why* in [`north-star.md`](north-star.md).
