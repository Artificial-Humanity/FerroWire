# FerroWire (`fl-wire`)

**Durable agent-to-agent messaging: lasting agent IRIs, case-insensitive addresses, per-agent mailboxes with delivery receipts, a local Unix-socket daemon, a WebSocket/TLS peer relay, and native push delivery to five coding-agent CLIs.**

FerroWire is an independent sub-product of the [FerroLoop](https://github.com/Artificial-Humanity/FerroLoop) suite. It is the transport half of agent messaging: `fl-wire` holds the daemon, the peer protocol, the route table and the push dispatcher. The message model (`fl_core::ferrowire`) and the storage traits (`fl_core::store::{MessageStore, AgentRegistry}`) live in `fl-core`, with a durable `RedbStore` backend in `fl-store`. Both come from the FerroLoop repository as git dependencies.

## Why it exists

A coding agent that is busy, restarted or on another machine still has to receive its messages. FerroWire stores every message in a durable mailbox first, then pushes it to the live session when one exists. A message is never lost because the recipient was idle, and a receipt records what happened to it.

## Capabilities

### Lasting agent identity and addresses

- Every agent has an immutable IRI (`urn:uuid:...`). A persona that is addressed before it registers gets the deterministic fallback IRI `urn:fl:agent:<name>`.
- Addresses are case-insensitive. `WireAddress` accepts:

  | Address | Reaches |
  | --- | --- |
  | `ferris`, `Ferris` | the persona with that name key |
  | `urn:uuid:...` | one agent by immutable IRI |
  | `urn:fl:agent:ferris` | a persona by fallback IRI |
  | `@all` | every agent |
  | `@area:<slug>` | every agent in a coverage area or project |
  | `@role:<role>` | every agent holding a role, such as `@role:lead` |
  | `@area:<slug>/<role>` | one role inside one area, such as `@area:ferroloop/lead` |

- The sender is resolved in a fixed order: explicit argument, then `AGENT_PERSONA` or `AGENT_NAME`, then vendor session and route-cache inspection, then the project lead found from the working directory in the agent registry, then `unknown`.

### Durable mailboxes with receipts

- Message kinds: `mail`, `ask`, `reply`, `handoff`, `task_offer`, `approval_request`, `system`. Urgency is `normal` or `urgent`. Replies join a thread by correlation ID.
- Limits are enforced at the boundary: subjects up to 512 bytes, bodies up to 262,144 bytes.
- Each recipient gets a `DeliveryReceipt`. Status moves `pending` → `delivered` → `acked`, or ends at `expired`. Receipts merge by a monotonic rank, so two machines that saw different halves of the story converge on the same result.
- Reading a mailbox with `ack` acknowledges what it returns.

### Local Unix-socket daemon

- One daemon per host listens on a Unix domain socket: `$XDG_RUNTIME_DIR/fl-wire.sock`, falling back to `~/.cache/fl/wire.sock`.
- The protocol is newline-delimited JSON. Requests: `send`, `read`, `ack`, `register_route`, `subscribe`, `status`, `shutdown`. Requests are capped at 2 MiB.
- `subscribe` streams events for one recipient or one correlation ID, which is how a caller waits for a reply.
- `LocalClient` is a small blocking client for the same socket.

### WebSocket/TLS peer relay

- Daemons on different machines relay to each other over RFC 6455 WebSocket, on `ws://` or `wss://` (rustls with the `ring` provider).
- A peer connection starts with a mutual challenge/response handshake. Each side proves it holds the shared token with an HMAC-SHA256 over both nonces and its own host name, with the role bound into the MAC so a reflected proof is useless.
- After the handshake the peers exchange `SyncBatch` catch-up frames keyed by per-stream sequence numbers (messages, receipts, registry events), then live `Publish`, `ReceiptUpdate` and `RegistryUpdate` frames. Pings run every 15 seconds and an idle peer is dropped after 45.
- Inbound peer connections are capped at 64. A daemon that listens for or dials peers refuses to start without a non-empty pre-shared key.

### Push delivery to five vendor CLIs

The dispatcher keeps a route table (persona → live vendor session) and pushes each new message to the recipient's session. Routes are registered explicitly or discovered passively from the host.

| Vendor | Route kind |
| --- | --- |
| Claude Code | `claude` — session socket and transcript key |
| Antigravity | `agy` — conversation ID and language-server address |
| Codex | `codex` — thread ID and working directory |
| Muse | `muse` — session ID; pastes into a live tmux pane, else runs `muse exec` headless |
| Grok | `grok` — session ID; pastes into a live tmux pane, else runs `grok -p` headless |

A `webhook` route exists for deterministic tests that need to capture pushes without a vendor binary. A failed push is counted and retried up to 5 deliveries, then left in the mailbox. Headless turns get a 15-minute ceiling by default (`FL_WIRE_HEADLESS_TIMEOUT_SECS` overrides it). The dispatcher can also plan and run an interrupt (tmux send-keys or an Antigravity cascade cancel) and clear or re-forge a session.

### `agent-msg` compatibility

FerroWire is a drop-in replacement for the older `agent-msg` bus. The `fl wire` subcommands and an `agent-msg`-compatible command line (`register`, `roster`, `read --ack`, `send`, `broadcast`, `ack`) live in the FerroLoop CLI and drive this crate through the daemon socket. This crate has no binary of its own.

## Use as a library

```toml
[dependencies]
fl-wire = { git = "https://github.com/Artificial-Humanity/FerroWire.git" }
```

```rust
use std::sync::Arc;
use fl_wire::daemon::{DaemonConfig, default_socket_path, spawn_daemon};

// `store` is any type that implements `MessageStore + AgentRegistry`,
// for example `fl_store::RedbStore`.
let config = DaemonConfig {
    node_id: "node-a".into(),
    host: "host-a".into(),
    socket_path: default_socket_path(),
    routes_path: None,
    home_override: None,
    listen_addr: None, // set to "0.0.0.0:7700" to accept peers (needs `psk`)
    peers: vec![],     // peer URLs to dial, such as "wss://host-b:7700"
    psk: vec![],
};
let handle = spawn_daemon(Arc::new(store), config)?;
println!("listening on {}", handle.socket_path().display());
```

| Module | Holds |
| --- | --- |
| `daemon` | Unix-socket server and client, peer relay, `spawn_daemon` |
| `peer` | peer frames, protocol version 2, PSK handshake |
| `ws` | RFC 6455 framing, client and server upgrade, `ws://` and `wss://` |
| `identity` | persona resolution, `VendorRoute`, `RouteTable`, passive route discovery |
| `dispatcher` | push delivery per vendor, retry bound, interrupt and session reset |

## Verification

```bash
cargo fmt --all -- --check
cargo clippy --all-targets --workspace -- -D warnings
cargo test --workspace
```

## License

Apache-2.0. See [LICENSE](LICENSE).
