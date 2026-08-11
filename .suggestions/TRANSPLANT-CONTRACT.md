# Recommended transplant contract

## 1. North-star ownership model

| Concern | Authority |
| --- | --- |
| Beep code and product documentation | `ubuntu-zombie/products/beep` |
| Shared family protocol source | A versioned family-contract location |
| Export rules and exporter implementation | Creator repository |
| Mirrored managed files | Export manifest |
| Downstream automation and community metadata | `ubuntu-beep` reserved namespaces |
| Product presentation | `ubuntu-beep` |
| Product releases | Prefer `ubuntu-beep`; otherwise declare one creator-repository authority |
| Historical lineage | One product provenance document plus machine-readable manifest |

“Untouched” should mean that no human edits exported product files in the
mirror. It should not mean that the mirror has no CI, provenance, governance,
or release controls.

## 2. Source-tree contract

The `products/beep/` directory should be standalone-first:

- commands work when the current directory is `products/beep/`;
- all local Markdown links resolve within the export set;
- product prose describes Beep, not its monorepo location;
- package commands produce the canonical root-first distribution;
- `PRODUCT.json` describes artifact/runtime identity, not repository location;
- shared files are declared imports rather than reached through `../..`;
- monorepo-only tests and commands are invoked by the monorepo root, not by
  changing product files; and
- no content transformation is required during export.

Monorepo context belongs in source-root maintainer documentation and CI.
Product context belongs inside `products/beep/`.

## 3. Canonical distribution layout

Adopt one root-first layout:

| Path in archive and mirror | Content |
| --- | --- |
| `PRODUCT.json`, `VERSION`, `README.md`, `LICENSE` | Product identity and entry documentation |
| `scripts/` | Lifecycle entry points |
| `payload/` | Runtime payload |
| `docs/` | Product/operator documentation |
| `tests/` | Product-owned tests |
| `family/schemas/` | Exact compatible protocol schemas |

The source release job and mirror `make package` must package the same staged
export, with the same file modes, ordering, timestamps, ownership metadata, and
compression settings. Given the same source revision and exporter version,
their SHA-256 digests must match.

The current `source_root` field combines two concepts. Replace it in a
versioned descriptor migration with:

- a product artifact root, normally `.`;
- a lifecycle path relative to that artifact root; and
- source-repository location kept only in export/provenance metadata.

Family-manager readers should support both the existing nested v1 archive and
the root-first successor during a bounded migration. Do not silently reinterpret
v1.

## 4. Source-owned exporter

Move standalone preparation from inline downstream YAML into a reviewed,
source-owned exporter used by:

1. creator-repository CI;
2. the Beep release workflow;
3. downstream sync verification; and
4. local maintainer checks.

The exporter should:

- resolve a full source commit before reading files;
- accept a declarative import profile for product source, schemas, and licence;
- reject absolute paths, traversal, special files, unsafe links, duplicate
  targets, and reserved target paths;
- preserve executable modes;
- validate product and protocol descriptors;
- generate product-ready docs without text substitution;
- produce a deterministic output tree and archive;
- emit a complete manifest;
- run product tests against the staged output; and
- fail if a transformation list is non-empty.

Reusable implementation is important. The current workflow duplicates policy
between extraction, mirroring, package inspection, and documentation
rewrites. One exporter gives every product the same safety properties.

## 5. Manifest version 3

The manifest should distinguish provenance, content, and ownership:

### Provenance

- schema version;
- product ID and product version;
- source repository;
- full source revision;
- resolved source ref;
- source path;
- exporter path, version, and content digest;
- generation run URL or ID; and
- release authority policy version.

### Imported content

- source path, target path, Git object ID, and object type for every import;
- path, mode, size, and SHA-256 for every managed file;
- deterministic output Git tree;
- canonical archive SHA-256; and
- applicable protocol/schema versions.

### Ownership

- managed path list;
- reserved downstream path list;
- package inclusion list;
- explicit collision policy; and
- prior manifest/tree from which the update was derived.

The source commit and output tree are immutable facts. Volatile metadata such
as generation time and workflow run ID must not alter the deterministic tree or
archive digest.

## 6. Downstream augmentation boundary

Reserve namespaces before accepting another sync:

| Namespace | Owner | Shipped? |
| --- | --- | --- |
| `.github/` | Downstream repository | No |
| `.suggestions/` | Downstream analysis | No |
| `.mirror/` or equivalent control metadata | Sync system | No |
| All manifest-managed product paths | Source exporter | Yes, if on package allow-list |

The exact names may differ, but the rules must not:

1. Source exports cannot create a reserved path.
2. Sync cannot overwrite an unmanaged path.
3. Stale cleanup can delete only paths owned by the previous valid manifest.
4. Packaging reads an allow-list from the export manifest; it never archives
   the checkout wholesale.
5. Installation copies the verified product file set, not repository metadata.
6. SBOM and provenance subjects cover product bytes only.

This allows issue templates, CI, policies, and analysis to augment the viewport
without mutating or contaminating Beep.

## 7. Documentation layers

Use four explicit layers:

### Product layer

README, vision, architecture, installation, configuration, operations,
security, privacy, testing, and troubleshooting should stand alone and speak
about Beep.

### Provenance layer

One `PROVENANCE.md` (or the existing `UPSTREAM.md` with a clearer title) should
contain:

- original lesson/bootstrap commit;
- current source repository, path, and revision;
- export method;
- release authority and signing identity;
- compatibility relationship to family contracts; and
- the rule that future source-product changes require review and a Beep version.

The README should link to this page once. Hiding provenance would be wrong;
repeating it throughout the product story is unnecessary.

### Protocol layer

Family schema names, `$id` values, actor enums, and catalogue rules are
compatibility interfaces. In the long term they should use a neutral,
versioned family namespace. Until then, keep exact values and document them as
protocol identity rather than Beep branding.

### Monorepo-maintainer layer

Relative source-root commands, shared conformance-suite paths, workflow
implementation, and product-generation instructions belong outside the export
tree in creator-repository maintainer documentation.

## 8. Release authority

### Recommended end state

Make `ubuntu-beep` the Beep release authority:

1. creator CI produces and validates a deterministic export;
2. mirror sync verifies and merges that export;
3. a downstream workflow triggered by an accepted `VERSION` change rebuilds
   the identical archive;
4. the downstream workflow publishes, attests, and signs `beep-v<VERSION>`;
5. release discovery and `beep-verify-release` trust the downstream repository
   and exact workflow identity; and
6. family catalogue entries support a per-product release repository.

This keeps code authorship in the monorepo while giving the standalone product
a truthful public identity and avoiding routine creator-brand references.

### Safe interim

If releases must remain in the creator repository:

- describe them as Beep-scoped releases from the canonical source repository;
- define repository, workflow path, issuer, and tag pattern in one reviewed
  release policy;
- make runtime discovery, catalogue validation, docs, and verifier consume that
  policy;
- link to the policy from product docs rather than repeating repository names;
  and
- do not claim that the mirror publishes its own GitHub releases.

Never accept arbitrary signing identities merely to make forks convenient.
Changing a trusted repository or OIDC workflow identity is a security migration
that requires an explicit versioned policy and review.

## 9. Sync and review flow

The target flow is:

1. A Beep-specific source change passes monorepo tests and standalone export
   tests at a full commit SHA.
2. Creator CI creates one deterministic export manifest and archive.
3. A repository dispatch requests downstream reconciliation; a schedule remains
   only as recovery.
4. Downstream automation verifies source revision, exporter digest, output tree,
   archive digest, and signature/attestation if used.
5. Automation checks reserved paths and unmanaged-file collisions.
6. A bot opens or refreshes one sync pull request containing the exact managed
   diff and a provenance summary.
7. Independent downstream CI validates the candidate without network-dependent
   mutation.
8. Merge records source revision and output tree.
9. If `VERSION` changed, the selected release authority publishes exactly those
   bytes.

This provides traceability without making the creator repository the subject of
every product-facing commit and document. A routine commit subject can be
“Sync Beep source snapshot” while its body and manifest retain the full source
URL and revision.

## 10. Required automated invariants

The following should be executable policy:

- no source-brand term in product-facing docs outside an allow-listed
  provenance, protocol, release-policy, security, or compatibility context;
- no exact-text rewrite in a successful export;
- no unmanaged or reserved file in an archive;
- no managed/unmanaged target collision;
- no mutable normative contract link;
- no source revision shorter than 40 hexadecimal characters in provenance;
- no difference between source-built and mirror-built archive digests;
- no broken relative documentation link;
- no release whose version, tag, manifest, artifact, SBOM, and attestation
  disagree;
- no direct default-branch publication without all required checks; and
- no weakening of schema or signing identity during a branding cleanup.
