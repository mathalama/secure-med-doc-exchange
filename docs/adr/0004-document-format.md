# ADR 0004: Document format — HL7 FHIR R4 DocumentReference

- **Status:** Accepted (revised 2026-10-23: limitation about national requirements added)
- **Date:** 2026-10-19
- **Task:** [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2)

## Context

Medical organizations use different information systems. Exchanged documents need a standard structured description (patient, author, organization, type, date, integrity data) so that other systems can process them. Implementing a complete health data standard is beyond the scope of the project. No national requirement of the Ministry of Health of the RK to use HL7 FHIR was found during the analysis. Detailed comparison: [05-technology-comparison.md](../05-technology-comparison.md#decision-5-document-format).

## Decision

Describe each exchanged document with an **HL7 FHIR R4 `DocumentReference`** resource. The original file (PDF or image) and its CMS signature are stored in file storage and referenced from `DocumentReference.content.attachment` together with the file hash. The `fhir.resources` library is used for validation. A full FHIR server is not implemented.

## Alternatives considered

- **HL7 CDA R2** — HL7 standard of an earlier generation with a complex XML structure.
- **Custom JSON format** — easiest to implement, but not interoperable with other systems.

## Consequences

- (+) Documents are described in an international standard and can be integrated with FHIR-based systems later.
- (+) Only one resource type is needed, which keeps the implementation small.
- (–) Only a subset of FHIR is supported; this limitation is stated in the project scope.
- (–) Integration with national systems of the RK may require mapping to their formats. The literature review ([03-literature-review.md §2.7](../03-literature-review.md#27-kazakhstan-national-context-and-exchange-formats), DIP-7) found that the 2013–2020 e-health concept targeted HL7 v3 and that no public FHIR requirement exists; mapping to national classifications is future work.
