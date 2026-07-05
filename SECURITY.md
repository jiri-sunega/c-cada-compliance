# Security Policy

C-CADA takes the security of its platform seriously. This document is our
**Coordinated Vulnerability Disclosure (CVD)** policy: how to report a
vulnerability, what to expect, and the support commitments behind our security
updates.

## Supported versions

Security updates are provided for the **current major version** and the
**immediately preceding major version**. Older majors are out of support.

| Version | Supported |
|---------|-----------|
| current major | ✅ |
| previous major | ✅ |
| older | ❌ |

## Support period

We provide **free security updates for 5 years** from the release of a given
version. This is the window during which reported vulnerabilities affecting
that version will be triaged and remediated under this policy.

## Reporting a vulnerability

**Please report privately — do not open a public issue for a suspected
vulnerability.**

- **Preferred:** [GitHub private vulnerability reporting](https://github.com/jiri-sunega/c-cada-platform/security/advisories/new)
  (the "Report a vulnerability" button on the repository's *Security* tab).
  This gives us a private channel, supports a coordinated fix, and lets us
  issue a CVE.
- **Fallback:** email **security@koworx.net**.

Please include: the affected component and version, a description and impact
assessment, and clear reproduction steps (proof-of-concept welcome).

## What to expect (response targets)

| Stage | Target |
|-------|--------|
| Acknowledge your report | within **3 business days** |
| Triage + severity assessment (CVSS) | within **10 business days** |
| Remediation — **Critical** | within **30 days** |
| Remediation — **High** | within **90 days** |
| Remediation — **Medium** | within **180 days** |
| Remediation — **Low** | best-effort, next scheduled release |

These are targets, not guarantees; complex issues may take longer, and we will
keep you informed.

## Coordinated disclosure

We follow coordinated disclosure. By default we ask for a **90-day** embargo
between report and public disclosure, so a fix and security update can be
released first. We publish advisories via GitHub Security Advisories (with a
CVE assigned where applicable) and **credit reporters** unless you ask us not
to.

## Actively exploited vulnerabilities

For vulnerabilities that are being actively exploited, or severe incidents
affecting the platform, we will notify the relevant CSIRT / ENISA in line with
the EU Cyber Resilience Act timelines (early warning within 24 hours,
notification within 72 hours, and a final report within 14 days), in addition
to remediating under this policy.

## Safe harbor

We will not pursue legal action against researchers who act in good faith under
this policy: who avoid privacy violations, data destruction, and service
degradation; who only interact with systems/accounts they own or have explicit
permission to test; and who give us reasonable time to remediate before public
disclosure.

## Scope

- **In scope:** the C-CADA platform (this repository) and the koworx base
  packages.
- **Out of scope:** third-party deployments and operator misconfigurations,
  social-engineering attacks, volumetric denial-of-service testing, and
  already-disclosed third-party CVEs without a demonstrated C-CADA-specific
  impact.

C-CADA's security architecture (device authority, package signing & trust,
strict CSP + Trusted Types, token hardening) is documented separately in the
platform's security-hardening design records.

## Verifying platform releases

Every `v*` GitHub Release ships `release-manifest.json` (enumerating EVERY asset
with its SHA-256 and size) plus `release-manifest.sig` — an Ed25519 signature over
the exact manifest bytes. The release public key is:

- Base64 (raw 32-byte Ed25519): see `tools/security/release-signing.pub`
- Fingerprint (SHA-256): `62a5471d39090f53a2b44ca13515fab49031795627b4710708e3e9245c1b72a4`
- PEM (SPKI): `tools/security/release-signing.pub.pem`

Obtain the key ONCE, out-of-band (this repository / the printed technical
documentation) — not from the release you are verifying.

### Primary path (cicada CLI)

    cicada verify-release --manifest release-manifest.json --dir <download-dir>

Exits non-zero on any signature/hash/size mismatch or missing asset, and refuses
dry-run (`keyRole: dev`) manifests. Note the bootstrap caveat: the authoring
archive contains `cicada` itself — for first-time verification use the
independent path below, or a `cicada` from an already-verified installation.

### Independent path (openssl + jq only — no C-CADA code involved)

Requires OpenSSL 3.x (not LibreSSL — macOS users: `brew install openssl` and use
its binary, e.g. `$(brew --prefix openssl)/bin/openssl`); macOS's bundled
system `openssl` is LibreSSL and cannot parse Ed25519 SPKI keys or verify
Ed25519 signatures.

    # 1. Signature over the exact manifest bytes
    base64 -d < release-manifest.sig > /tmp/release-manifest.sig.bin
    openssl pkeyutl -verify -pubin -inkey release-signing.pub.pem -rawin \
      -in release-manifest.json -sigfile /tmp/release-manifest.sig.bin
    # must print: Signature Verified Successfully

    # 2. Reject dry-run manifests
    jq -e '.keyRole == "release"' release-manifest.json

    # 3. Every asset hash
    jq -r '.assets[] | "\(.sha256)  \(.name)"' release-manifest.json | shasum -a 256 -c -
    # every line must end in: OK
    # (shasum ships with Perl; on minimal Linux images use sha256sum -c - instead)

Dev/dry-run manifests are signed by the committed development key
(`tools/security/release-signing-dev.pub`, `keyRole: dev`) and are NOT releases.
