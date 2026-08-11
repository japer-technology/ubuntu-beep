# Implementation roadmap

## Decision gate: release ownership

Before changing URLs or prose, choose one:

- **Recommended:** `ubuntu-zombie` remains edit authority and `ubuntu-beep`
  becomes Beep's release/signing authority.
- **Interim:** `ubuntu-zombie` remains both edit and release authority, while
  `ubuntu-beep` is explicitly a verified source viewport.

Record the choice in a machine-readable release policy. Do not support both
authorities for the same version.

## Phase 0 — correct distribution ambiguity

Complete before the next public Beep release.

### Creator repository

- [ ] Choose root-first as the canonical extracted artifact layout.
- [ ] Build the release archive from an export staging tree rather than directly
      archiving `products/beep/`.
- [ ] Split repository source location from artifact root in a versioned product
      descriptor and family-contract migration.
- [ ] Make installation, upgrade, release, and verification instructions use
      the same extracted paths.
- [ ] Make archives reproducible, including order, timestamps, owner/group,
      modes, and compression metadata.

### Mirror repository

- [ ] Change packaging and lifecycle source-copy logic to consume a managed-file
      allow-list.
- [ ] Reserve downstream-only namespaces and exclude them from package, install,
      SBOM, and attestation inputs.
- [ ] Add a test proving `.github/`, `.suggestions/`, mirror metadata, and an
      arbitrary unmanaged fixture cannot enter the artifact.
- [ ] Fail sync when an upstream target would overwrite an unmanaged file.

### Exit criteria

- [ ] Source and mirror builds of one source revision have identical SHA-256.
- [ ] Extracted `PRODUCT.json`, `VERSION`, and `scripts/manage.sh` occupy one
      documented location.
- [ ] Every documented install/update command works against the canonical
      archive.
- [ ] Repository-only files are absent from the archive and installed product.

## Phase 1 — make the source folder export-ready

### Creator repository

- [ ] Move the exporter and its validation policy into source-controlled,
      reusable tooling.
- [ ] Add a declarative Beep export profile covering the product tree, exact
      family schema set, licence, reserved paths, and package allow-list.
- [ ] Rewrite `products/beep/` documentation to be product-first and valid from
      the product directory without downstream substitutions.
- [ ] Move monorepo-only commands, shared-suite locations, and workflow details
      to creator-repository maintainer docs.
- [ ] Export or version the normative Beep and lifecycle contracts so mirror
      readers do not follow mutable `main` content.
- [ ] Generate manifest version 3 with full source revision, exporter identity,
      per-file metadata, output tree, and archive digest.

### Tests

- [ ] Run the exporter twice at the same commit and compare tree and archive
      digests.
- [ ] Run lint, unit, integration, parity, link, and package-content checks
      against the staged export.
- [ ] Assert that the successful export reports zero content rewrites.
- [ ] Assert malformed paths, links, modes, duplicate targets, and reserved
      paths fail closed.

### Exit criteria

- [ ] Deleting all downstream text-replacement logic does not change intended
      product content.
- [ ] The source product directory and extracted export present the same
      commands and documentation.
- [ ] A reviewer can trace every output byte to an imported source object or the
      identified exporter.

## Phase 2 — make mirroring reviewable and event-driven

### Mirror automation

- [ ] Keep sync reconciliation separate from ordinary CI.
- [ ] Trigger reconciliation after accepted Beep source changes by repository
      dispatch; retain a scheduled drift check as fallback.
- [ ] Fetch and verify an exact source commit and exporter identity.
- [ ] Open or update a bot-authored sync pull request instead of pushing directly
      to the default branch.
- [ ] Include source revision, product version, old/new output trees, archive
      digest, changed paths, and test evidence in the pull-request summary.
- [ ] Require a clean re-run after rebasing rather than rebasing a validated
      commit and immediately pushing it.

### Downstream CI

- [ ] Run on every pull request and push.
- [ ] Validate manifest schema, managed-file hashes/modes, reserved paths, and
      ownership boundaries.
- [ ] Run existing lint and non-host tests against managed product content.
- [ ] Verify reproducible package contents and documentation links.
- [ ] Run secret scanning, dependency review, and CodeQL where applicable.
- [ ] Keep all default workflow permissions read-only; grant write only to the
      narrow sync/release jobs.

### Exit criteria

- [ ] A source update cannot reach the default branch without a visible diff and
      required checks.
- [ ] A downstream-only change cannot trigger source reconciliation.
- [ ] A source update cannot overwrite or ship downstream augmentation.
- [ ] Reconciliation is idempotent when source revision and exporter are
      unchanged.

## Phase 3 — separate Beep identity from provenance

### Product documentation

- [ ] Lead the README with Beep's purpose, risk, support status, and verified
      installation path.
- [ ] Consolidate bootstrap history, source location, and mirror mechanics into
      one provenance document.
- [ ] Keep exact peer names only in concrete co-installation, protocol,
      disclosure, and signing contexts.
- [ ] Replace mutable normative links with exported or version-pinned contracts.
- [ ] Correct “independent GitHub release” language to match the release-owner
      decision.

### Control plane

- [ ] Define release repository, tag pattern, workflow identity, OIDC issuer,
      catalogue authority, and schema set in reviewed versioned policy.
- [ ] Make release discovery, verifier defaults, family manager, and tests
      consume that policy rather than separate hard-coded strings.
- [ ] If the mirror becomes release authority, migrate the family catalogue to
      per-product repositories and update trust roots in one compatibility
      release.
- [ ] Plan a neutral, versioned namespace for future family schemas; preserve v1
      identifiers indefinitely for compatibility.

### Internal cleanup

- [ ] Rename the 134 legacy `uz...` browser identifiers in a mechanical,
      test-backed change.
- [ ] Keep historical commit messages and provenance immutable.
- [ ] Add a source-reference allow-list check rather than a global word ban.

### Exit criteria

- [ ] A new reader can understand Beep without first learning about its creator
      product.
- [ ] Lineage and signing authority remain discoverable in one click.
- [ ] No protocol, schema, catalogue, or signature check was weakened to improve
      presentation.

## Phase 4 — world-class repository operation

### Governance and contributor experience

- [ ] Add a generated-mirror contribution policy explaining where product
      changes, mirror automation changes, support reports, and vulnerabilities
      belong.
- [ ] Add a code of conduct, support policy, pull-request template, issue forms,
      CODEOWNERS, and a private vulnerability-reporting path appropriate to the
      selected ownership model.
- [ ] Configure repository description, topics, release link, and archived/read-
      only expectations consistently.
- [ ] Protect the default branch with required reviews/checks and disallow force
      pushes and deletion.

### Supply chain and maintenance

- [ ] Keep all third-party actions pinned to full commit SHAs and automate
      reviewed updates.
- [ ] Generate an SBOM from the canonical export and verify that its paths and
      subjects match the release manifest.
- [ ] Publish checksums, provenance, signatures, and test evidence beside every
      release from the single selected authority.
- [ ] Record the still-open Ubuntu 22.04/24.04 VM, co-installation,
      failure-injection, restore, and external-security-review gates without
      implying they passed.
- [ ] Add release rollback/revocation guidance and an auditable keyless-signing
      identity migration process.

### Quality

- [ ] Add Markdown/link validation and machine-readable schema validation.
- [ ] Add coverage reporting only after defining meaningful risk-based targets;
      do not substitute percentage for lifecycle and policy tests.
- [ ] Add fuzz/property tests first at untrusted parsers, policy classification,
      archive extraction, and family request boundaries.
- [ ] Publish a concise API/extension guide for tools, skills, HTTP endpoints,
      and stable error envelopes.

## Suggested issue breakdown

| Priority | Repository | Issue |
| --- | --- | --- |
| P0 | Creator | Produce one root-first deterministic Beep export and release artifact |
| P0 | Creator + mirror | Migrate descriptor/archive-root semantics with v1 compatibility |
| P0 | Mirror | Enforce managed-file packaging and reserved augmentation paths |
| P0 | Creator | Reconcile install, upgrade, release, and verification paths |
| P1 | Creator | Extract inline standalone preparation into reusable exporter |
| P1 | Creator | Make `products/beep/` docs standalone-first with zero rewrites |
| P1 | Mirror | Add manifest v3 verification and unmanaged-collision rejection |
| P1 | Mirror | Replace direct publication with checked sync pull requests |
| P1 | Both | Decide and migrate to one Beep release authority |
| P2 | Creator | Centralise release/family authority in versioned policy |
| P2 | Creator | Consolidate product lineage into one provenance document |
| P2 | Mirror | Add continuous CI, governance, and contribution routing |
| P3 | Creator | Version or neutralise future family protocol identities |
| P3 | Creator | Rename legacy browser identifiers mechanically |

## Final acceptance checklist

- [ ] The creator product folder is valid without downstream text edits.
- [ ] The manifest identifies the exact source commit and exporter.
- [ ] The mirror has an explicit managed/unmanaged ownership boundary.
- [ ] Source and mirror produce byte-identical canonical archives.
- [ ] Repository augmentation never enters product bytes.
- [ ] Exactly one repository/workflow identity publishes each Beep version.
- [ ] Runtime discovery and verification agree with that identity.
- [ ] Product docs are Beep-first and provenance remains complete.
- [ ] Normative contracts are immutable or version-pinned.
- [ ] Sync updates are reviewable, idempotent, and collision-safe.
- [ ] Existing non-host tests pass in source, staged export, and mirror.
- [ ] Host/security gates are recorded honestly.

## Avoid

- Do not bulk-replace creator names in schemas, actor enums, or certificate
  identities.
- Do not make trusted signing identity an unrestricted convenience setting.
- Do not maintain two hand-edited copies of Beep.
- Do not permit downstream runtime patches that the source exporter cannot
  reproduce.
- Do not package a repository checkout with a growing exclusion list.
- Do not call two different archive layouts the same release artifact.
- Do not hide source history; concentrate it.
