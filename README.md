# Remnawave dual-core fork

This repository is the deployment and maintenance guide for the Remnawave fork maintained under
the GitHub account [`Cd1s`](https://github.com/Cd1s).

The fork keeps Xray compatibility and adds:

- per-config-profile core selection (`xray` or `singbox`);
- sing-box 1.13.14 lifecycle management in Remnawave Node;
- AnyTLS inbounds, users, traffic statistics, subscriptions, and frontend editing;
- tested upstream synchronization;
- multi-architecture GHCR images.

## Repositories and images

| Component | Source | Image |
| --- | --- | --- |
| Backend | `Cd1s/backend:singbox` | `ghcr.io/cd1s/remnawave-backend:singbox` |
| Frontend | `Cd1s/frontend:singbox` | Built into the backend image |
| Node | `Cd1s/node:singbox` | `ghcr.io/cd1s/remnawave-node:singbox` |

For repeatable deployments, pin an OCI digest. `sha-<40-character-commit>` tags identify the source
commit, but a CI rebuild of the same commit can update the tag's OCI index because build metadata
and provenance are regenerated.

The immutable images validated in an isolated test environment are:

- `ghcr.io/cd1s/remnawave-backend@sha256:635a7e4a25c8234cafb7b2eb501a5c7e7bd569265d213cbcc80a93b483b4bd43`
- `ghcr.io/cd1s/remnawave-node@sha256:2667c2e11f3b1c38cd4a35f8dcca115fa31de32d9797cb5aa1f17ee3ed93318e`

## Compatibility

Existing config profiles default to Xray. Creating or editing an Xray profile follows the original
Remnawave path. A profile only uses sing-box when its `coreType` is explicitly set to `singbox`.

AnyTLS credentials reuse each user's `vlessUuid`; the numeric Remnawave user ID is sent as the
sing-box user name so traffic can be attributed without changing the existing database model.

## Quick start

1. Deploy the panel stack from [`compose/panel.example.yml`](compose/panel.example.yml).
2. Register the initial administrator.
3. Obtain a Node `SECRET_KEY` from the panel.
4. Deploy the Node from [`compose/node.example.yml`](compose/node.example.yml).
5. Create a sing-box config profile and select the AnyTLS inbound for the Node.

Do not reuse the example passwords, domains, certificates, or Node secret.

## Documentation

- [Architecture](docs/architecture.md)
- [Isolated validation](docs/isolated-validation.md)
- [Production migration and rollback](docs/production-migration.md)

## Upstream policy

Each fork has a scheduled workflow that merges the corresponding `remnawave/*` `main` branch,
runs its validation suite, and only pushes the tested merge. Backend and Node workflows then
publish amd64 and arm64 images.

An upstream conflict or failed validation stops the synchronization before the fork branch is
updated.
