# Project map

This document tells maintainers and AI agents where a change belongs. The project intentionally
uses separate repositories so upstream Remnawave can continue to be merged with a small,
auditable fork delta.

## Repository responsibilities

| Repository | Upstream relationship | Owns | Does not own |
| --- | --- | --- | --- |
| `Cd1s/remnawave-panel:singbox` | Fork of `remnawave/panel:main` | Public project entry, fork documentation, unified releases, links to maintained components | Backend, frontend, or Node runtime source |
| `Cd1s/remnawave-backend:singbox` | Fork of `remnawave/backend:main` | API, database, config profiles, Node orchestration, user preparation, subscriptions, traffic attribution, backend image assembly | Browser presentation, core processes |
| `Cd1s/remnawave-frontend:singbox` | Fork of `remnawave/frontend:main` | Core-aware UI, config editor schema, profile and Node presentation | API semantics, database migrations, proxy runtime |
| `Cd1s/remnawave-node:singbox` | Fork of `remnawave/node:main` | Xray/sing-box processes, config activation, dynamic users, health, versions, traffic collection, Node image | Panel database, subscriptions, browser UI |
| `Cd1s/remnawave-singbox:main` | Coordination repository; no Remnawave upstream | Architecture, feature registry, maintenance policy, validation evidence, public deployment examples | Product runtime source |

## Runtime relationship

```text
Frontend
   │ API contracts and core-aware editing
   ▼
Backend ── prepared config, users, coreType ──► Node
   │                                           │
   │ subscriptions and traffic attribution     ├─ Xray
   ▼                                           └─ sing-box / AnyTLS
Clients
```

The backend is the contract center. A frontend change must not invent backend behavior, and a Node
change must not silently reinterpret backend commands.

## Branches and artifacts

The maintained fork branch is `singbox` in backend, frontend, and Node.

- Backend image: `ghcr.io/cd1s/remnawave-backend:singbox`
- Node image: `ghcr.io/cd1s/remnawave-node:singbox`
- Frontend: built into the backend image from an explicitly resolved frontend commit

Mutable tags are convenient for CI, but releases and production validation must record immutable
OCI digests.

## Change routing

| Requested change | Primary repository | Also inspect |
| --- | --- | --- |
| New core type or profile field | Backend | Frontend, Node |
| New sing-box inbound or protocol | Backend and Node | Frontend schema, subscription generators |
| Editor support only for an already supported field | Frontend | Backend contract/schema |
| Core process, reload, user, health, or stats behavior | Node | Backend commands and collectors |
| Subscription URI or client JSON | Backend | Client compatibility and isolated transfer test |
| Database migration | Backend | Rollback guide and older Xray data |
| Automatic upstream synchronization | The affected fork | This repository's maintenance guide |
| Deployment procedure or validation evidence | Coordination repository | Exact source commits and image digests |

## Coordinated release order

For a feature spanning all repositories:

1. Define the backend/Node contract and update the feature registry.
2. Implement and validate the Node capability.
3. Implement and validate the frontend against the intended backend contract.
4. Implement or finalize the backend behavior.
5. Build the backend image with the exact tested frontend commit.
6. Run isolated full-stack validation using immutable backend and Node image digests.
7. Record evidence here before production rollout.

This order prevents the backend image from accidentally embedding an older frontend and prevents a
new backend command from being enabled before a compatible Node image exists.
