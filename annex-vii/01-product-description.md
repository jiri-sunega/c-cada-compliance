# 1. Product description (Annex VII point 1(a)–(c))

## c-cada-authoring

**Intended purpose:** engineering workstation software for designing SCADA
applications — device types, instances, forms, layouts, graphics, roles,
and device-authority policy — and compiling them into signed runtime
bundles.

**Versions affecting compliance:** released as `v*` tags; the product
version is the tag with its leading `v` stripped (`release.yml`). Every
release carries its SBOM.

**Composition:** `Cicada.Authoring.Api` (.NET 10) together with a
host-served Angular SPA, the `cicada` compiler CLI, and EF migration
bundles (per the archive contents fixed in ADR-0139).

## c-cada-runtime

**Intended purpose:** on-premises runtime that executes compiled bundles —
process connectivity via OPC UA southbound, the unified state bus, and the
operator UI.

**Composition:** `Cicada.Runtime.Host` together with a host-served runtime
SPA.

## c-cada-marketplace (registry)

Thin section: `c-cada-marketplace` is operated by koworx.net as a hosted
service and not placed on the market; per ADR-0139 (C-6) it ships no
update archive; its SBOM/VEX are still published per ADR-0128/0129.

**Composition:** `Cicada.Registry.Api`, using dedicated publish tokens with
TOFU (trust-on-first-use) key binding (ADR-0116).

## Form factor

All three products are software-only. Annex VII point 1(c) — photographs
or illustrations of external features, marking, and internal layout for
hardware products — is not applicable.

## Deployment topology

C-CADA follows an on-premises operator deployment model: authoring runs on
engineering workstations, and runtime runs on plant servers. Both
products' SPAs are host-served directly from their API binaries (Control
3, ADR-0120). Runtime clients are additionally gated by mTLS-based device
authority (ADR-0107/0108).
