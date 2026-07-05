# 5. Cybersecurity risk assessment (Annex VII point 3)

This chapter is the dossier's cybersecurity risk assessment referred to by
Annex I Part I point (2) and required as a standalone artifact by Annex VII
point 3. It is organized in three parts: a method statement; a per-product
threat model expressed in STRIDE vocabulary; and a requirement register that
maps every Annex I Part I requirement (`AI-I-1`, `AI-I-2a` through `AI-I-2m`)
onto the threats it addresses, the controls that implement it, and any
residual risk that remains. A residual-risk index closes the chapter,
collecting every residual named across the register into one table together
with the queue item, ADR, or carry-forward that owns it.

## Method

The assessment is requirement-organized rather than asset-organized: each of
the fourteen Annex I Part I paragraphs gets its own section stating whether it
applies to C-CADA's three products, which threats it is meant to counter,
which shipped controls implement it, and what — if anything — remains open.
Threats are named using STRIDE vocabulary (Spoofing, Tampering, Repudiation,
Information disclosure, Denial of service, Elevation of privilege) because it
maps directly onto Annex I's own requirement groupings (unauthorised access →
spoofing/elevation of privilege; data manipulation → tampering; availability
→ denial of service) without introducing a second taxonomy.

Where a requirement or a control below is oriented by IEC 62443 concepts
(zones and conduits, security levels), that reference is used only to situate
the design decision — **this assessment makes no IEC 62443 conformance
claim**, certified or otherwise; the C-CADA platform has not been assessed
against that standard by any accredited body, and citing it here is
orientation, not certification. Likewise, every verdict below is phrased as
"Applicable — addressed by …" or "Not applicable — justification …", never
as "complies" or "conforms": conformity is Sub-project E's declaration
(`09-declaration-of-conformity.md`), not a claim this chapter makes for
itself.

Verdicts and their supporting citations were fixed by the CRA Sub-project D
task plan before this chapter was written; this chapter writes them out and
verifies each cited ADR resolves to a real, accepted decision record in
`c-cada-knowledge/catalog.md`. No new factual claim about the platform is
introduced here beyond what earlier chapters (`01`–`04`) and their cited
ADRs already establish.

## Threat models

Three tables below cover the three products this dossier treats separately
(`01-product-description.md`): `c-cada-authoring`, `c-cada-runtime`, and the
thin `c-cada-marketplace` registry section. Trust-boundary and attack-surface
language follows `03-design-and-development.md`'s architecture description.

### `c-cada-authoring` threat model

| Asset | Trust boundary | Attack surface | Primary threats (STRIDE) | Governing controls |
|---|---|---|---|---|
| Authoring API + SignalR hubs | Authenticated engineer session ↔ API process | REST endpoints, `StateHub`/authoring hubs | Spoofing, Tampering, Elevation of privilege | OIDC (OpenIddict) + RBAC + layered authorization glob grants (ADR-0067–ADR-0071) |
| Package install path | Local filesystem ↔ untrusted `.cpkg` archive | `cicada add`, install-time extraction | Tampering | Ed25519 verify-before-extract; CIC810 unsigned-package refusal by default (ADR-0112–ADR-0119) |
| Git materialization of authored state | Authoring process ↔ tenant git repository | Commit writes to the materialized project history | Tampering, Repudiation | Native git commit signing with the tenant's per-active keypair; untrusted (signature-failing) commits block saves pending author review (ADR-0035) |
| Browser (Angular SPA + package-contributed UI) | Browser ↔ page content, including third-party package UI | Rendered DOM, script execution | Information disclosure, Tampering (XSS) | Strict nonce-based CSP, no `unsafe-inline`, Trusted Types (ADR-0120, ADR-0121) |
| Compile chain (`cicada compile`) | Authored working state ↔ emitted bundle | Compiler input resolution and bundle emission | Tampering | Deterministic, reproducible compiler output distributed exclusively via signed release archives (ADR-0139) |

### `c-cada-runtime` threat model

| Asset | Trust boundary | Attack surface | Primary threats (STRIDE) | Governing controls |
|---|---|---|---|---|
| Runtime API + hubs | Authenticated operator session ↔ API process | REST endpoints, `StateHub` streaming | Elevation of privilege | Operator RBAC + per-user read-egress gating (ADR-0067–ADR-0071) |
| OPC UA southbound connectivity | Runtime host ↔ field device network | Outbound OPC UA client connections | Spoofing, Denial of service | Device-authority IP/FQDN allow-listing plus an mTLS client-certificate facet (ADR-0107, ADR-0108); a per-connector monitored-item ceiling and notification-rate shed cap DoS pressure from this connection — see `§AI-I-2h` |
| Update channel | Operator ↔ downloaded release archive | `cicada verify-release`, manual install steps | Tampering, Rollback | Ed25519-signed release manifest, explicit asset enumeration (ADR-0139); no anti-rollback protection is an accepted consequence of that same decision — see `§AI-I-2c` |
| Unified state bus egress | Authorized user session ↔ connected browser | `StateClientService.resolve()` streamed values | Information disclosure | Compile-time usage-driven exposure gating (`exposeToFrontend`, ADR-0086) composed with per-user read-egress authorization (ADR-0084, ADR-0085) |

### `c-cada-marketplace` (registry) threat model — thin section

| Asset | Trust boundary | Attack surface | Primary threats (STRIDE) | Governing controls |
|---|---|---|---|---|
| Publish path | Publisher ↔ hosted registry service | `publish` endpoint | Spoofing | Dedicated publish tokens with trust-on-first-use key binding (ADR-0116) |
| Hosted-service boilerplate | Operator (koworx.net) ↔ public network | Standard web-service surface | Spoofing, Tampering, Denial of service | Ordinary hosted-service hardening; the registry is not placed on the market and ships no update archive (ADR-0139, decision C-6) |

## Requirement register (Annex I Part I)

Each section below opens with a one-line paraphrase of the corresponding
paragraph of Annex I Part I of Regulation (EU) 2024/2847, followed by
Applicability, Threats addressed, Implemented controls, and Residual risk.
Paraphrases are drawn from the regulation text archived at
`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
(lines 2607–2665).

## AI-I-1 — Appropriate level of cybersecurity based on risk

Requirement (paraphrase): products with digital elements must be designed,
developed, and produced to ensure an appropriate level of cybersecurity
based on the risks.

**Applicability:** Applicable — this is the umbrella requirement Annex I
Part I opens with; every other paragraph in this register is a specific
instance of it.

**Threats addressed:** All STRIDE categories, at the level of the overall
design posture rather than any single surface.

**Implemented controls:** The existence of this risk assessment itself is
part of addressing the requirement, together with the security-hardening
roadmap's four completed controls (device authority, package distribution
and trust, strict CSP/Trusted Types, and hardened token storage — ADR-0104
through ADR-0124) and the layered-authorization model underlying access
control platform-wide (ADR-0067–ADR-0071).

**Residual risk:** None named at this umbrella level; specific residuals are
tracked against the paragraph they arise under, below.

## AI-I-2a — No known exploitable vulnerabilities at market availability

Requirement (paraphrase): products must be made available on the market
without known exploitable vulnerabilities.

**Applicability:** Applicable.

**Threats addressed:** Tampering and elevation of privilege via known,
unpatched vulnerabilities in shipped dependencies.

**Implemented controls:** A fail-closed dependency-vulnerability scan gate
(`tools/scripts/scan-vulnerable-dependencies.py`) runs on every push, pull
request, and a weekly schedule; findings that are not suppressed surface
honestly as `in_triage` VEX statements — the `v0.1.0` release shipped 53 such
statements rather than hiding them (ADR-0126, ADR-0129).

**Residual risk:** The scan gate currently runs report-only, not enforcing —
`security-scan.yml` warns on findings rather than blocking merges, pending a
single owner triage pass across the ~30 pre-existing findings, after which the
gate flips to enforcing. Owning item: CRA Sub-project B triage carry-forward
(dashboard §4).

## AI-I-2b — Secure by default configuration

Requirement (paraphrase): products must be made available with a secure by
default configuration, including the possibility to reset to the original
state, unless otherwise agreed for a tailor-made product.

**Applicability:** Applicable.

**Threats addressed:** Spoofing and elevation of privilege arising from
permissive defaults (unsigned code execution, unrestricted script execution,
unchanged seeded credentials).

**Implemented controls:** Unsigned packages are refused by default (CIC810,
ADR-0112–ADR-0119); a strict, nonce-based Content-Security-Policy and Trusted
Types ship on both products (ADR-0120, ADR-0121); seeded default credentials
must be changed at commissioning per `02-user-information.md §II.8(a)`.

**Residual risk:** The shipped CSP default is `ReportOnly`, not enforcing —
flipping to enforce mode is a deliberate operator step documented as a
commissioning instruction (`02-user-information.md §II.8(a)`), not a shipped
default. This is recorded as an accepted, documented posture rather than a
tracked open item.

## AI-I-2c — Security updates, including automatic installation (worked example)

Requirement (paraphrase): vulnerabilities must be addressable through
security updates, including — where applicable — automatic security updates
installed within an appropriate timeframe as a default, with a clear opt-out,
notification of available updates, and the option to postpone them.

**Applicability:** **Partially applicable.** This paragraph illustrates how
this register treats a multi-clause requirement where sub-clauses diverge:
each clause below gets its own verdict rather than one blended answer.

- **Security updates exist:** Yes — Applicable. Platform updates ship as
  Ed25519-signed release archives with an explicit, enumerated asset
  manifest, independently verifiable before install (ADR-0139).
- **Automatic installation:** Not applicable. C-CADA targets
  availability-critical industrial deployments where an unattended update
  could interrupt plant operations; installing an update is a deliberate
  operator action by design, not an automatic one (ADR-0139). A documented
  opt-out mechanism is therefore not applicable — there is no automatic
  default to opt out of (`02-user-information.md §II.8(e)`).
- **Update notification:** Applicable, via GitHub Releases and GHSA
  advisories rather than an in-product mechanism.

**Threats addressed:** Tampering (unpatched or maliciously substituted
updates) and, for the automatic-installation clause specifically,
denial of service via an update interrupting a running industrial process.

**Implemented controls:** `cicada verify-release` (or the openssl-only
independent path in `SECURITY.md`) verifies the signed manifest before
install; GHSA advisories and GitHub Release notes serve as the notification
channel (ADR-0139; `04-vulnerability-handling.md §AI-II-7`).

**Residual risk:** No in-product update-notification mechanism exists —
operators must watch GitHub Releases/GHSA externally rather than being
notified inside the running product. Owning item: none currently tracked in
the dashboard §4 queue or `BRAINSTORM-BACKLOG.md`; recorded here as a new,
currently-untracked residual (candidate for a future queue item, not yet
minted).

## AI-I-2d — Protection from unauthorised access

Requirement (paraphrase): products must protect against unauthorised access
through appropriate control mechanisms — including authentication and
identity/access management — and report on possible unauthorised access.

**Applicability:** Applicable.

**Threats addressed:** Spoofing and elevation of privilege.

**Implemented controls:** OIDC authentication (OpenIddict) composed with RBAC
and layered, path-scoped authorization (ADR-0067–ADR-0071); device authority
as a further, orthogonal gate on runtime device connections (ADR-0107,
ADR-0108). State-changing request attempts — including denied ones — are
recorded in the `security.audit_log` table: `AuditLoggingMiddleware` records
every POST/PUT/PATCH/DELETE after the pipeline completes, capturing the
actual HTTP response status code, so a 401/403 denial is preserved as an
audit entry, not only successful writes. Precisely stated: the middleware
enqueues entries asynchronously (best-effort): under channel saturation
entries are dropped with a logged warning. **`/connect/` division of labor:**
the middleware still skips requests under `/connect/` outright, for
request-body capture safety (the OAuth form body carries credentials it must
never see) — but the token endpoint is not left unaudited. `AuthorizationController`
calls `AuthenticationAuditor` (`Cicada.Security.Application.Services`) directly
around each grant-handling branch, enqueueing to the same `security.audit_log`
table: failures carry a `reason` — `unknown_user` / `invalid_password` /
`locked_out` / `invalid_refresh_token` — and successes carry the resolved
user identity. The auditor's signatures accept only identifiers, never
password or token material, so the credential-safety property the
middleware exclusion exists for is preserved by construction.

**Residual risk:** Stated precisely rather than rounded up to full coverage:

1. **Read-request denials are not audit-logged.** `AuditLoggingMiddleware`
   audits only state-changing methods (POST/PUT/PATCH/DELETE); a denied GET
   or a denied streaming read (read-egress, ADR-0071) produces no
   `security.audit_log` entry. This is the same underlying logging-coverage
   gap named under `§AI-I-2l`. Owning item: Testing Strategy (dashboard §4
   queue).
2. No push notification exists for authorization-state changes (no
   `HubAuthzNotifier`/`authzChanged` mechanism) — a connected client's
   effective permissions can go stale until its next request or reconnect.
   Owning item: Layout Designer epic carry-forward (dashboard §4).

## AI-I-2e — Confidentiality of data at rest and in transit

Requirement (paraphrase): products must protect the confidentiality of
stored, transmitted, or otherwise processed data — personal or other — such
as through state-of-the-art encryption at rest or in transit and other
technical means.

**Applicability:** Applicable.

**Threats addressed:** Information disclosure.

**Implemented controls:** TLS protects data in transit. Token/session
handling supports an `HttpOnly`, `SameSite=Strict` cookie mode with the
access token kept in memory only, additionally bindable to the client's mTLS
certificate (ADR-0122–ADR-0124). At rest, confidentiality relies on
PostgreSQL and OS-level protections rather than application-layer
encryption.

**Residual risk:** No application-layer encryption at rest is implemented.
This is justified rather than tracked as a gap: C-CADA's deployment model is
on-premises, single-operator plant infrastructure, and database-level
encryption at rest is deployment guidance for the operator's own
infrastructure (`02-user-information.md §II.8(a)`) rather than a platform
control the product itself must provide.

## AI-I-2f — Integrity of data, commands, programs, and configuration

Requirement (paraphrase): products must protect the integrity of stored,
transmitted, or otherwise processed data, commands, programs, and
configuration against unauthorised manipulation, and report on corruptions.

**Applicability:** Applicable.

**Threats addressed:** Tampering.

**Implemented controls:** Signed packages (ADR-0112–ADR-0119), signed release
archives (ADR-0139), EF-migration-controlled schema evolution, and a
git-materialized authoring history with commit signing (ADR-0035) together
protect integrity across the authoring-to-runtime chain. Corruption is
reported through `cicada verify-release` failures, package-verification
failures, and their associated CIC diagnostic codes (CIC810–CIC816).

**Residual risk:** None named beyond what is already tracked elsewhere in
this register (package-distribution known gaps, `§AI-I-2k`).

## AI-I-2g — Data minimisation

Requirement (paraphrase): products must process only data — personal or
other — that is adequate, relevant, and limited to what is necessary for the
product's intended purpose.

**Applicability:** Applicable.

**Threats addressed:** Information disclosure, in the broad sense of
over-collection increasing exposure surface.

**Implemented controls:** The platform processes industrial process data and
operator identities only, with no telemetry and no external data flows —
consistent with an on-premises, AGPL-licensed deployment model that has no
vendor-hosted collection point to send data to.

**Residual risk:** None named.

## AI-I-2h — Availability of essential and basic functions

Requirement (paraphrase): products must protect the availability of
essential and basic functions, including after an incident, through
resilience and mitigation measures against denial-of-service attacks.

**Applicability:** Applicable.

**Threats addressed:** Denial of service.

**Implemented controls:** On-premises network isolation (assumed segmented
plant networks, `02-user-information.md §II.5`) remains the primary
availability control; device authority limits which clients may reach
runtime device connections at all (ADR-0107, ADR-0108). On top of that,
DoS-hardening (2026-07-05) bounds resource exhaustion at each of the
runtime's externally-reachable surfaces:

- **API surface:** a per-IP rate limit on the OAuth token endpoint
  (`POST /connect/token`, ADR-0135) plus a global, per-user rate limit
  applied to every `/api/v1` endpoint
  (`RuntimeDosOptions.ApiRequestsPerMinutePerUser`, default 300/minute,
  fixed-window, partitioned by authenticated identity with an IP fallback
  for anonymous callers).
- **SignalR hub surface:** a maximum inbound message size
  (`RuntimeDosOptions.HubMaxReceiveMessageBytes`) and a per-connection
  live-subscription ceiling (`SubscriptionCapGuard`) on `StateHub`; the
  ceiling clamps rather than throws, so a connection at its limit keeps the
  subscriptions it already holds.
- **OPC UA southbound connectivity:** a per-connector monitored-item
  ceiling (`OpcUaConnectorOptions.MaxMonitoredItemsPerConnector`, default
  10,000; enforced via the pure `MonitoredItemAdmission.Admit` helper)
  refuses to create monitored items beyond the ceiling — a malformed bundle
  or a device advertising an unexpectedly large address space cannot make
  one connector create unbounded subscriptions — and a per-connector
  notification-rate shed (`NotificationRateLimiter`, `MaxNotificationsPerSecond`,
  default 50,000/second) drops excess inbound data-change notifications
  rather than letting a flooding or faulty device starve the runtime's
  processing.

All three are availability-first by construction (design D-3): the
affected connector, hub connection, or API caller stays exactly as
connected as it was before — a capped OPC UA connector never transitions
to `Faulted` and is never disconnected, it just keeps the items/rate it
could stay within bounds on. Capping is loud rather than silent: throttled
`Console.Error` warnings (logged only at power-of-two drop counts, so the
warning itself cannot become a flood), cumulative diagnostic counters
(`SkippedMonitoredItemCount`, `DroppedNotificationCount`), and a
`Degraded` — not `Unhealthy` — result from `ConnectorsHealthCheck` once
either counter goes nonzero for an otherwise-`Connected` connector.

None of this replaces the OPC UA client's baseline resilience, which
pre-existed this hardening pass and is unchanged by it: exponential-backoff
reconnection (`OpcUaConnectorOptions.ReconnectBaseDelay`/`ReconnectMaxDelay`),
a per-monitored-item server-side queue (`QueueSize = 1`,
`DiscardOldest = true`), and a per-subscription publish cap
(`MaxNotificationsPerPublish = 1000`) were already in place; the new
ceiling and shed are additive bounds on top of that existing machinery, not
a replacement for it.

**Residual risk:** No high-availability / multi-node failover exists — a
single runtime host is still a single point of failure no matter how well
it protects itself against resource exhaustion. Owning item: Multi-node
cooperative deployment, Sessions D–F (dashboard §4 queue).

## AI-I-2i — Minimise negative impact on other devices or networks

Requirement (paraphrase): products must minimise the negative impact that
the product itself, or connected devices, has on the availability of
services provided by other devices or networks.

**Applicability:** Applicable.

**Threats addressed:** Denial of service (as a source, not just a target).

**Implemented controls:** OPC UA southbound connections are client-initiated
by the runtime toward field devices under device-authority gating, with no
amplification-style surfaces; the runtime's own polling load against each
field device is itself bounded and operator-configurable — sampling and
publishing intervals (`OpcUaConnectorOptions.DefaultSamplingInterval`,
default 500ms; `DefaultPublishingInterval`, default 1000ms; both
overridable per tag mapping) cap how aggressively the runtime reads from a
given device, so the runtime cannot itself become a DoS source against the
devices it connects to. Both products' SPAs are also served same-origin
from their own API binary rather than pulling third-party CDN content
(Control 3, ADR-0120), so there is no outbound load imposed on third-party
infrastructure as a side effect of normal operation.

**Residual risk:** None named beyond `§AI-I-2h`'s remaining high-availability
/ multi-node-failover residual, which applies here too: a runtime host that
goes down under load stops protecting other devices/networks from whatever
it was mediating, just as it stops serving its own functions.

## AI-I-2j — Limit attack surfaces, including external interfaces

Requirement (paraphrase): products must be designed, developed, and produced
to limit attack surfaces, including external interfaces.

**Applicability:** Applicable.

**Threats addressed:** All STRIDE categories, reduced by shrinking the
number of reachable entry points.

**Implemented controls:** Host-served SPAs eliminate a separate front-end
web-server surface for both products (Control 3, ADR-0120); each product
exposes a single API surface rather than multiple parallel ones; third-party
package contributions are sandboxed on the frontend via an iframe sandbox
(package-distribution Control 2 Slice 4, ADR-0119); the registry's own
external surface is
limited to a minimal publish-token endpoint (ADR-0116).

**Residual risk:** None named beyond the backend contribution-isolation
residual tracked under `§AI-I-2k`.

## AI-I-2k — Exploitation mitigation mechanisms

Requirement (paraphrase): products must be designed, developed, and produced
to reduce the impact of an incident using appropriate exploitation
mitigation mechanisms and techniques.

**Applicability:** Applicable.

**Threats addressed:** Elevation of privilege and tampering, specifically
the blast radius if an individual component is compromised.

**Implemented controls:** Strict CSP and Trusted Types constrain exploitable
script execution in the browser (ADR-0120, ADR-0121); fail-closed verifiers
throughout the package-trust and release-verification chain refuse to
proceed on ambiguous or corrupt input rather than degrading silently
(ADR-0112–ADR-0119, ADR-0139); the .NET managed runtime provides memory
safety for both API processes.

**Residual risk:** No backend `AssemblyLoadContext`-based process isolation
exists for third-party `Trusted`-tier backend contributions — today, backend
service contributions (e.g. `AlarmConnectivityService`) are first-party,
in-process only. This is a deliberately deferred, cleanly-seamed gap, per
`c-cada-knowledge/knowledge/security/package-distribution-and-trust.md`
§"Known gaps and deferred hooks".

## AI-I-2l — Security-related recording and monitoring, with opt-out

Requirement (paraphrase): products must provide security-related information
by recording and monitoring relevant internal activity — including access to
or modification of data, services, or functions — with an opt-out mechanism
for the user.

**Applicability:** Applicable.

**Threats addressed:** Repudiation, and information disclosure/tampering
detection after the fact.

**Implemented controls:** The `security.audit_log` table (`AuditLoggingMiddleware`,
recording user, endpoint, action, and response status code for every
state-changing request outside the `/connect/` OAuth paths — which the
middleware still excludes for request-body capture safety, but which now
carry their own dedicated audit path via `AuthenticationAuditor` — see
`§AI-I-2d`) together with structured application logs record
modification of data, services, and functions.

**Residual risk:** Two residuals, stated precisely rather than rounded up:

1. No user-facing opt-out mechanism for this logging exists. This is a
   justified design choice rather than a tracked gap: C-CADA is a
   professional B2B SCADA product, and continuous activity logging is itself
   an operator obligation in regulated industrial environments, not a
   user-optional feature.
2. **Read access is not covered by `security.audit_log`** — only
   state-changing methods are audited (see `§AI-I-2d`), so "access to" data
   (as opposed to modification of it) is not recorded by this mechanism.
   Whether logging coverage should be extended to reads is not yet
   systematically evaluated. Owning item: Testing Strategy (dashboard §4
   queue).

## AI-I-2m — Secure and permanent data removal

Requirement (paraphrase): products must provide the possibility for users to
securely and easily permanently remove all data and settings, and, where
such data can be transferred to other products or systems, ensure that
transfer is done securely.

**Applicability:** Applicable.

**Threats addressed:** Information disclosure (residual data surviving
decommissioning).

**Implemented controls:** Secure decommissioning is a documented procedure —
deleting the PostgreSQL database, removing tenant repository directories,
deleting `~/.cicada-keys`, and allowing browser-held sessions to expire
(`02-user-information.md §II.8(d)`).

**Residual risk:** No built-in, automated "wipe" command exists — removal is
a manual, documented operator procedure rather than a one-click in-product
action. This is accepted as proportionate for an on-prem, professional
deployment rather than tracked as an open item.

## Residual-risk index

Every residual risk named across `§AI-I-1`–`§AI-I-2m` above, collected in one
table with its owning tracked item:

| Residual risk | Requirement | Owning item |
|---|---|---|
| Scan gate report-only, not enforcing (~30 pre-existing findings) | `§AI-I-2a` | CRA Sub-project B triage carry-forward (dashboard §4) |
| CSP default is `ReportOnly`; enforce-mode flip is an operator step | `§AI-I-2b` | Documented commissioning instruction (`02-user-information.md §II.8(a)`) — not a tracked queue item |
| No automatic update install/notification inside the product; external channel only | `§AI-I-2c` | Deliberate design choice (ADR-0139); in-product notification recorded here as a new, currently-untracked residual |
| No anti-rollback protection on signed updates | `§AI-I-2c` | Accepted consequence of ADR-0139 |
| Read-request denials are not audit-logged (`security.audit_log` covers only POST/PUT/PATCH/DELETE) | `§AI-I-2d`, `§AI-I-2l` | Testing Strategy (dashboard §4 queue) |
| Failed authentication attempts against `/connect/token` are not audit-logged (`AuditLoggingMiddleware` excludes `/connect/` paths; no fallback logging in token handler) | `§AI-I-2d` | Resolved 2026-07-05 — `AuthenticationAuditor` (this commit) |
| No `HubAuthzNotifier`/`authzChanged` push on permission change | `§AI-I-2d` | Layout Designer epic carry-forward (dashboard §4) |
| No application-layer encryption at rest | `§AI-I-2e` | Justified deployment-guidance item (`02-user-information.md §II.8(a)`) — not a tracked queue item |
| No general API/OPC UA-southbound rate limiting or DoS hardening | `§AI-I-2h` | Resolved 2026-07-05 — global per-user API rate limit + SignalR hub caps + OPC UA monitored-item ceiling/notification-rate shed (this commit) |
| No high-availability / multi-node failover | `§AI-I-2h`, `§AI-I-2i` | Multi-node cooperative deployment, Sessions D–F (dashboard §4 queue) |
| No backend `AssemblyLoadContext` isolation for third-party backend contributions | `§AI-I-2k` | `package-distribution-and-trust.md` §"Known gaps and deferred hooks" |
| No user-facing logging opt-out | `§AI-I-2l` | Justified design choice — not a tracked queue item |
| No automated "wipe"/secure-removal command | `§AI-I-2m` | Accepted as proportionate — not a tracked queue item |
