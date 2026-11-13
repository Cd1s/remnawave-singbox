# Architecture

## Core selection

`config_profiles.core_type` stores either `xray` or `singbox` and defaults to `xray`. The backend
uses a common core-config interface for parsing, inbound discovery, user injection, certificate
processing, snippet replacement, and stable hashing.

When a Node is started, the backend includes `coreType` while retaining the original
`/node/xray/start` route and `xrayConfig` field for wire compatibility. The Node dispatches the
request to the selected core service.

## Node runtime

The Node image contains both:

- Xray 26.6.27;
- sing-box 1.14.4 built with V2Ray API, Clash API, QUIC, and uTLS tags.

Only one managed core is active for a profile. sing-box configurations are written atomically,
validated with `sing-box check`, and then applied through the s6-managed process.

## User synchronization

sing-box 1.14 does not expose a stable API for dynamically managing every supported inbound user.
The adapter therefore:

1. mutates the managed configuration;
2. validates it;
3. reloads sing-box;
4. preserves already collected traffic in an accumulator.

This supports initial bulk synchronization plus individual add/remove operations without losing
traffic already observed before a reload.

## Managed inbound mapping

Remnawave manages users for these sing-box inbound types; every other type is passed through
unchanged. The user `name` is always the Remnawave numeric user ID so v2ray_api traffic statistics
can be attributed.

| sing-box inbound | User credentials | Requirements and client output |
| --- | --- | --- |
| `anytls` | `password` = `vlessUuid` | TLS required. sing-box JSON, Mihomo, Shadowrocket `anytls://`; not Xray JSON |
| `vless` | `uuid` = `vlessUuid`; `flow` = `xtls-rprx-vision` for raw TCP with TLS or Reality and without transport or multiplex | TLS, Reality (public key derived from `tls.reality.private_key`, first `short_id`) or plain; `ws`, `grpc`, `httpupgrade` transports. Xray VLESS route IDs are not applied |
| `trojan` | `password` = `trojanPassword` | Same TLS/Reality/transport handling as VLESS |
| `shadowsocks` | `2022-blake3-aes-256-gcm`: `password` = base64(`ssPassword`), client password `<server key>:<user key>`; AEAD methods: `password` = `ssPassword` | Inbound `password` must be a base64 32-byte server key for SS2022. Users whose `ssPassword` is not 32 bytes are skipped for SS2022 |
| `hysteria2` | `password` = `vlessUuid` | TLS required. `obfs` (salamander or gecko) is published to clients; host `finalMask` keys override it |
| `tuic` | `uuid` and `password` = `vlessUuid` | TLS required. `congestion_control` and `zero_rtt_handshake` are published; set `tls.alpn: ["h3"]` for Stash. Not Xray JSON or Clash |

Not supported by sing-box and therefore unavailable on sing-box profiles: XHTTP, mKCP, and the
`2022-blake3-chacha20-poly1305`/`2022-blake3-aes-128-gcm` Shadowsocks methods (no multi-user
support, or 16-byte user keys). Hosts on `http` or `quic` V2Ray transports are omitted from
subscriptions because no client format represents them.

An empty Shadowsocks inbound would fall back to single-user mode keyed by the published SS2022
server key, so the Node writes a random, unpublished placeholder user in that case.

Each sing-box reload drops live connections. QUIC clients (TUIC in particular) can need several
seconds and a retry to reconnect after a user change on the same Node.

Upgrade note: before Hysteria2 and TUIC were managed, such inbounds were passed to every Node with
their hand-written users. They are now Remnawave inbounds: users are replaced by Remnawave users,
and the inbound is only sent to Nodes where it is enabled (and only reaches users through internal
squads), like any other managed inbound.

## Compatibility boundary

The original Xray routes, response fields, profile behavior, and status fallbacks remain present.
Existing rows migrate to `xray`; no profile is silently converted to sing-box.
