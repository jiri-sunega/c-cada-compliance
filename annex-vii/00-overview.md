# CRA Annex VII Technical Documentation — C-CADA Platform

## Purpose and legal basis

This dossier is the technical documentation required by Regulation (EU)
2024/2847 (the Cyber Resilience Act) Art. 31 and Annex VII. Per Art. 31(2),
"the technical documentation shall be drawn up before the product with
digital elements is placed on the market and shall be continuously updated,
where appropriate, at least during the support period." This dossier is
maintained on that continuous-update basis for the duration of C-CADA's
committed support period (see `06-support-period.md`).

## Products covered

The dossier treats three C-CADA platform products as separate products for
CRA purposes, per the taxonomy decided in ADR-0128:

| Product | Role |
|---|---|
| `c-cada-authoring` | Engineering-workstation software for designing and compiling SCADA applications |
| `c-cada-runtime` | On-premises runtime that executes compiled bundles |
| `c-cada-marketplace` (registry) | Package registry/marketplace — see thin-section rationale below |

**Application tier.** Every SCADA application composed and compiled on
C-CADA is, in principle, its own product, and CRA obligations for such an
application sit with whoever places it on the market — not with C-CADA
itself. C-CADA's role is to make that application's SBOM derivable: the
compiler emits a bundle SBOM (`sbom.cdx.json`) into every compiled
application bundle, which joins with the runtime product's own SBOM to form
the complete deployed picture (ADR-0128).

**Registry thin-section rationale.** `c-cada-marketplace` is operated by
koworx.net as a hosted service and not placed on the market; per ADR-0139
(C-6) it ships no update archive; its SBOM/VEX are still published per
ADR-0128/0129.

## Dossier map

| Chapter | File | Annex VII point |
|---|---|---|
| 1 | `01-product-description.md` | 1(a)–(c) |
| 2 | `02-user-information.md` | 1(d) / Annex II |
| 3 | `03-design-and-development.md` | 2(a) + 2(c) |
| 4 | `04-vulnerability-handling.md` | 2(b) |
| 5 | `05-risk-assessment.md` | 3 |
| 6 | `06-support-period.md` | 4 |
| 7 | `07-standards-and-solutions.md` | 5 |
| 8 | `08-test-reports.md` | 6 |
| 9 | `09-declaration-of-conformity.md` — classification analysis, module-A route, Art. 32(5) fallback, CE mechanics, draft DoCs | 7 |

Annex VII point 8 — the software bill of materials, provided on a market
surveillance authority's reasoned request — is addressed by the SBOM/VEX
assets published on every `v*` GitHub Release (ADR-0129) and by the SBOM
carried inside each signed platform update archive (ADR-0139).

## Maintenance rule

Dossier updates ride the same commit as the security-relevant behavior
change they document. Git history together with the project's release
tags constitutes the Art. 31(2) continuous-update record: each `v*` tag
pins the dossier state that shipped with that version. This dossier is
also published on the public compliance mirror
(`https://github.com/jiri-sunega/c-cada-compliance`), refreshed via
`tools/scripts/publish-compliance.sh`; the platform repository itself
remains private.
