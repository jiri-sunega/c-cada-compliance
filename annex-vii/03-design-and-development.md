# 3. Design, development and production (Annex VII points 2(a), 2(c))

## System architecture overview

```
┌───────────────────────────┐   packages    ┌──────────────────────┐
│ Authoring workstation     │◄──────────────│ Package registry     │
│ Cicada.Authoring.Api      │──────────────►│ (c-cada-marketplace) │
│ + host-served Angular SPA │    publish    │ Cicada.Registry.Api  │
└───────────────┬───────────┘               └──────────────────────┘
                │ `cicada compile`
                ▼
       ┌───────────────────┐
       │ Signed bundle     │
       │ (+ sbom.cdx.json) │
       └─────────┬─────────┘
                 │ deploy / load
                 ▼
               ┌───────────────────────────┐    OPC UA     ┌───────────────┐
               │ Runtime host              │◄─────────────►│ Field devices │
               │ Cicada.Runtime.Host       │               └───────────────┘
               │ + host-served Angular SPA │
               └───────────────────────────┘
```

An engineer authors device types, instances, forms, layouts, graphics, roles,
and device-authority policy on an authoring workstation; `cicada compile`
turns that authored state into a deterministic, signed bundle — including the
application's own `sbom.cdx.json` — that a runtime host loads and executes
against OPC UA field devices southbound (ADR-0128). The package registry
(`c-cada-marketplace`) is consulted during authoring as the trusted source of
third-party packages, not as a runtime dependency
(`c-cada-knowledge/knowledge/security/package-distribution-and-trust.md`).
Both the authoring and runtime APIs serve their own Angular single-page
application directly from the API binary — there is no separate front-end
web server anywhere in the deployed topology (ADR-0120).

## Authoring-to-runtime chain

Authoring persists every entity — device types and instances, forms,
layouts, graphics, roles, and device-authority policy — through a git-backed
materialization layer, so a project's authored state is itself a git history
rather than an opaque database blob. Compiling a project (`cicada compile`)
walks that materialized state and emits a deterministic bundle for the
runtime tier, including the application-level `sbom.cdx.json` that joins with
the runtime product's own SBOM to form the complete deployed picture
(ADR-0128).

The runtime host loads a compiled bundle and executes it: OPC UA southbound
connectivity, the unified state bus, and the operator UI all run against
that loaded bundle rather than against live authoring state
(`c-cada-knowledge/knowledge/architecture/layout-designer-container-model.md`,
`c-cada-knowledge/knowledge/architecture/unified-state-bus.md`).

## Package distribution and trust chain

A package's trust chain runs keygen → pack → publish → verified install, and
trust always originates with the publisher's signature rather than with the
registry that happens to host it. An author generates an Ed25519 key
(`cicada keygen`), signs a package locally at `pack` time, and `publish`es it
to the registry using a dedicated publish token; the registry itself only
verifies that the uploaded signature is well-formed, then binds that signing
key to the package's namespace on its first-ever publish
(trust-on-first-use key binding) as an anti-squatting measure, not a trust
assertion (ADR-0116). Every consumer re-verifies, locally, against its own
`trusted-publishers.json` at install time and again at every subsequent
compile, so a publish being accepted by the registry is never itself a trust
signal.

The platform's own reference packages (`base.*`/`koworx.*`) distribute
through a separate repository, `c-cada-modules`, using a two-key model
distinct from the per-publisher signing keys above: a committed development
key for local iteration and a secret release key for real publishes
(ADR-0118).

## Unified state bus and frontend exposure

The runtime consolidates what were four parallel, per-topic live-state
mechanisms (device tags, feature state, alarms, events) into one backend
`StateHub`/`IStateBus` and one frontend `StateClientService.resolve()` seam
that every binding site now uses (ADR-0084, ADR-0085). What actually reaches
a connected browser is gated a second, earlier way: frontend exposure is
usage-driven and computed at compile time — a property only crosses the wire
if some authored binding actually references it (`exposeToFrontend`,
`StateExposurePass` → `state-exposure.compiled.json`) — and this exposure
gate is a bandwidth scope, not an access-control mechanism (ADR-0086).

Per-user read-egress authorization (below) runs after the exposure gate and
is unaffected by it: a broader exposure glob only ever changes how much data
an already-authorized user's client transfers, never who is authorized to
see it.

## Authorization layers

Access control composes four layers in sequence: authentication via
OpenIddict OIDC establishes who a caller is; role-based access control
(RBAC) then determines a user's assigned roles; layered authorization
narrows that further with a `<source>:<path>` glob grammar scoping grants
per path, where a write grant implicitly also grants read for the same
scope (read = read ∪ write) and read-egress is enforced per-user on both
streaming and REST reads
(`c-cada-knowledge/knowledge/security/layered-authorization.md`;
ADR-0067–ADR-0071). Device authority is a further, orthogonal gate specific
to runtime device connections: an IP/FQDN allow-list composed on top of
user RBAC, with an mTLS client-certificate facet as the strongest rung of
that identity check (ADR-0107, ADR-0108).

Each layer supersedes package defaults without erasing them: package
defaults are superseded at compile time by the project's role model; the
runtime reconciles role→permission grants to the deployed bundle
idempotently on every boot and never edits them itself (ADR-0067). The
project's role model is authoring-only and bundle-authoritative — the
runtime itself only ever edits user→role assignments, never
role→permission mappings (ADR-0067, ADR-0068).

Audit trail coverage for this chain splits across two purpose-built paths
rather than one: `AuditLoggingMiddleware` records generic state-changing
REST requests, but skips everything under `/connect/` outright for
request-body capture safety, since the OAuth form body there carries
credentials the middleware must never see. The `/connect/token` endpoint —
where authentication itself happens — is instead audited directly by
`AuthorizationController` through `AuthenticationAuditor`, a dedicated,
credential-free path (identifiers and a failure reason only, never
password/token material) that enqueues to the same `security.audit_log`
table (`05-risk-assessment.md §AI-I-2d`).

## Browser security controls

Both authoring and runtime ship a strict, nonce-based Content-Security-Policy
(`script-src 'self' 'nonce-…'`, no `unsafe-inline`) together with Trusted
Types, delivered as one comprehensive slice whose default posture is
`ReportOnly` — flipping to enforce mode is a deliberate operator step, not a
shipped default (ADR-0120, ADR-0121). Both products' Angular single-page
applications are host-served directly by their own API binary rather than
through a separate front-end web server, which is what makes a same-origin
nonce CSP practical in the first place.

Session and token handling layers a further hardening option on top: a
`Security:TokenHardening` toggle can move the refresh token into an
`HttpOnly`, `SameSite=Strict` cookie with the access token kept in memory
only (`Cookie` mode), and can additionally bind the access token to the
client's mTLS certificate (`CookieMtls` mode) — both modes default off,
byte-for-byte identical to the pre-hardening behavior unless an operator
opts in (ADR-0122, ADR-0123, ADR-0124).

## Availability and DoS-hardening bounds

The runtime's externally-reachable surfaces are each bounded against
resource exhaustion rather than left open-ended: a global per-user rate
limit on every `/api/v1` endpoint, a SignalR inbound message-size cap and
per-connection live-subscription ceiling on `StateHub`, and, southbound, a
per-connector OPC UA monitored-item ceiling paired with a notification-rate
shed that drops excess inbound data-change traffic. All three are
availability-first by construction: each clamps or sheds the excess rather
than tearing down the connection or bundle it is protecting, so a capped
API caller, a capped hub connection, and a capped OPC UA connector all stay
usable on whatever fits within bounds. This is additive to the OPC UA
client's pre-existing baseline resilience (exponential-backoff reconnect,
a per-item server-side queue, a per-subscription publish cap), which this
pass leaves unchanged. Full detail, defaults, and the honest scope of what
remains open (multi-node failover) live in
`05-risk-assessment.md §AI-I-2h`.

## Signed platform updates

Platform updates ship as signed release artifacts: each `v*` release carries
an Ed25519-signed release manifest binding the release's tag, version, and
explicit archive contents, verified independently by an operator before
installation (`cicada verify-release`, or the openssl-only path documented
in `SECURITY.md`) — full detail on the signing chain, key model, and the
deliberate absence of anti-rollback protection lives in
`04-vulnerability-handling.md §AI-II-7` (ADR-0139).

## Development and production process (2(c))

Development is a solo-maintainer flow committing directly to `main` — there
is no feature-branch or pull-request review gate, since CI itself is the
quality gate. Every push and pull request runs the full backend test suite
(9 test projects across the five backend solutions, ~1,400+ tests)
alongside the Angular frontend's Karma unit tests, a dependency-review
check that blocks pull requests introducing
High-or-above-severity vulnerable dependencies, and a full-tree dependency
vulnerability scan (`.github/workflows/ci.yml`,
`.github/workflows/security-scan.yml`). The vulnerability scan and the
release pipeline additionally run on a weekly schedule even absent any code
change or tag push, so neither gate can silently rot between releases.

The release pipeline is itself the validated production process for
shipping a signed update: it builds the per-product SBOMs and update
archives, signs an explicit, enumerated asset list into a release manifest,
and re-verifies that manifest against the release public key before
anything is published — a tag push that fails re-verification never reaches
a GitHub Release (`.github/workflows/release.yml`). Process "validation" in
the Annex VII sense rests on two continuously-repeated mechanisms rather
than a one-time audit: a sign→verify→tamper-detect self-test that runs on
every CI build, and a weekly dry-run of the full release pipeline against a
non-tag build, which together keep the pipeline's correctness continuously
demonstrated rather than assumed
(`.github/workflows/ci.yml`, `.github/workflows/release.yml`).
