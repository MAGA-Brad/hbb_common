# hbb_common (sanitized fork)

This is a fork of [rustdesk/hbb_common](https://github.com/rustdesk/hbb_common), the shared library
used by RustDesk's client and server (AGPL-3.0). Full credit for this library goes to the RustDesk
team and its contributors.

It's included here as the submodule dependency of
[rustdesk-managed-client](https://github.com/MAGA-Brad/rustdesk-managed-client), which adds a
small number of changes on top of the otherwise-unmodified upstream library: the protocol messages
for device passports and relay tickets, quieter logging and a newer `webrtc` crate (see Status
below), and these config hooks: protecting the 2FA setting from being overwritten by generic
config-sync writes, and (post-1.5 `libs/base` split) managed clients always using `wss://`, never
falling back to plaintext `ws://`. It also makes "adaptive" the default desktop view style
(upstream defaults to "original"). The two managed-mode option keys (`preset-device-email`,
`managed-local-input-priority-ms`) now live in `libs/base::config::keys` alongside the rest of the
client-level option keys, not here.

## Status (RDC build 48)

Build 48 moves this branch onto upstream commit fae47e0, the hbb_common that RustDesk 1.5.1 uses,
and carries the managed changes forward:

- **Protocol** (`protos/rendezvous.proto`). Every added field uses number 101, clear of
  upstream's numbering:
  - `KxParams.managed_capabilities` (since build 34): what else hbbs accepts on a connection; the
    value 1 (lowest bit) means it takes a `DeviceAuth`. It's signed with the rest of `KxParams`,
    so it can't be stripped on the way, and clients that don't know the field ignore it.
  - `DeviceAuth` (`RendezvousMessage` field 101, since build 34): a managed device's passport and
    its identity key's signature over that connection's key exchange, sent right after the
    exchange when hbbs offers it.
  - `relay_ticket` in `RequestRelay` and `RelayResponse` (since build 43): hbbs's ticket for hbbr,
    which shows that each side of a relay belongs to a relay hbbs set up. The controlled side gets
    its ticket in the relay request hbbs forwards to it (or, when it started the relay itself, back
    in a `RelayResponse`), the controller gets its own with the answer, and each copies its ticket
    into the `RequestRelay` it sends hbbr.
- **WebRTC.** The `webrtc` crate is 0.17.3 instead of upstream's 0.13, which upstream keeps for
  its Rust 1.75 CI. The relayed-route and remote-IPv6 checks now read the selected candidate
  pair's fields directly instead of parsing its text form.
- **Logging.** Release builds log at info instead of debug (`RUST_LOG` still overrides that).
  Noisy WebRTC targets log errors only; the old `webrtc-sctp=warn` directive never matched (Rust
  log targets use the crate name with underscores, `webrtc_sctp`), so the filter now names
  `webrtc_sctp`. A WebSocket connection that keeps failing logs its error once, then at most every
  10 minutes.

Earlier builds' encrypted rendezvous connections and sealed WebRTC signaling use protocol messages
upstream hbb_common already has (the signed `KeyExchange` with `KxParams`, `webrtc_sdp_offer` /
`webrtc_sdp_answer` and `IceCandidate`); the managed-only logic lives in the client, and the server
half in [rustdesk-managed-relay](https://github.com/MAGA-Brad/rustdesk-managed-relay).

The `managed-relay` branch is what the relay uses: an older upstream commit (229b904) plus only the
protocol fields above (the passport message and the relay ticket), none of the client changes.
