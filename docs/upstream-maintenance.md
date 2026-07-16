# Upstream synchronization and custom iteration

This guide is the operating procedure for keeping the fork compatible with future official
Remnawave releases.

## Automation model

Each fork has a scheduled `upstream-sync.yml` workflow. It:

1. checks out the maintained `singbox` branch;
2. fetches the matching official `remnawave/*` `main` branch;
3. merges upstream in the temporary runner checkout;
4. runs repository-specific validation;
5. pushes the merge only after all required validation passes;
6. publishes images where applicable.

Therefore, a merge conflict, build failure, or regression test failure must leave
`origin/singbox` unchanged. A red synchronization workflow is a request for maintenance, not
permission to bypass validation.

## Daily failure triage

Classify a failed workflow before editing code:

| Failure | Meaning | First action |
| --- | --- | --- |
| Merge conflict | Upstream and fork changed overlapping code | Reproduce the merge on a temporary branch and inspect both implementations |
| Dependency/install failure | Lockfile, runtime, registry, or action changed | Confirm whether the same failure exists on upstream before changing fork code |
| Compile/type failure | A contract, API, type, or library changed | Trace all cross-repository consumers; do not patch only the first error |
| Custom regression failure | The merge broke a fork invariant | Preserve the invariant or deliberately migrate it with registry updates |
| Image build failure | Dockerfile, binary source, architecture, or frontend assembly changed | Build each target architecture and verify exact source inputs |
| Publish-only failure | Code passed but artifact upload failed | Retry publication without creating a different source merge |

## Safe manual repair

Run the repair inside the affected repository:

```bash
git fetch origin singbox
git fetch upstream main
git switch -c repair/upstream-YYYYMMDD origin/singbox
git merge upstream/main
```

Then:

1. Read the upstream commits touching every conflict.
2. Compare `upstream/main...HEAD` so unrelated fork modifications are visible.
3. Resolve toward the new upstream structure, then reattach fork behavior through the narrowest
   adapter or extension point.
4. Run the affected repository's `AGENTS.md` validation gates.
5. Run cross-repository validation when a contract, config, Node command, subscription, migration,
   or runtime behavior changed.
6. Review the final diff against upstream, not only the conflict markers.
7. Merge the repair through the maintained branch's normal CI path.

Do not:

- force-push `singbox`;
- rebase away published fork merge history;
- reset the branch to upstream and reapply only obvious commits;
- delete a published database migration;
- disable a custom regression test without replacing its coverage;
- resolve generated contracts independently in backend, frontend, and Node;
- deploy an unvalidated moving image tag.

## Conflict priority areas

### Backend

Inspect config-profile entities and contracts, Prisma migrations, raw JSON handling, Node command
extensions, user preparation, profile startup, subscription resolution, and generators.

### Frontend

Inspect API contracts, profile create/update hooks, Monaco schema selection, config validation,
profile cards, and Node status/version presentation.

### Node

Inspect command schemas, Xray service changes, common core dispatch, handler and stats semantics,
s6 services, binary build arguments, and Docker architecture stages.

## Designing custom features for the next upstream release

Every new feature should be cheap to carry across an upstream merge.

### Use explicit boundaries

Put core-specific behavior behind factories, adapters, services, validators, or capability checks.
Avoid scattering `if singbox` conditions through unrelated upstream code.

### Keep changes additive

Prefer optional contract fields, safe defaults, new modules, new migrations, and new tests. Existing
Xray behavior should remain the fallback until an operator explicitly selects another core.

### Minimize overlap

Do not reformat or rewrite a large upstream file to add a small feature. The number of touched
upstream lines is part of the maintenance cost.

### Test invariants, not implementations

Tests should assert that profiles survive round trips, old Xray behavior remains valid, users and
traffic survive reloads, subscriptions are usable, and failed validation does not replace a
working config. Such tests remain valuable even when upstream reorganizes files.

### Coordinate contracts

For a shared change, define:

- backend API and database representation;
- Node command and capability behavior;
- frontend availability and fallback;
- client subscription representation;
- mixed-version behavior during rollout.

Record these in `docs/custom-feature-registry.md`.

## Validation levels

1. **Repository validation:** compile, typecheck, lint/format, local regression scripts.
2. **Cross-repository contract validation:** matching backend, frontend, and Node branches.
3. **Isolated runtime validation:** clean database, temporary panel and Node, real client transfer,
   user updates, traffic collection, cleanup.
4. **Canary validation:** one non-critical Node/profile while existing Xray Nodes remain connected.
5. **Production release:** immutable digests, backup, observation window, and documented rollback.

Passing a lower level does not replace a higher level when the change affects runtime behavior.

## Release record

For each validated release, record:

- upstream commit included by each fork;
- fork source commit for backend, frontend, and Node;
- backend and Node immutable OCI digests;
- core versions;
- database migration status;
- validation scenarios and result;
- known compatibility limitations;
- rollback artifact or previous digest.

Never place real infrastructure identifiers, credentials, subscription URLs, or user data in the
public release record.
