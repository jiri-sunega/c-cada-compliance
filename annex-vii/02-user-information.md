# 2. Information and instructions to the user (Annex VII point 1(d) / Annex II)

## II.1 Manufacturer identity

The manufacturer is Jiří Sunega, operating under the name koworx.net, contactable at
jiri.sunega@koworx.net; the manufacturer's website is koworx.net. A postal
address (Annex II item 1) will be completed in the EU DoC's manufacturer
field at market placement — see
`docs/compliance/doc/EU-DoC-c-cada-authoring.md` §2 and
`docs/compliance/go-live-checklist.md`; until then the digital contacts
above are the operational channels.

## II.2 Vulnerability reporting single point of contact

The single point of contact for vulnerability reports is GitHub private
vulnerability reporting (preferred), with `security@koworx.net` as a
fallback address, under the coordinated vulnerability disclosure policy
published at `SECURITY.md` in the repository root (ADR-0127). Noted
honestly rather than glossed over: actively monitoring that inbox, and the
other live-monitoring go-live actions (CSIRT/ENISA registration, enabling
GitHub's private vulnerability reporting), remain an open owner action
carried forward from Sub-project B (ADR-0127).

## II.3 Product identification

Products are identified by product name together with the `v*` release-tag
and version convention: the product version is the tag with its leading
`v` stripped. Every signed release's `release-manifest.json` binds the tag
and version inside the signed manifest bytes, so a verified release
carries its own tamper-evident identification (ADR-0139).

## II.4 Intended purpose, security environment, essential functionality, and security properties

Each product's intended purpose is described in `01-product-description.md`.
The assumed security environment is an on-premises deployment behind plant
network segmentation. The table below summarizes the security properties
relevant to the user, each detailed further in ch. 03 (design and
development) and ch. 05 (risk assessment):

| Property | Summary |
|---|---|
| Access control | RBAC composed with layered authorization |
| Device authority | IP/FQDN gating plus an mTLS client-certificate facet |
| Frontend hardening | Strict Content-Security-Policy and Trusted Types |
| Session/token handling | mTLS-bound tokens |
| Supply chain | Signed packages and signed platform updates |
| Accountability | Audit logging |

## II.5 Known or foreseeable circumstances of significant risk (reasonably foreseeable misuse)

The following circumstances are known or reasonably foreseeable ways the
product could be used outside its intended security envelope: exposing
authoring or runtime endpoints to untrusted networks, since the products
assume segmented plant networks rather than direct exposure; running with
`--allow-unsigned` outside development; installing packages from
untrusted publishers by editing `trusted-publishers.json`; and skipping
release verification before applying an update.

## II.6 EU declaration of conformity address

The EU declaration of conformity (draft until market placement) is
published at `https://github.com/jiri-sunega/c-cada-compliance` under
`doc/` — see `09-declaration-of-conformity.md` for the draft DoC files and
the fields still to be completed at go-live.

## II.7 Support

Support consists of security updates delivered via signed `v*` releases,
announced through GHSA advisories. The committed support period is **5
years from the first release, `v0.1.0` (2026-07-04), running through
2031-07-04.** This end date is restated wherever the support period is
referenced elsewhere in this dossier.

## II.8 Detailed instructions

### (a) Secure commissioning

Before operational use: change the seeded default credentials; provision
per-tenant signing keys using the `seed-tenant` CLI verb; configure
`trusted-publishers.json` for the publishers the deployment trusts; enable
CSP enforce mode (the shipped default is `ReportOnly`, ADR-0120); and
complete mTLS device provisioning.

### (b) How changes affect data security

Updates replace product binaries and run the accompanying EF migration
bundles; existing configuration and data are preserved across an update.
Each release's SBOM diff against the prior release lets an operator review
what changed in the update's dependency composition.

### (c) Update installation

Download the release assets, then verify them before installing — either
with `cicada verify-release --manifest release-manifest.json --dir <dir>`
or via the openssl-only independent path documented in `SECURITY.md`. As
recorded in ADR-0139, the platform's signed-update mechanism has no
anti-rollback protection: an on-path attacker could serve an older,
validly-signed release, so operators must always confirm that the printed
version/tag in the verification output is the release they intended to
install. Only after verification succeeds: stop the running services,
unzip the release archive, run the EF migration bundles, and restart.

### (d) Secure decommissioning

Decommissioning a deployment involves: deleting the PostgreSQL database
(credentials, roles, and audit log); removing the tenant repository
directories; deleting `~/.cicada-keys`; and allowing any browser-held
sessions to expire — sessions using the httpOnly-cookie token-hardening
mode expire via the cookie's own lifetime (ADR-0123).

### (e) Automatic-update opt-out

Not applicable: no automatic-update default exists to opt out of. This is
a deliberate design choice for availability-critical industrial systems —
updates remain a deliberate operator action rather than an automatic one
(ADR-0139; see also `05-risk-assessment.md §AI-I-2c`).

### (f) Integrator information

A SCADA application composed on C-CADA is the customer's own product for
CRA purposes. Its bundle SBOM (`sbom.cdx.json`) joins with the runtime
product's own SBOM to form the complete deployed picture (ADR-0128).
Package trust is verified at install time; the mechanism is documented in
the package distribution & trust ADRs (ADR-0112 through ADR-0119; see
`c-cada-knowledge/knowledge/security/package-distribution-and-trust.md`).

## II.9 Software bill of materials access

SBOMs/VEX are delivered to customers with each release, not published
openly: per-product CycloneDX SBOMs and the platform VEX are attached as
release assets to every `v*` GitHub Release of `jiri-sunega/c-cada-platform`
(ADR-0128, ADR-0129), and each signed update archive additionally embeds
its own product SBOM (ADR-0139). They are also provided to market
surveillance authorities on reasoned request, per Annex VII point 8
(`00-overview.md`). The platform repository that hosts these releases
remains private; the public compliance mirror publishes the dossier, the
DoCs, and the CVD policy (`00-overview.md`; II.6 above), not the SBOMs/VEX.
The app-tier SBOM for a compiled application is emitted into the
application bundle as `sbom.cdx.json` (ADR-0128).
