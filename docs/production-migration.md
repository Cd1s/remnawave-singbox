# Production migration and rollback

No production action should be taken until it is explicitly approved.

## Before migration

1. Record the current backend, frontend, Node, Xray, and database versions.
2. Export the production Compose files and environment without publishing secrets.
3. Take and verify a PostgreSQL backup.
4. Record container image digests, Node assignments, profile UUIDs, and active inbounds.
5. Confirm spare loopback ports and disk space for a parallel canary.

## Recommended rollout

1. Pin the tested backend and Node OCI digests recorded in
   [the isolated validation report](isolated-validation.md).
2. Start the forked backend on parallel loopback ports against a restored database copy.
3. Verify login, existing Xray profiles, users, subscriptions, and node status.
4. Upgrade one non-production/canary Node while leaving its profile on Xray.
5. Verify that the forked Node still runs the Xray profile.
6. Create a separate sing-box profile; do not convert the production Xray profile in place.
7. Attach only the canary Node and test users to the sing-box profile.
8. Validate AnyTLS transfer, traffic accounting, subscription output, and dynamic user updates.
9. Move production traffic only after an observation window and another database backup.

## Fast rollback

If a sing-box profile fails but the forked panel remains healthy:

1. switch the affected Node back to its previous Xray profile;
2. restart only that Node from the panel;
3. verify Xray health and subscriptions.

If the forked backend fails:

1. stop the forked backend containers;
2. restore the previous backend image and Compose definition;
3. keep the database unless a migration failure requires the verified backup;
4. verify login, Xray nodes, users, and subscription responses.

The added `core_type` column defaults to `xray`; the migration is additive. Nevertheless, the
database backup remains the authoritative rollback point.

## Production guardrail

Do not restart or replace the current production Remnawave deployment, and do not repoint its
reverse proxy, until an operator explicitly approves the production change window.
