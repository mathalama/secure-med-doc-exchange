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

_To be completed in [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) by the Security Analyst (risk, probability, impact, mitigation, owner)._

Risks already identified during planning:

| Risk | Source |
|---|---|
| Delayed access to the KalkanCrypt SDK / NCA RK test keys; transition to GOST 34.10-2015 | [ADR 0005](adr/0005-eds-signing-and-verification.md) |
| NCALayer not available in the demo environment | [ADR 0005](adr/0005-eds-signing-and-verification.md) |
| Limited access to real users for interviews and validation | [Proposal §10](01-proposal.md#10-main-risks-summary) |
| Two backend services in two languages increase integration and deployment effort | [ADR 0001](adr/0001-server-platform.md) |
| Limited time of a three-person team | [Proposal §10](01-proposal.md#10-main-risks-summary) |
