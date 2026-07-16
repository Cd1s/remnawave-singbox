# Custom feature registry

This file is the inventory of behavior carried by the fork. Update it whenever a feature is added,
removed, moved, or replaced by upstream functionality.

The registry is intentionally behavior-oriented. File paths may move during upstream merges, but
the invariants and ownership must remain explicit.

## Active features

| ID | Feature | Owners | Compatibility invariants | Required validation |
| --- | --- | --- | --- | --- |
| `CORE-001` | Per-profile core selection (`xray` or `singbox`) | Backend, frontend, Node | Xray is the default; existing profiles are not converted; legacy Node routes and fields remain usable | Backend build and seed validation; frontend create/edit test; Xray and sing-box Node startup |
| `CONFIG-001` | Native sing-box config storage and validation | Backend, frontend | Native snake_case keys and complete inbound JSON survive API/database round trips; Xray validation is unchanged | Config round-trip test; frontend schema/build; isolated profile startup |
| `NODE-001` | Dual-core Node lifecycle | Node, backend | Only the selected managed core runs; Xray behavior remains intact; invalid sing-box config does not replace the active config | Node build/trace; Xray canary; sing-box config check and real transfer |
| `USER-001` | sing-box user synchronization | Backend, Node | Add/remove operations preserve unrelated users and inbounds; established user identity mapping remains stable | User count transition test; disable/remove test; restart and reload test |
| `STATS-001` | sing-box traffic attribution and reload accumulation | Node, backend | Traffic is attributed to the correct Remnawave user; reloads neither lose nor double-count collected traffic | Measured transfer; collector update; traffic reset and reload tests |
| `ANYTLS-001` | AnyTLS inbound and client support | Backend, frontend, Node | TLS is required; AnyTLS is not emitted as Xray JSON; unsupported clients are not falsely advertised | Real AnyTLS transfer; config editor validation; user and traffic tests |
| `SUB-001` | AnyTLS sing-box JSON and Shadowrocket URI output | Backend | Credentials and server fields match the prepared Node config; certificate verification behavior is represented correctly; existing subscription formats remain unchanged | `validate:shadowrocket-anytls`; sing-box JSON inspection; real client transfer |
| `COMPAT-001` | Backward-compatible backend/Node wire extension | Backend, Node | Existing routes and payload fields remain accepted; new fields are optional or have safe fallbacks | Mixed-version contract review; Xray health/start/user tests |
| `BUILD-001` | Multi-architecture fork images | Backend, Node | Images publish for amd64 and arm64; backend embeds a resolved frontend commit; releases record immutable digests | CI build; manifest inspection; isolated deployment by digest |
| `SYNC-001` | Tested automatic upstream synchronization | Backend, frontend, Node | A conflict or failed validation never updates the maintained branch; custom validators cannot be skipped to force a merge | Scheduled/manual workflow; repair runbook when failing |

## Adding a feature

Add a row before or with the first implementation commit and specify:

- a stable feature ID;
- behavior, not only file names;
- owning repositories;
- invariants that an upstream merge must preserve;
- concrete validation that can fail when compatibility is broken.

If the feature changes a shared API or runtime command, update the relevant repository
`AGENTS.md` files in the same change.

## Upstream replacement lifecycle

When upstream implements overlapping behavior:

1. Mark the registry row as `migration pending`; do not delete it immediately.
2. Compare data models, API contracts, runtime semantics, and client output.
3. Add a migration or adapter from the fork representation to the upstream representation.
4. Validate existing Xray and sing-box installations.
5. Remove the custom implementation only after the upstream path passes the same required
   validation.
6. Mark the row `retired`, recording the upstream replacement and the last fork release that used
   the custom path.

The goal is not to preserve custom code forever. The goal is to preserve user-visible behavior and
safe upgrade paths.
