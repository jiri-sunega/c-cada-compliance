# 6. Support period determination (Annex VII point 4)

## Committed period

C-CADA commits to **5 years of free security updates from the release of
v0.1.0**: **2026-07-04 to 2031-07-04**. This commitment is published in
`SECURITY.md`'s "Support period" section (ADR-0127).

## Determination factors

CRA Art. 13(8) requires manufacturers to determine a support period that
reflects how long a product is reasonably expected to be in use, taking
into account user expectations, the nature of the product, and relevant
Union law bearing on product lifetime — and sets five years as the floor
for that determination unless the product's own expected use time is
shorter. C-CADA's determination rests on the following factors:

- **Expected use time.** SCADA installations are commissioned once and then
  operated for a long service life — industrial deployments routinely
  remain in service well beyond a decade. Against that backdrop, five years
  is treated as the floor Art. 13(8) sets, not as an estimate of the
  product's actual useful life.
- **Maintainer sustainability.** C-CADA is currently maintained by a single
  owner (koworx.net). A five-year window is the period that owner can
  commit to sustaining without over-promising a support horizon the project
  cannot realistically staff.
- **Runtime substrate cadence.** The authoring and runtime hosts run on
  .NET; each .NET version has its own defined support window, and .NET's
  Long-Term-Support releases bound how long the underlying substrate itself
  keeps receiving upstream security fixes. C-CADA's support-period planning
  tracks which .NET release the platform targets so the platform's own
  window stays inside a substrate that is still supported upstream.
- **Frontend framework cadence.** The Angular frontend framework's own
  long-term-support release cadence is tracked the same way, for the same
  reason on the client side.
- **Re-evaluation, not re-negotiation.** The support period is re-evaluated
  at each major release. Consistent with Art. 13(8)'s five-year floor and
  with the end-date being a commitment users and market surveillance
  authorities rely on once published, re-evaluation can extend the
  committed period but is not used to shorten it below what has already
  been published.
