# Project Management

YouTrack: [DIP-3](https://medexchange-diploma.youtrack.cloud/issue/DIP-3) (plan, responsibilities, dependencies), [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) (risk register)

## 1. Tools and links

| Tool | Link / location |
|---|---|
| Repository | https://github.com/mathalama/secure-med-doc-exchange |
| Task tracker | YouTrack project DIP: https://medexchange-diploma.youtrack.cloud/issues/DIP |
| Documentation | [`docs/`](./) |
| Diagrams | [`diagrams/src/`](../diagrams/src/) (editable), [`diagrams/export/`](../diagrams/export/) (PNG) |

## 2. Team and responsibilities

| Member | Role | Areas of responsibility |
|---|---|---|
| Sagyntai Aikyn | Backend Lead, Team Lead | Project planning, repository and YouTrack, proposal, technology choice, architecture, main API, signature service, deployment (Docker), pre-defense submissions |
| Korazbay Alikhan | Security Analyst & QA Engineer | Literature review, existing systems comparison, threat model, security and non-functional requirements, verification plan, test automation, security testing |
| Almasuly Damir | Frontend Developer & UX Designer | User interviews, functional requirements, use cases, UX prototype, web client, NCALayer integration in the browser |

**Supervisor:** Bakiyeva Aigerim Muratovna — reviews key deliverables at each milestone.

### Responsibility matrix (R — responsible, C — contributes, Rv — reviewer)

| Deliverable | Aikyn | Alikhan | Damir |
|---|---|---|---|
| Proposal, technology comparison (DIP-2) | R | Rv | C |
| Architecture, project plan (DIP-3) | R | C | Rv |
| Pre-defense submission (DIP-4) | R | C | C |
| Interviews, functional requirements, use cases (DIP-5) | C | C | R |
| UX prototype (DIP-6) | Rv | — | R |
| Literature review, existing systems (DIP-7) | Rv | R | — |
| Threat model, NFR, verification plan, risks (DIP-8) | C | R | Rv |
| Main API, signature service | R | C (tests) | — |
| Web client, NCALayer integration | C | C (tests) | R |
| Testing and security testing | C | R | C |

## 3. Milestones

Dates of pre-defense 2 and the final pre-defense will be fixed according to the department schedule.

| # | Milestone | Target date | Main results | Git tag |
|---|---|---|---|---|
| M1 | Pre-defense 1: analysis and design | 24.10.2026 | Proposal, literature review, comparisons, requirements, diagrams, prototype, verification plan, YouTrack | `pre-defense-1` |
| M2 | Iteration 1: foundation | 30.11.2026 | Docker Compose environment, database schema, organizations/users/roles, authentication, CI with tests | — |
| M3 | Iteration 2: EDS and exchange | 31.12.2026 | Signature service with NCA RK test keys, NCALayer signing in the web client, sending/receiving documents, encrypted file storage | — |
| M4 | Pre-defense 2: working core | _TBD (department schedule)_ | End-to-end scenario "send → verify → receive → open" demonstrated | `pre-defense-2` |
| M5 | Iteration 3: completeness | _TBD_ | FHIR `DocumentReference`, audit log and its review, statuses, notifications, admin functions | — |
| M6 | Iteration 4: verification | _TBD_ | Security testing per threat model, measurements per verification plan, fixes | — |
| M7 | Final pre-defense | _TBD (department schedule)_ | Complete system, test report, final documentation | `final-pre-defense` |

## 4. Work plan until pre-defense 1

| Task | Responsible | Deadline | Depends on |
|---|---|---|---|
| DIP-1 Repository setup | Aikyn | 19.10.2026 (done) | — |
| DIP-2 Proposal and technology comparison | Aikyn | 24.10.2026, 18:00 | — |
| DIP-3 Architecture and project plan | Aikyn | 24.10.2026, 18:00 | DIP-2 |
| DIP-5 Interviews and functional requirements | Damir | 24.10.2026, 18:00 | — |
| DIP-6 UX prototype | Damir | 24.10.2026, 18:00 | DIP-5 |
| DIP-7 Literature review and existing systems | Alikhan | 24.10.2026, 18:00 | — |
| DIP-8 Security analysis and verification plan | Alikhan | 24.10.2026, 18:00 | DIP-2, DIP-3 |
| DIP-4 Pre-defense 1 submission | Aikyn | 24.10.2026, 18:00 | all of the above |

## 5. Dependencies

### Between tasks

- The architecture (DIP-3) and the threat model (DIP-8) depend on the technology decisions (DIP-2).
- The Use Case diagram (DIP-5), System Context diagram (DIP-8) and Component diagram (DIP-3) must use the same actors and components ([07-architecture.md §2–3](07-architecture.md)).
- The prototype (DIP-6) depends on the functional requirements and use cases (DIP-5).
- Implementation of the web client signing depends on the signature service API (M3).

### External

| Dependency | Needed for | Owner |
|---|---|---|
| KalkanCrypt SDK and NCA RK test keys/certificates (pki.gov.kz) | Signature service, demo | Aikyn |
| NCALayer installed on demo workstations | Signing in the web client | Damir |
| Access to potential users (doctors, registrars, medical IT staff) | Interviews, validation | Damir, Alikhan |
| Supervisor review at each milestone | Pre-defense submissions | Aikyn |

## 6. Working process

- **YouTrack:** every meaningful work item is a task with a responsible student, expected outcome, acceptance criteria and deadline. A task is moved to *Done* only when its acceptance criteria are met and the pull request is merged.
- **Branches:** `main` is the reviewed baseline. Work is done in branches named `DIP-<id>-short-description`.
- **Commits:** `DIP-<id>: <description>`; each commit is a coherent change.
- **Pull requests:** each PR references its YouTrack task, explains the changes and how they were checked. The link to the PR is added to the task.
- **Review:** another team member reviews every PR before merging. Default rotation: Aikyn → reviewed by Alikhan, Alikhan → reviewed by Damir, Damir → reviewed by Aikyn.
- **Tags:** each pre-defense submission is tagged (`pre-defense-1`, `pre-defense-2`, `final-pre-defense`); tags are never moved.
- **Meetings:** short weekly team sync; meeting with the supervisor before each milestone. Supervisor feedback is recorded in [`11-supervisor-feedback.md`](11-supervisor-feedback.md).

## 7. Risk register

Owner of the register: Korazbay Alikhan ([DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8)). The register is reviewed at the weekly sync and at every milestone. Security risks of the system itself are analyzed in the [threat model](security/threat-model.md); this register covers project risks.

Scale: probability (P) and impact (I) are **L** = 1, **M** = 2, **H** = 3; score = P × I. A score of 6 or more is a high risk and is discussed at every weekly sync.

| ID | Risk | P | I | Score | Mitigation | Contingency (if it happens) | Owner | Trigger / indicator | Source |
|---|---|:-:|:-:|:-:|---|---|---|---|---|
| R-01 | Delayed access to the KalkanCrypt SDK or NCA RK test keys; NCA RK transition to GOST 34.10-2015 changes the algorithms | M | H | 6 | Request SDK and test keys in M2; define the verification API first and develop against a stub; test with both RSA and GOST test keys | Demonstrate with test keys only; describe GOST support as a limitation | Aikyn | SDK or keys not received by 15.11.2026 | [ADR 0005](adr/0005-eds-signing-and-verification.md) |
| R-02 | NCALayer does not work in the demo environment (version, browser, certificate) | M | H | 6 | Dedicated demo workstation prepared and checked one week before each defense; NCALayer stub for automated E2E | Recorded video of the signing scenario; stub mode clearly labelled | Damir | Failed rehearsal | [ADR 0005](adr/0005-eds-signing-and-verification.md) |
| R-03 | Limited access to real users for interviews and the usability test | H | M | 6 | Contacts through university clinics and the supervisor; remote sessions; medical students as proxy participants | Requirements stay "Assumption"; limitation stated in the report | Damir (interviews), Alikhan (UT-01) | Fewer than 3 interviews by M2 | [Proposal §10](01-proposal.md#10-main-risks-summary) |
| R-04 | Two backend services in two languages increase integration and deployment effort | M | M | 4 | Narrow internal API (`verify`, `certificate-info`) with contract tests; integration in M3, not at the end | Simplify the signature service to a single endpoint | Aikyn | Integration tests failing for more than one week | [ADR 0001](adr/0001-server-platform.md) |
| R-05 | Limited time of a three-person team | H | H | 9 | MoSCoW priorities; Could requirements dropped first; weekly sync; one-week buffer before every pre-defense | Re-plan the iteration with the supervisor; cut Should scope | Aikyn | Iteration goals missed twice | [Proposal §10](01-proposal.md#10-main-risks-summary) |
| R-06 | A High / Critical vulnerability is found late, close to the final pre-defense | M | H | 6 | CI security scans from M2 (NFR-20); threat-model tests run in each iteration, not only in M6 | Fix window in M6; document a justified exception | Alikhan | Open High finding older than 2 weeks | [Verification plan](09-verification-plan.md) |
| R-07 | Real patient data accidentally enters the repository or the test database | L | H | 3 | Synthetic data generator; secret / pattern scanning in CI; compliance checklist CL-01; rule in the README | Remove from history, rotate any exposed secrets, inform the supervisor | Alikhan | Scanner finding, review comment | NFR-10 |
| R-08 | Secrets (JWT key, file master key, test key passwords) leak through the repository or logs | M | M | 4 | `.env` not committed; gitleaks in CI; log hygiene tests (ST-14) | Rotate keys using the documented procedure (NFR-18) | Alikhan | gitleaks or ST-14 finding | [Threat model](security/threat-model.md) T-23 |
| R-09 | NCA RK test OCSP / CRL services unavailable during tests or the demo | M | M | 4 | Fail-closed handling (NFR-24) is itself demonstrated; cached CRL within validity; schedule tests with retries | Show "verification pending" behavior; repeat the run later | Aikyn | OCSP errors in the signature service logs | ADR 0005, NFR-24 |
| R-10 | Legal requirements (personal data, consent, storage location, EDS) are interpreted incorrectly | M | M | 4 | Requirements cite the laws; questions recorded and checked with the supervisor; only test data processed | Adjust requirements; state the interpretation as a limitation | Alikhan | Supervisor or committee comment | [Proposal §3](01-proposal.md#3-problem-statement) |
| R-11 | A chosen component stops being maintained (as happened with MinIO) | L | M | 2 | Access through the S3 API; Garage kept as a fallback; dependency scanning shows unmaintained packages | Switch to the fallback component | Aikyn | Archived repository, no security fixes | [ADR 0002](adr/0002-data-storage.md) |
| R-12 | A team member is unavailable for several weeks | L | H | 3 | Reviewer rotation gives every area a second person; documentation in the repository | Redistribute tasks; reduce Should scope | Aikyn | Absence longer than one week | Section 2 |
| R-13 | Supervisor feedback arrives late or requires major changes | M | M | 4 | Drafts shared early; feedback recorded in `11-supervisor-feedback.md` | Plan a dedicated rework iteration | Aikyn | No feedback one week before a milestone | Section 6 |
| R-14 | Performance targets (NFR-07) are not met because of encryption and signature verification | L | M | 2 | First measurement in M4; verification is asynchronous and does not block lists | Optimize or relax the target with justification | Aikyn | PT-01 p95 above target | NFR-07 |
| R-15 | Scope creep: full FHIR server, real MIS integration, patient portal | M | M | 4 | Out-of-scope list in [proposal §8](01-proposal.md#8-project-scope); new scope only through YouTrack and the team lead | Move to "future work" | Aikyn | New tasks outside proposal §8 | Proposal §8 |

Status of every risk is tracked in YouTrack (tag `risk`). A risk is closed when its trigger can no longer occur (for example, R-01 when the SDK and keys are received and verified).
