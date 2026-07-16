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
- sing-box 1.13.14 built with V2Ray API, Clash API, QUIC, and uTLS tags.

Only one managed core is active for a profile. sing-box configurations are written atomically,
validated with `sing-box check`, and then applied through the s6-managed process.

## User synchronization

sing-box 1.13 does not expose a stable API for dynamically managing every supported inbound user.
The adapter therefore:

1. mutates the managed configuration;
2. validates it;
3. reloads sing-box;
4. preserves already collected traffic in an accumulator.

This supports initial bulk synchronization plus individual add/remove operations without losing
traffic already observed before a reload.

## AnyTLS mapping

For an AnyTLS inbound:

- sing-box `name` = Remnawave numeric user ID;
- sing-box `password` = Remnawave `vlessUuid`.

The same password is emitted in SINGBOX subscription JSON. Formats that have no standardized
AnyTLS representation are skipped instead of emitting a misleading URI.

## Compatibility boundary

The original Xray routes, response fields, profile behavior, and status fallbacks remain present.
Existing rows migrate to `xray`; no profile is silently converted to sing-box.
