# 9. EU declaration of conformity (Annex VII point 7)

Annex VII point 7 requires the technical documentation to contain "a copy of
the EU declaration of conformity"
(`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
line 3023). This chapter does not issue that declaration. What it does is the
classification analysis Sub-project D deferred to Sub-project E: whether a
C-CADA product is unlisted, an "important" product with digital elements
(Annex III), or a "critical" one (Annex IV), and which conformity assessment
route follows from that classification. Every statement below is phrased as
the route the manufacturer **will** follow, never as a claim that a C-CADA
product **is** conformant — the actual declaration is a separate, signed,
dated document (Art. 28) drawn up under the manufacturer's sole
responsibility, and it remains a draft until Sub-project E's go-live step
(`docs/compliance/go-live-checklist.md`) is completed. Nothing in this
chapter, or elsewhere in this dossier, is or should be read as that
declaration (`05-risk-assessment.md §Method` establishes the same
never-"complies" discipline for every other chapter's verdicts).

## Classification against Annex III and Annex IV

The adopted Annex III / Annex IV lists
(`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
lines 2771–2899, regulation text as published, OJ L, 2024/2847, 20.11.2024):

- **Annex III Class I** ("important products with digital elements"):
  identity management systems and privileged access management software and
  hardware (including authentication and access-control readers, including
  biometric readers); standalone and embedded browsers; password managers;
  software that searches for, removes, or quarantines malicious software; VPN
  products; network management systems; SIEM systems; boot managers; PKI and
  digital-certificate-issuance software; physical and virtual network
  interfaces; operating systems; routers, modems intended for connection to
  the internet, and switches; microprocessors, microcontrollers, ASICs, and
  FPGAs with security-related functionality; smart-home general-purpose
  virtual assistants; smart-home products with security functionality (door
  locks, security cameras, baby monitoring systems, alarm systems);
  internet-connected toys with social-interactive or location-tracking
  features; personal health-monitoring wearables (outside Regulation (EU)
  2017/745 or 2017/746) or wearables intended for children.
- **Annex III Class II**: hypervisors and container runtime systems; firewalls
  and intrusion detection/prevention systems; tamper-resistant
  microprocessors; tamper-resistant microcontrollers.
- **Annex IV** ("critical products with digital elements"): hardware devices
  with security boxes; smart-meter gateways within smart metering systems
  (and other devices for advanced security purposes, including secure
  cryptoprocessing); smartcards or similar devices, including secure
  elements.

SCADA / industrial-automation-and-control software — C-CADA's core
function — appears in none of these categories in the adopted act.

**The one arguable hook.** Annex III Class I item 1 covers "identity
management systems and privileged access management software and hardware."
C-CADA embeds exactly such a subsystem: the Security Manager /
Device-Authority layered-authorization model
(`03-design-and-development.md#authorization-layers`). But the Annex III
categories attach to a product's core function, not to every security
control a product happens to contain, and C-CADA's core function is SCADA
application authoring and execution — the identity/access-management
subsystem is an internal security control serving that function, not a
general-purpose identity-management or PAM product placed on the market in
its own right. On that reading, item 1 does not reach C-CADA.

**Conclusion: C-CADA (authoring and runtime) is not listed in Annex III or
Annex IV.**

**Honest uncertainty.** The "core function" reading above is this dossier's
own interpretation; the Commission has not yet issued guidance clarifying how
Annex III's categories apply to embedded subsystems within a differently
purposed product, and none is cited here because none exists in the archived
text. This analysis records the regulation text version and date it was made
against (OJ L, 2024/2847, 20.11.2024, as archived) and should be re-run if
Commission guidance on Annex III interpretation narrows or widens that
reading — independent of any product change on C-CADA's side. Tracked as a
go-live checklist item (`docs/compliance/go-live-checklist.md`).

## Conformity assessment route

Because C-CADA's products are not listed in Annex III or Annex IV, only
Art. 32(1) applies to them; the additional, stricter procedures Art. 32(2)–(4)
layer onto listed products (module B+C, module H, or a certification scheme)
do not attach
(`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
lines 1575–1626). Art. 32(1) lets the manufacturer choose among four
procedures, the first of which is "the internal control procedure (based on
module A)" (same source, line 1579).

**The manufacturer will follow module A — internal control, Annex VIII Part
I** (same source, line 3033). No notified body is involved: the manufacturer
assesses and declares conformity under its own sole responsibility. This is
the route this chapter's classification analysis supports; it is a statement
of the route to be followed, not a conformity assertion.

## Article 32(5) fallback (free and open-source software)

C-CADA qualifies as free and open-source software under the Art. 3(48)
definition — source code openly shared, licensed (AGPL-3.0-only, the SPDX
canonical per ADR-0137) under terms granting all rights to freely access,
use, modify, and redistribute it
(`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
line 895). koworx.net's parallel commercial license does not remove that
qualification: the AGPL-3.0-only grant made to recipients independently
satisfies Art. 3(48) regardless of what other license is also offered.

That same dual licensing is what brings C-CADA within the CRA's scope at
all. Recital 18 draws the scope line at monetization: FOSS that a
manufacturer does not monetize is not supplied "in the course of a
commercial activity" and so is not "making available on the market"
(Art. 3(22)); FOSS that is monetized is (recital 18, same source, line 123).
C-CADA is monetized via its commercial license, so the CRA applies to it
despite its FOSS status. This has been the workstream's working premise
since it was chartered — ADR-0125 decomposed the CRA workstream into six
sub-projects on that premise — and this chapter is the first place the
premise is checked directly against the regulation text rather than asserted
informally.

On top of that baseline, Art. 32(5) is a fallback available even under a
contested classification: manufacturers of FOSS products **in the Annex III
categories** may use any Art. 32(1) procedure — including module A —
provided the Art. 31 technical documentation is made available to the
public at the time of placing the product on the market (same source, line
1627; the Art. 32(5) carve-out does not extend to Annex IV). This dossier is
published on the compliance mirror (`00-overview.md`;
https://github.com/jiri-sunega/c-cada-compliance), so module A remains
available under Art. 32(5) even on a contested Annex III Class I reading of
the identity/access-management hook above. Both readings — "not listed" and
"listed but FOSS-published" — converge on module A; Art. 32(5) is recorded
here as insurance, not as this chapter's primary basis for the route
(decision recorded in the sub-project E ADR).

## CE marking

Art. 30(1) requires CE marking to be affixed visibly, legibly, and
indelibly to the product; for products with digital elements in the form of
software, it is affixed either to the EU declaration of conformity (Art. 28)
or on the website accompanying the software product — in practice, the
product's download/distribution page — with that section of the site easily
and directly accessible to consumers
(`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
line 1543). Because module A involves no notified body, no notified-body
identification number will need to follow the marking (Art. 30(4), same
source line 1549). CE marking is affixed before the product is placed on
the market (Art. 30(3), same source line 1547) — for C-CADA, that means at
go-live, not before (`docs/compliance/go-live-checklist.md`).

## Declaration of conformity documents

Two draft EU declarations of conformity — one per platform product —
accompany this dossier:

- `docs/compliance/doc/EU-DoC-c-cada-authoring.md`
- `docs/compliance/doc/EU-DoC-c-cada-runtime.md`

(created by Task 2; both remain **drafts** until signed, dated, and
published at go-live.) `c-cada-marketplace` (registry) does not get a DoC:
per ADR-0139 C-6, the registry is koworx-hosted and never placed on the
market ("beneficial to have it fully under our control"); it keeps its
product SBOM per ADR-0128/0129 and rejoins the manifest only if an
air-gapped mirror ever ships.

Per Annex V
(`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
lines 2905–2943), each draft DoC will contain:

1. Name and type, and any additional information enabling the product's
   unique identification.
2. Name and address of the manufacturer (or its authorised representative).
3. A statement that the declaration is issued under the sole responsibility
   of the provider.
4. Object of the declaration — identification of the product allowing
   traceability (which may include a photograph, where appropriate — not
   applicable to C-CADA's software-only products).
5. A statement that the object of the declaration is in conformity with the
   relevant Union harmonisation legislation.
6. References to any relevant harmonised standards, common specifications,
   or cybersecurity certification schemes used — or, since
   `07-standards-and-solutions.md` records that none of those apply yet, a
   cross-reference to that chapter's "descriptions of solutions adopted"
   route instead.
7. Where applicable, the name and number of the notified body, a description
   of the conformity assessment procedure performed, and identification of
   the certificate issued — not applicable under module A (no notified body
   involved; see "Conformity assessment route" above).
8. Additional information, and the signature block (place and date of
   issue; name, function, and signature) — blank until the manufacturer
   signs at go-live.

Once issued, the II.6 public address (`02-user-information.md §II.6`) will
point to the mirror repository where each signed DoC is published:
https://github.com/jiri-sunega/c-cada-compliance — satisfying both Annex II
item 6 ("where applicable, the internet address at which the EU declaration
of conformity can be accessed,"
`c-cada-knowledge/sources/archive/SRC-eu-cra-regulation-2024-2847/extracted.md`
line 2731) and the Art. 32(5) publication condition above.
