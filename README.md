# hbb_common (sanitized fork)

This is a fork of [rustdesk/hbb_common](https://github.com/rustdesk/hbb_common), the shared library
used by RustDesk's client and server (AGPL-3.0). Full credit for this library goes to the RustDesk
team and its contributors.

It's included here as the submodule dependency of
[rustdesk-managed-client](https://github.com/MAGA-Brad/rustdesk-managed-client), which adds a
small number of config hooks on top of the otherwise-unmodified upstream library: protecting the
2FA setting from being overwritten by generic config-sync writes, and (post-1.5 `libs/base` split)
managed clients always using `wss://`, never falling back to plaintext `ws://`. It also makes
"adaptive" the default desktop view style (upstream defaults to "original"). The two managed-mode
option keys (`preset-device-email`, `managed-local-input-priority-ms`) now live in
`libs/base::config::keys` alongside the rest of the client-level option keys, not here.
