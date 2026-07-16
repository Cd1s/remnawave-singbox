# AI maintainer guide

## Repository role

This repository is the coordination and maintenance source of truth for the Remnawave dual-core
fork. It contains public architecture, compatibility, validation, deployment examples, and
cross-repository maintenance policy. It does not contain the panel or Node runtime source.

The runtime repositories are:

- `Cd1s/backend`, branch `singbox`;
- `Cd1s/frontend`, branch `singbox`;
- `Cd1s/node`, branch `singbox`.

Read these files before making a cross-repository change:

1. `docs/project-map.md`
2. `docs/custom-feature-registry.md`
3. `docs/upstream-maintenance.md`
4. `docs/architecture.md`
5. `docs/isolated-validation.md`

## Responsibilities

Use this repository to record:

- which repository owns each custom feature;
- compatibility invariants that must survive upstream releases;
- the validation evidence and immutable image digests for a tested release;
- deployment and rollback procedures that do not expose real infrastructure;
- changes to the automatic upstream synchronization strategy.

Do not place product runtime logic here. Runtime changes belong in the owning fork and must be
registered in `docs/custom-feature-registry.md`.

## Maintenance rules

1. Keep all documentation free of credentials, private host aliases, provider names, production
   addresses, user identifiers, certificate fingerprints, and subscription URLs.
2. Do not treat a successful build as full compatibility evidence. Cross-repository behavior
   requires isolated integration validation.
3. Never update a validated image digest until a new immutable digest has passed the documented
   test matrix.
4. When a custom feature changes, update its owner, contracts, invariants, and tests in the feature
   registry in the same maintenance cycle.
5. When automatic synchronization fails, follow `docs/upstream-maintenance.md`; do not advise
   force-pushing or deleting fork functionality.
6. Examples must use placeholders and remain safe to publish.

## Validation

For documentation-only changes:

```bash
cp compose/panel.env.example compose/panel.env
docker compose -f compose/panel.example.yml config
docker compose -f compose/node.example.yml config
! grep -RInE '[[:blank:]]+$' --include='*.md' --include='*.yml' --include='*.example' .
```

Also run the privacy checks in `.github/workflows/docs-ci.yml`.

## Definition of done

A coordination change is complete when the repository map and feature registry agree with the
actual fork branches, links work, examples validate, and no private deployment detail is present.
