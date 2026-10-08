# hbb_common (sanitized fork)

This is a fork of [rustdesk/hbb_common](https://github.com/rustdesk/hbb_common), the shared library
used by RustDesk's client and server (AGPL-3.0). Full credit for this library goes to the RustDesk
team and its contributors.

It's included here as the submodule dependency of
[rustdesk-managed-client](https://github.com/MAGA-Brad/rustdesk-managed-client), which adds a
small number of changes on top of the otherwise-unmodified upstream library: the device-passport
protocol messages and a log-filter fix (see Status below), and these config hooks: protecting the
2FA setting from being overwritten by generic config-sync writes, and (post-1.5 `libs/base` split)
managed clients always using `wss://`, never falling back to plaintext `ws://`. It also makes
"adaptive" the default desktop view style (upstream defaults to "original"). The two managed-mode
option keys (`preset-device-email`, `managed-local-input-priority-ms`) now live in
`libs/base::config::keys` alongside the rest of the client-level option keys, not here.

## Status (RDC build 34)

Build 34 adds device passports to the rendezvous protocol (`protos/rendezvous.proto`):

- `KxParams.managed_capabilities` (field 101): what else hbbs accepts on a connection; the value 1
  (lowest bit) means it takes a `DeviceAuth`. It's signed with the rest of `KxParams`, so it can't be stripped on the
  way, and clients that don't know the field ignore it.
- `DeviceAuth` (`RendezvousMessage` field 101): a managed device's passport and its identity key's
  signature over that connection's key exchange, sent right after the exchange when hbbs offers it.

Field number 101 keeps clear of upstream's numbering. The log filter also changed: the old
`webrtc-sctp=warn` directive never matched (Rust log targets use the crate name with underscores,
`webrtc_sctp`), so it now names `webrtc_sctp` and quiets a few other noisy WebRTC targets.

Earlier builds' encrypted rendezvous connections and sealed WebRTC signaling use protocol messages
upstream hbb_common already has (the signed `KeyExchange` with `KxParams`, `webrtc_sdp_offer` /
`webrtc_sdp_answer` and `IceCandidate`); the managed-only logic lives in the client, and the server
half in [rustdesk-managed-relay](https://github.com/MAGA-Brad/rustdesk-managed-relay).

The `managed-relay` branch is what the relay uses: upstream plus only the passport message above.
