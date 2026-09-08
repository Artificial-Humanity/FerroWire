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

⚠ **Until the owner assigns an identity here: do not run `agent-env` to learn who you are,
and do not commit using what it returns.** Ask the owner. Committing as another repo's
agent is a convention failure that leaves no error behind — only a wrong name in the log.

⚠ **The sibling repos are not this repo.** FerroStep and FerroTrack each have their own
`AGENTS.md`, and a convention that holds in one of them can be reversed in another. Re-read
this file's rules on arrival rather than carrying habits across the boundary.

**Keep this file short.** It exists to route; the rules live in `AGENTS.md`, the reasoning
in `notes/`, and the *why* in [`north-star.md`](north-star.md).
