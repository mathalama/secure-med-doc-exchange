# Supervisor Feedback

YouTrack: [DIP-4](https://medexchange-diploma.youtrack.cloud/issue/DIP-4)

**Supervisor:** Bakiyeva Aigerim Muratovna, Assistant Professor

This document records the supervisor's comments on key deliverables and the decisions taken to address them. It is updated after every review.

## Pre-defense 1

### Review request

| Field | Value |
|---|---|
| Requested on | YYYY-MM-DD |
| Requested by | Sagyntai Aikyn |
| Channel | _e-mail / messenger / meeting_ |
| Materials sent | Repository link; [proposal](01-proposal.md), [technology comparison](05-technology-comparison.md), [architecture](07-architecture.md), [requirements](06-requirements.md) |
| Status | **Awaiting feedback** — no response received at the time of the pre-defense 1 submission |

### Comments and decisions

| # | Date | Deliverable | Supervisor's comment | Decision / action | Task | Status |
|---|---|---|---|---|---|---|
| — | — | — | _No comments received yet_ | — | — | — |

The table will be filled in as soon as the feedback is received; resulting changes will be made through pull requests that reference the corresponding YouTrack tasks.

### Internal review before submission

Until the supervisor's feedback is received, the deliverables were checked by the team against the pre-defense 1 requirements. Changes made as a result:

| Deliverable | Finding | Change |
|---|---|---|
| Technology comparison | MinIO community edition is no longer maintained (no security fixes) | File storage changed to SeaweedFS with application-level encryption ([ADR 0002](adr/0002-data-storage.md)) |
| Technology comparison, proposal | Signing was described as server-side; in Kazakhstan users sign with their own keys through NCALayer | Signing moved to the browser (NCALayer), verification to a Java service on the official KalkanCrypt SDK ([ADR 0005](adr/0005-eds-signing-and-verification.md)) |
| Proposal | Legal basis was given without references | Laws added with numbers and dates |
| Technology comparison | National use of HL7 FHIR in Kazakhstan was stated without evidence | Claim replaced with a stated limitation; national formats reviewed in the [literature review](03-literature-review.md) |
| Interviews | Interviews could not be held before 24 Oct | Requirements are marked as assumptions; assumptions to check are listed in [interviews.md](02-evidence/interviews.md) |
