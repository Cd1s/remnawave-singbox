# Isolated validation

The validation environment was intentionally separated from every existing service:

- unique Podman container, network, and volume names;
- loopback-only management ports;
- a dedicated AnyTLS test port;
- generated short-lived certificates and credentials;
- automatic cleanup on success or failure;
- existing service state and file fingerprints checked before and after.

## Node adapter test

The Node-only test verifies:

- sing-box 1.13.14 startup and health reporting;
- real AnyTLS traffic;
- per-user traffic attribution;
- traffic preservation across a reload;
- dynamic user count transitions `1 → 2 → 1`;
- traffic reset semantics.

## Full-stack test

The full-stack test starts temporary PostgreSQL, Valkey, backend, Node, target, and client
containers. It then exercises the public backend API:

1. register the temporary administrator;
2. create a sing-box AnyTLS profile and internal squad;
3. inject the first user during Node startup;
4. create a second user and observe dynamic Node synchronization;
5. generate a SINGBOX subscription containing an AnyTLS outbound;
6. use that outbound for a real proxied HTTP transfer;
7. wait for Remnawave's scheduled statistics collector to update user traffic;
8. disable the second user and observe removal from the Node.

Exact image commits and final evidence are recorded after each successful run.

## Validated baseline

The isolated full-stack run completed successfully on 2026-07-16 using these immutable images:

- backend:
  `ghcr.io/cd1s/remnawave-backend@sha256:635a7e4a25c8234cafb7b2eb501a5c7e7bd569265d213cbcc80a93b483b4bd43`
  from source commit `1b25ac45ed6d591a378c1155f8c651d347d8843e`;
- Node:
  `ghcr.io/cd1s/remnawave-node@sha256:2667c2e11f3b1c38cd4a35f8dcca115fa31de32d9797cb5aa1f17ee3ed93318e`
  from source commit `bf945ec517ef6bea5256ca861fa05c4c596e897b`;
- core: sing-box 1.13.14;
- protocol: AnyTLS;
- measured traffic recorded by the temporary panel: 229 bytes.

Both images expose public OCI indexes containing `linux/amd64` and `linux/arm64` manifests.
Their source-SHA tags were used during the run. The digests above remain the authoritative
validation references because later CI rebuilds can move a tag without changing source code.

The successful run verified:

- Node connected with `coreType=singbox` and reported sing-box 1.13.14;
- the initial user was present in the generated sing-box configuration;
- a second user was added dynamically;
- the SINGBOX subscription contained a valid AnyTLS outbound;
- that outbound completed a real proxied HTTP transfer;
- Remnawave's scheduled collector wrote non-zero user traffic;
- disabling the second user removed it dynamically;
- native sing-box JSON fields, including `listen_port`, survived API storage and retrieval unchanged.

After cleanup:

- the pre-existing sing-box service remained active with the same process identity;
- its binary and configuration fingerprints were unchanged;
- every temporary loopback and test port was released;
- no temporary validation containers remained.

Host aliases, addresses, provider names, production fingerprints, and other infrastructure
identifiers are intentionally excluded from this public report.
