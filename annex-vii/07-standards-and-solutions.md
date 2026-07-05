# 7. Standards and technical specifications applied (Annex VII point 5)

Annex VII point 5 asks first for a list of harmonised standards, common
specifications, or European cybersecurity certification schemes applied,
and — only where those have not been applied — for descriptions of the
solutions adopted to meet the essential cybersecurity requirements of Annex
I Parts I and II instead, including a list of other relevant technical
specifications applied
(`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
lines 3013–3016). This chapter states, as of 2026-07-04, why the first route
does not apply to C-CADA and points to where the second route's content
already lives in this dossier.

## Harmonised standards, common specifications, and certification schemes

None of the three apply. No harmonised standard for the Cyber Resilience Act
has yet had its reference published in the Official Journal of the European
Union; no common specification has been adopted under CRA Art. 27 for the
essential cybersecurity requirements; and no European cybersecurity
certification scheme has been adopted under Regulation (EU) 2019/881 and
identified under CRA Art. 27(8) that C-CADA could claim against. This is a
statement about the regulatory landscape as of 2026-07-04, not a gap in
C-CADA's own controls — with nothing yet published to apply, the "applied in
full or in part" branch of Annex VII point 5 is not available to any
manufacturer, and the regulation's own text anticipates exactly this case by
providing the alternative route below.

## The alternative route: descriptions of solutions adopted

Annex VII point 5's alternative — "descriptions of the solutions adopted to
meet the essential cybersecurity requirements set out in Parts I and II of
Annex I" — is not re-derived here. It is already written out, requirement by
requirement, as:

- **Part I** (the fourteen essential requirements, `AI-I-1` through
  `AI-I-2m`): the full applicability/threats/controls/residual-risk register
  in `05-risk-assessment.md`.
- **Part II** (the eight vulnerability-handling requirements, `AI-II-1`
  through `AI-II-8`): the requirement-by-requirement mapping in
  `04-vulnerability-handling.md`.

This chapter's job is narrower: name the orientation frameworks and the
concrete technical specifications those two chapters' controls are actually
built on, without restating the controls themselves.

## IEC 62443 — orientation only, no conformance claim

Where a control in `03-design-and-development.md` or `05-risk-assessment.md`
is shaped by IEC 62443 concepts — zones and conduits, security levels — that
reference situates the design decision in familiar SCADA/industrial-control
vocabulary only. **This is not a conformance claim.** C-CADA has not been
assessed against the IEC 62443 family by any accredited body, and citing it
here, as in ch. 05's method statement, is orientation rather than
certification. Should Sub-project E's conformity-assessment route ever call
for formal IEC 62443-3-2/3-3 rigor, that would be new work, not something
this chapter already asserts.

## Technical specifications actually applied

| Specification | Where it is applied | Governing citation |
|---|---|---|
| OAuth 2.0 / OpenID Connect (via OpenIddict) | Authentication for both the authoring and runtime APIs, composed with RBAC and layered authorization | `03-design-and-development.md §Authorization layers`; ADR-0067–ADR-0071 |
| mTLS certificate-bound access tokens (RFC 8705) | The optional `CookieMtls` token-hardening mode | `03-design-and-development.md §Browser security controls`; ADR-0122–ADR-0124 |
| Ed25519 digital signatures (RFC 8032) | Package signing, signed release manifests, and git commit signing across the authoring-to-runtime chain | `03-design-and-development.md §Package distribution and trust chain` and `§Signed platform updates`; ADR-0112–ADR-0119, ADR-0139, ADR-0035 |
| CycloneDX 1.6 | Per-product SBOMs, the platform VEX document, and the application-tier bundle SBOM | `02-user-information.md §II.9`; ADR-0128, ADR-0129 |
| OPC UA | Runtime southbound connectivity to field devices | `01-product-description.md`; device-authority gating of those connections at ADR-0107, ADR-0108 |
| CSP Level 3 nonce/source-list mechanics + the W3C Trusted Types specification | Browser hardening (`script-src 'self' 'nonce-…'`, no `unsafe-inline`, `require-trusted-types-for 'script'`) on both host-served single-page applications | `03-design-and-development.md §Browser security controls`; ADR-0120, ADR-0121 |
| Semantic Versioning 2.0.0 | The `v*` release-tag/version convention bound into every signed release manifest | `02-user-information.md §II.3`; ADR-0139 |

As with IEC 62443 above, naming a specification in this table describes what
technical basis a control rests on — it is not a certification or
conformance claim against that specification. The conformance question
itself, for any of the essential cybersecurity requirements, is answered in
`04-vulnerability-handling.md` and `05-risk-assessment.md`, never here.
