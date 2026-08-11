# Ubuntu Beep mirror recommendations

## Purpose

This folder is an evidence-based proposal for making `ubuntu-beep` a
product-first, read-only distribution view of the Beep source maintained in
`japer-technology/ubuntu-zombie`.

The product itself is already unusually strong: it has explicit trust
boundaries, isolated runtime namespaces, fail-closed lifecycle handling,
hermetic tests, guarded host tests, signed release design, and extensive
operator documentation. The main opportunity is not another rewrite of Beep.
It is a cleaner **source-to-distribution contract**.

## Analysis snapshot

| Item | Value |
| --- | --- |
| Downstream repository | `japer-technology/ubuntu-beep` |
| Downstream commit reviewed | `6ef6aeb5de420fc91770433c8ea59c3dd4cdff7f` |
| Mirrored source commit | `fd8b92dc550a48dbadbabd958bfe0b62b91dfb79` |
| Mirrored source path | `products/beep` |
| Manifest output tree | `a7aa7fbfb73b212244040aa648665cec3b98c634` |
| Analysis date | 2026-08-11 |

The source repository had advanced to
[`5415f87b8b20263f14779dd8f8b3550056f9a047`](https://github.com/japer-technology/ubuntu-zombie/commit/5415f87b8b20263f14779dd8f8b3550056f9a047)
when this analysis was completed. Findings about the mirrored product are
therefore pinned to the source commit recorded above, not inferred from a
moving `main` branch.

## Executive verdict

Keep the monorepo as the edit authority, but make `products/beep/` valid as a
standalone product **before** export. A downstream sync should copy and verify
an already-correct distribution; it should not reinterpret prose, paths, test
coverage, or package topology.

The current design has five material distribution risks:

1. The 872-line downstream sync workflow performs 15 exact-text rewrites to
   turn monorepo content into standalone content. This is fail-fast, but
   inherently coupled to wording and formatting.
2. The source and mirror produce different directory layouts under the same
   `beep-<VERSION>.tar.gz` name. The resulting mirror documentation contains
   both root-level and `products/beep/` command paths.
3. The mirror packages the entire checkout except a short deny-list. Any
   downstream augmentation, including this `.suggestions/` folder, can enter a
   source artifact and an installation unless explicitly excluded.
4. Product independence and repository independence are conflated. Release
   discovery, release verification, family catalogues, schema identities, and
   publication still name the creator repository.
5. Provenance is visible as repeated product narrative rather than concentrated
   control-plane metadata. There are 67 `Ubuntu Zombie`/`ubuntu-zombie`
   references across 26 tracked files, but only a subset is technically
   necessary.

The desired end state is:

- **one edit authority**: the creator monorepo;
- **one deterministic export**: generated and tested by source-owned tooling;
- **one package layout and digest**: identical wherever it is built;
- **one explicit release authority**: preferably the Beep repository;
- **one reserved downstream augmentation boundary**; and
- **one concise provenance page**, with the rest of the product documentation
  speaking only about Beep.

## Documents

1. [CURRENT-STATE.md](CURRENT-STATE.md) — verified strengths, coupling,
   correctness risks, and reference classification.
2. [TRANSPLANT-CONTRACT.md](TRANSPLANT-CONTRACT.md) — the recommended
   source/export/mirror architecture and content policy.
3. [IMPLEMENTATION-ROADMAP.md](IMPLEMENTATION-ROADMAP.md) — ordered work,
   ownership, decision gates, and acceptance criteria.

## Scope and safety

These are recommendations only. No runtime, workflow, schema, package, test,
or product documentation file has been changed.

Do not solve the branding problem with a global replacement. Schema IDs,
protocol enum values, co-installation tests, release identities, and provenance
records are security or compatibility controls. They must be migrated
deliberately or retained in one technical location.

This folder is downstream-owned analysis and must never be included in a Beep
source archive, SBOM, installed product tree, or release attestation.
