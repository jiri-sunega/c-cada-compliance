# C-CADA — CRA Compliance Record

This repository is the **public compliance record** for C-CADA, published
under the EU Cyber Resilience Act (CRA). It is a one-way mirror: content is
snapshot-copied here from the private source repository (`c-cada-platform`)
by a manual publish script and is not edited directly (except this README
and `PROVENANCE.md`, which are mirror-owned).

The source repository — application code, build pipelines, and internal
process docs — remains **private**. Nothing beyond the compliance
documentation below is mirrored here.

## Status: DRAFT

The EU Declarations of Conformity (`doc/EU-DoC-c-cada-authoring.md`,
`doc/EU-DoC-c-cada-runtime.md`) in this repository are **DRAFT** and are
**not yet signed declarations**. They will not be finalized until the
associated products reach market placement. Nothing in this repository —
including the Annex VII technical documentation — should be read as a
completed, in-effect conformity declaration until that status changes and
this notice is updated.

## Manufacturer contact

- Jiri Sunega
- jiri.sunega@koworx.net
- koworx.net

## Vulnerability disclosure

See `SECURITY.md` in this repository for the coordinated vulnerability
disclosure (CVD) policy and reporting channel.

## Layout

| Path | Contents |
|---|---|
| `annex-vii/` | Annex VII technical documentation (product description, user information, design & development, vulnerability handling, risk assessment, support period, standards & solutions, test reports, declaration of conformity) |
| `doc/` | EU Declarations of Conformity — draft, per product (authoring, runtime) |
| `SECURITY.md` | Coordinated vulnerability disclosure (CVD) policy |
| `PROVENANCE.md` | Source commit SHA and timestamp of the most recent publish |

## Publishing

Content here is republished from `c-cada-platform` via
`tools/scripts/publish-compliance.sh` (manual, by design — see that
script's header for the CI-automation seam). `PROVENANCE.md` always
reflects the exact source commit last published.
