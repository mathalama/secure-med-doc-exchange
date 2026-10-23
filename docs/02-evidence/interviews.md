# User Interviews

YouTrack: [DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5) · Guide: [interview-guide.md](interview-guide.md)

> **Status: deferred.** Interviews are being arranged and will be added after the 24 Oct baseline. The table and the records below are filled in only from real interviews. Until then all requirements in [06-requirements.md](../06-requirements.md) remain *Assumption*.

## Summary

| ID | Role | Organization type | Date | Interviewer | Format | Status |
|---|---|---|---|---|---|---|
| P1 | Medical staff | | | | | Deferred |
| P2 | Medical staff | | | | | Deferred |
| P3 | IT / information security | | | | | Deferred |

## Findings across interviews

Fill in after at least three interviews. A finding is **Verified** when it is confirmed by two or more participants, otherwise **Single source**.

| ID | Finding | Participants | Evidence | Requirements |
|---|---|---|---|---|
| F-01 | | | | |

## Assumptions to check

These statements come from the proposal and the requirements. Each interview should confirm or reject some of them.

| ID | Assumption | Related requirements | Result |
|---|---|---|---|
| A-01 | Documents are often sent through e-mail or messengers as scans or photos | Proposal §3 | Not checked |
| A-02 | The sender does not know whether the recipient received and opened the document | FR-12 | Not checked |
| A-03 | The recipient cannot easily check that a document is genuine and unchanged | FR-08, FR-09 | Not checked |
| A-04 | Medical staff already have EDS keys and use NCALayer | FR-07, FR-09 | Not checked |
| A-05 | Patient consent must be confirmed before a document is sent to another organization | FR-05 | Not checked |
| A-06 | Organizations need a log of who opened or sent each document | FR-15 | Not checked |
| A-07 | Access of an employee who leaves is not always revoked on time | FR-02, FR-03 | Not checked |
| A-08 | Users want to find documents by patient identifier (IIN) | FR-13 | Not checked |

---

## Record template

Copy this block for each interview.

```markdown
### P<n> — <role>, <organization type>

| Field | Value |
|---|---|
| Date | YYYY-MM-DD |
| Interviewer / note taker | |
| Format | In person / video call |
| Duration | min |
| Consent to notes | Yes |

**Current process.** How the participant sends and receives documents today.

**Key answers.**
- Q4: …
- Q8: …

**Quotes** (verbatim, anonymized).
- "…"

**Findings.**
| # | Finding | Type (statement / interpretation) | Supports / contradicts |
|---|---|---|---|
| 1 | | | A-01, FR-… |

**Prototype test** (optional). Task, where the participant hesitated, time to complete.
```
