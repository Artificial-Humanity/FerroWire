# FerroWire

**The agent communication platform.** A registry that agents enter when they start, a router
that carries messages between them, and a mechanism that wakes an agent when work arrives —
built in Rust, and deliberately **inter-brand**: Claude, Codex, Gemini, Grok and locally
hosted models are all meant to address one another through it.

Sibling to [FerroStep](https://github.com/Artificial-Humanity/FerroStep), the referee for
database-ledger agent loops, and [FerroTrack](https://github.com/Artificial-Humanity/FerroTrack),
the AI-native issue tracker. FerroWire was part of FerroTrack until 2026-09-07 and was
separated so that each product does one thing.

**Status: pre-alpha.** The repository was created 2026-09-07 and holds no product code. See
[`north-star.md`](north-star.md) for what it is for and [`AGENTS.md`](AGENTS.md) for what
has been decided. ⚠ The working notes those decisions were made on are kept privately and
are not part of this repository.

## License

Apache-2.0 — see [LICENSE](LICENSE).
