# 8. Test reports (Annex VII point 6)

Annex VII point 6 asks for "reports of the tests carried out to verify the
conformity of the product with digital elements and of the vulnerability
handling processes with the applicable essential cybersecurity requirements
as set out in Parts I and II of Annex I"
(`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
line 3019). C-CADA answers this two ways: a continuous test-and-verification
infrastructure that runs on every change (this chapter's first section), and
a generated, signed, per-release conformity test report designed to carry
that evidence forward for the life of the support period (second section).

## Continuous test and verification infrastructure

- **Backend tests.** Nine test projects across the platform's five backend
  solutions, roughly 1,400 or more tests in total, run via `dotnet test` on
  every push to `main` and every pull request
  (`.github/workflows/ci.yml`). Suite counts as of 2026-07-04: compiler 408,
  runtime 275 (plus 3 skipped), security 75, authoring 658.
- **Frontend tests.** Angular Karma unit tests for the authoring app, the
  runtime app, and the shared library run headless (`ChromeHeadless`) on the
  same triggers via `npx nx run-many -t test --watch=false
  --browsers=ChromeHeadless` (`.github/workflows/ci.yml`;
  `03-design-and-development.md §Development and production process`, which
  already records this as part of the CI gate).
- **Dependency vulnerability scanning.** A full-tree `dotnet`/`npm`
  vulnerability scan plus a pull-request-only dependency-review check that
  blocks the introduction of High-or-above-severity vulnerable dependencies
  (`.github/workflows/security-scan.yml`; `04-vulnerability-handling.md
  §AI-II-3`).
- **Release-pipeline self-verification.** A sign→verify→tamper-detect round
  trip (`release-manifest-selftest` job) runs on every CI build, deliberately
  corrupting a signed asset afterward to confirm the verifier actually
  rejects it, not just that it accepts a valid one
  (`.github/workflows/ci.yml`). The release pipeline itself re-verifies its
  own signed manifest before publishing, so a tag push that fails
  re-verification never reaches a GitHub Release
  (`.github/workflows/release.yml`).
- **Weekly canaries.** Both the vulnerability scan and the full release
  pipeline — the latter as a dev-key-signed dry run — additionally run on a
  Monday schedule independent of any code change or tag push, so neither
  check can silently rot between releases (`.github/workflows/security-scan.yml`,
  `.github/workflows/release.yml`).

## Per-release conformity test report (design)

Every `v*` release is designed to carry `conformity-test-report.json`
(machine-readable) and `conformity-test-report.md` (human-readable),
generated from the same release job's backend test run and added to the
Ed25519-signed `release-manifest.json`'s explicit asset list — the same
signature that already covers the update archives and SBOMs will cover this
test evidence too
(`c-cada-docs/architecture/2026-07-04-cra-d-annex-vii-docs-design.md §5`).
The generator and its `release.yml` wiring are the next two tasks in this
dossier's own implementation plan and are not yet built as this chapter is
written; this section documents the design those tasks implement, not a
shipped artifact.

- **Coverage.** The report is designed to summarize the backend test suites
  whose per-assembly TRX files feed the generator: compiler, runtime,
  security, and authoring (authoring runs as four separate test assemblies —
  Domain, Persistence, Application, Api — each producing its own TRX file so
  none are overwritten).
- **Fail-closed by design.** A missing or unparseable TRX file, or any suite
  carrying a failure, is designed to abort report generation — and therefore
  the release — rather than emit a report that looks clean.
- **Retrieval.** Once published, the report is retrievable from the
  release's GitHub Release page
  (`https://github.com/jiri-sunega/c-cada-platform/releases/tag/<tag>`)
  alongside the other signed assets, or via `gh release download <tag> -p
  "conformity-test-report.*"`.

## Retention rationale

GitHub Actions workflow-run logs expire on a rolling window of roughly 90
days; GitHub Release assets carry no such expiry. This is why the conformity
test report is designed as a signed release asset rather than left as CI log
output: Release assets are the mechanism this dossier's support-period
commitment (`06-support-period.md`) relies on for evidence retention across
the full five-year window, matching the same rationale already recorded for
the SBOM/VEX/update archives (`00-overview.md`; `release.yml`'s own top-of-file
comment records permanent Release-asset retention as "the CRA compliance
record for the 5-year support window").

## Honest gaps

Two things stated plainly rather than rounded up:

- **No end-to-end/integration test suite exists.** CI runs backend and
  frontend unit/integration tests (above), but there is no automated
  end-to-end suite exercising a deployed authoring-and-runtime pair together;
  `samples/_slice4-sandbox-e2e.sh` is a manual sandbox script, not an
  automated gate. Owning item: Testing Strategy (dashboard §4 queue).
- **The designed per-release conformity-test-report is backend-only.** Its
  suite coverage (§ above) is the per-assembly TRX files for the backend
  suites (compiler, runtime, security, authoring); it does not include the
  registry test suite or the frontend Karma results, even though both
  already run as part of ordinary CI (`.github/workflows/ci.yml`). Extending
  the signed per-release evidence to those suites is not yet scoped. Owning
  item: Testing Strategy (dashboard §4 queue).
