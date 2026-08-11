# Current-state assessment

## 1. Repository model

The repository contains 120 tracked files:

- 118 files are declared as mirror-managed by
  [`.github/upstream-manifest.json`](../.github/upstream-manifest.json);
- the manifest itself is generated distribution metadata; and
- [`.github/workflows/sync-upstream.yml`](../.github/workflows/sync-upstream.yml)
  is downstream-owned replication automation.

The manifest imports three source objects:

| Source | Downstream target | Purpose |
| --- | --- | --- |
| `products/beep` | repository root | Product source, docs, tests, and lifecycle |
| `family/schemas` | `family/schemas` | Shared lifecycle contracts |
| `LICENSE` | `LICENSE` | Applicable licence |

The sync correctly leaves unlisted paths in place. However, it does not reserve
an augmentation namespace or reject a new upstream file that collides with an
existing downstream-owned file.

## 2. What is already strong

| Area | Verified strength |
| --- | --- |
| Product separation | `PRODUCT.json` gives Beep its own account, paths, port, cookie, units, environment prefix, marker, and commands. |
| Runtime safety | The docs and implementation consistently declare root-equivalent risk, loopback scope, approval boundaries, recovery limits, and terminal lifecycle state. |
| Source validation | Sync validates the product identity and version, rejects product-owned `.github` automation, and runs the canonical lint, test, and package targets. |
| Export safety | Git object types, modes, paths, duplicates, parent collisions, and symlink targets are checked before materialisation. |
| Reproducible source tree | Both sync jobs recreate the export and compare the resulting Git tree ID before publication. |
| Stale-file safety | Only files recorded by the prior manifest can be removed, and recursive directory deletion is refused. |
| Standalone validation | The transformed tree is linted, tested, packaged, and inspected for unsafe archive members. |
| Workflow security | Default permissions are empty; write permission exists only in the publish job; concurrency and timeouts are bounded. |
| Documentation | Architecture, security, privacy, operations, configuration, installation, recovery, testing, and release concerns are covered. |
| Release design | The source release workflow pins actions by commit and creates checksums, an SPDX SBOM, provenance, and keyless cosign material. |

These controls should be preserved. The recommendation is to move them into a
smaller reusable exporter, not weaken them.

## 3. Material findings

### A. Two artifacts use one name but have different layouts — critical

The source
[`products/beep/Makefile`](https://github.com/japer-technology/ubuntu-zombie/blob/fd8b92dc550a48dbadbabd958bfe0b62b91dfb79/products/beep/Makefile)
packages:

- `products/beep/`;
- `family/schemas/`; and
- `LICENSE`.

The mirror [Makefile](../Makefile) instead archives the repository root. Both
outputs are named `beep-<VERSION>.tar.gz`.

That split is already visible in the mirror:

- [installation instructions](../docs/INSTALLATION.md) invoke
  `scripts/manage.sh`;
- [upgrade instructions](../docs/UPGRADING.md) invoke
  `products/beep/scripts/manage.sh`;
- [release verification](../docs/RELEASE.md) inspects
  `products/beep/VERSION`; and
- [`PRODUCT.json`](../PRODUCT.json) still declares
  `"source_root": "products/beep"` despite residing at the mirror root.

Consequences include ambiguous operator commands, different archive digests,
different SBOM paths, and no guarantee that the source release and mirror
package represent the same bytes.

**Required outcome:** choose one canonical archive topology. A root-first Beep
distribution is the natural fit for this repository; source CI should create it
from an export staging tree and publish that exact artifact.

### B. Downstream additions leak into packages and installations — critical

The mirror package target archives `.` and excludes only version-control data,
`.github`, `dist`, bytecode, and Python caches. It does not exclude
`.suggestions` or any future downstream community/governance files.

The lifecycle deployment also copies the source root while excluding only
`dist`, `__pycache__`, and `*.pyc`. Consequently, an install from this checkout
can copy downstream-only material into `/opt/beep/product`.

This is an allow-list problem, not a request for a longer deny-list.

**Required outcome:** packages and installations must consume only the
exporter's managed-file inventory. Repository augmentations must be
structurally unable to enter product artifacts.

### C. Export correctness depends on prose remaining unchanged — high

The `Prepare standalone distribution` step contains 15 exact replacement calls
covering:

- Makefile roots, tests, and package commands;
- README links and commands;
- provenance wording;
- security reporting links;
- installation paths;
- testing claims and matrices; and
- release-workflow descriptions.

Each replacement checks its expected occurrence count, which prevents silent
bad output. It does not prevent routine source editing from breaking the
replication pipeline, nor does it prove that a semantically equivalent but
incorrect statement was not introduced elsewhere.

**Required outcome:** author product files in standalone form. Put monorepo-only
commands, links, and release implementation notes outside the exported product
tree. The normal successful export should have zero content rewrites.

### D. Provenance, protocol, and product prose are mixed — high

A case-insensitive inventory found 67 occurrences of `Ubuntu Zombie` or
`ubuntu-zombie` across 26 tracked files:

| Class | Occurrences | Treatment |
| --- | ---: | --- |
| Product-facing README and docs | 24 | Reduce to one provenance link plus genuinely necessary co-installation/security references. |
| Shared family schemas | 21 | Preserve until a versioned protocol migration; never bulk-rename. |
| Sync workflow and manifest | 13 | Keep machine-readable repository identity; make routine workflow titles and commit subjects product-first. |
| Runtime, verifier, catalogue, and parity test | 9 | Replace with an explicit release/family authority policy, or retain as deliberate control-plane configuration. |

In addition, the browser template contains 134 internal identifiers with a
legacy `uz` prefix. They are not operator-visible branding, but they increase
copy-derived maintenance debt and make future extraction harder.

The references are not equivalent:

1. **Must remain or be version-migrated:** schema `$id` values, protocol enum
   values, signed workflow identity, source revision, and historical
   provenance.
2. **May remain in one technical policy:** release repository, catalogue
   authority, vulnerability-reporting destination, and root-peer test target.
3. **Should disappear from ordinary product prose:** comparisons in the README,
   vision, configuration, troubleshooting, and upgrade introductions.
4. **Should be renamed opportunistically:** internal `uz...` JavaScript
   identifiers, with behaviour-preserving tests.

### E. The manifest omits the source commit — high

The version-2 manifest records source repository, imported object IDs, managed
files, and output tree. It does not record the full source commit SHA. That SHA
exists only in the sync commit message and workflow summary.

Object IDs prove imported content but do not give tools a single reviewable
source snapshot, source URL, branch/ref context, or generator identity.

**Required outcome:** manifest version 3 should include the full source
revision, source ref, exporter version/digest, imported object IDs, managed-file
hashes and modes, output tree, and—when applicable—the canonical archive
digest.

### F. The mirror has no continuous validation for local augmentation — high

The only workflow is scheduled or manually dispatched. It validates a sync,
but it does not run on pull requests or ordinary pushes. Downstream-owned
workflow, governance, or documentation changes therefore have no required
local gate.

**Required outcome:** add a separate downstream CI workflow for pull requests
and pushes. It should validate ownership boundaries, manifest integrity,
product tests, package contents, documentation links, secrets, and security
analysis without initiating a sync.

### G. Valid snapshots are pushed directly to the default branch — medium

The publish job commits and pushes after validation. It retries by rebasing on
the latest default branch, but there is no reviewable sync pull request and no
explicit policy check for a collision with an unmanaged target file.

**Required outcome:** open or update one bot-authored sync pull request. Require
the downstream gates and a generated provenance/diff summary before merge.
Retain a documented emergency direct-publish path only if operationally
necessary.

### H. Product independence and release authority are unclear — medium

The docs describe an independent GitHub release, while the release workflow,
release API lookup, cosign certificate identity, attestation repository, family
catalogue, and release assets belong to the creator repository. The mirror has
no release workflow.

Product-scoped versioning inside a monorepo is valid, but it is not the same as
an independently published repository.

**Required outcome:** select exactly one release authority and describe it
accurately. The recommended end state is for the mirror to publish and sign
Beep releases after accepting a verified source export. The safe interim is to
retain creator-repository publication but centralise that fact in one release
policy and one provenance page.

### I. Mutable source references remain in normative material — medium

README contract links and schema IDs point at `main`. The mirror itself is
pinned to a source commit, but a reader following a normative link can receive
new content that was not part of the mirrored snapshot.

Schema `$id` is an identity and must not be casually changed. Normative
documentation links, however, should resolve to a contract version or the
recorded source revision.

**Required outcome:** version normative contracts and export the applicable
contract revision, or generate commit-pinned links from manifest metadata.

## 4. Product-facing content policy

| Context | Recommended wording |
| --- | --- |
| README opening | Define Beep by user value, supported platform, authority, and safety status. Do not define it as a duplicate of another product. |
| Architecture | Describe other root-capable products generically. Name a peer only in a concrete compatibility or threat scenario. |
| Security | Keep one private-reporting destination and one explicit warning that root peers are not isolated. |
| Testing | Name the exact peer in the co-installation matrix because it is a measurable gate. |
| Configuration and troubleshooting | Say “another product” or “sibling credential/cookie” unless a literal protocol value is required. |
| Provenance | Record creator repository, bootstrap commit, mirrored source commit, copied lesson history, and release authority in full. |
| Schemas and signed identities | Preserve exact values until a compatibility-reviewed migration exists. |

This policy removes repetition without hiding lineage or weakening trust.

## 5. Maturity summary

| Dimension | Assessment |
| --- | --- |
| Product architecture | Strong |
| Runtime namespace independence | Strong |
| Security documentation | Strong |
| Source and standalone tests | Strong, with host gates openly incomplete |
| Deterministic source-tree export | Strong |
| Package identity across repositories | Needs immediate correction |
| Downstream augmentation isolation | Needs immediate correction |
| Release-authority clarity | Needs a decision |
| Product-first presentation | Needs consolidation |
| Mirror governance and continuous CI | Missing |

The repository is best described as a well-engineered product behind a
transitional distribution boundary—not as a product that needs to be copied or
renamed again.
