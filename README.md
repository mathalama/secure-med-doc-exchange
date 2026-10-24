# MedExchange — Secure Medical Document Exchange

Development of a secure system for the exchange of digital documents between medical organizations.

> KZ: Медициналық ұйымдар арасында цифрлық құжаттармен алмасудың қауіпсіз жүйесін әзірлеу
> RU: Разработка защищённой системы обмена цифровыми документами между медицинскими организациями

MedExchange is a web system through which medical organizations send each other digital documents about patients (referrals, discharge summaries, test results, certificates). Every document is signed by the sender with their own electronic digital signature (EDS) of the National Certification Authority of the Republic of Kazakhstan (NCA RK), verified by the system before delivery, stored encrypted, described in the HL7 FHIR format and recorded in an audit log.

## Problem statement

Medical organizations often exchange patient documents outside a single protected channel: on paper, as scans sent by e-mail or through messengers. As a result:

- medical data — a special category of personal data — is exposed to leaks;
- the recipient cannot reliably verify who issued a document and whether it was changed;
- there is no log of who sent, received or opened a document;
- documents are delayed, lost or sent to the wrong recipient;
- documents are not exchanged in a common structured format.

The exchange must comply with the Law of the RK No. 370-II "On Electronic Document and EDS", the Law of the RK No. 94-V "On Personal Data and Their Protection" and the Code of the RK No. 360-VI "On Public Health and the Healthcare System" (Art. 60).

Details: [docs/01-proposal.md](docs/01-proposal.md).

## Target users

| User | Needs |
|---|---|
| Doctor | Send and receive patient documents, sign them with EDS, trust received documents |
| Registrar / medical records staff | Route incoming documents, track their status |
| Organization administrator | Manage users and roles of the organization |
| Security officer | Review the audit log, investigate incidents |

## Expected outcomes

- A working prototype deployable with Docker.
- Exchange of documents between organizations: signing in the browser via NCALayer, server-side verification of the signature and certificate status, delivery to the recipient.
- Role-based access control and isolation of data between organizations.
- Encrypted storage of files and an audit log of all actions.
- Document metadata in HL7 FHIR R4 `DocumentReference`.
- Documentation: requirements, architecture, threat model, test report.

## Team

| Member | GitHub | Role | Responsibilities |
|---|---|---|---|
| Sagyntai Aikyn | [@mathalama](https://github.com/mathalama) | Backend Lead, Team Lead | Planning, proposal, technology choice, architecture, main API, signature service, deployment |
| Korazbay Alikhan | [@AlikShal](https://github.com/AlikShal) | Security Analyst & QA Engineer | Literature review, existing systems, threat model, security requirements, verification plan, testing |
| Almasuly Damir | [@smuzyyq](https://github.com/smuzyyq) | Frontend Developer & UX Designer | User interviews, functional requirements, use cases, UX prototype, web client |

**Supervisor:** Bakiyeva Aigerim Muratovna, Assistant Professor

Detailed allocation of responsibilities: [docs/10-project-management.md](docs/10-project-management.md).

## Scope

**In scope:** web client; REST API; organizations, users and roles; sending, receiving and viewing documents; EDS signing via NCALayer and server-side verification (KalkanCrypt SDK); encryption in transit and at rest; document statuses; audit log; FHIR `DocumentReference` metadata.

**Out of scope:** issuing EDS keys; a full FHIR server; integration with real medical information systems and national eGov services; mobile applications; electronic medical records and other hospital information system functions; real patient data (test data only).

## Current status

**Stage: analysis and design — pre-defense 1 (24.10.2026).** A complete application is not required at this stage; the source code has not been started yet.

| Deliverable | Document | Task | Status |
|---|---|---|---|
| Project proposal and scope | [01-proposal.md](docs/01-proposal.md) | [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2) | Done |
| Evidence of relevance: interview guide | [02-evidence/interview-guide.md](docs/02-evidence/interview-guide.md) | [DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5) | Done |
| Evidence of relevance: interviews | [02-evidence/interviews.md](docs/02-evidence/interviews.md) | [DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5) | Deferred — interviews are being arranged; assumptions to check are listed |
| Literature review (22 sources, 12 scientific) | [03-literature-review.md](docs/03-literature-review.md) | [DIP-7](https://medexchange-diploma.youtrack.cloud/issue/DIP-7) | Done |
| Existing systems comparison | [04-existing-systems.md](docs/04-existing-systems.md) | [DIP-7](https://medexchange-diploma.youtrack.cloud/issue/DIP-7) | Done |
| Technology comparison | [05-technology-comparison.md](docs/05-technology-comparison.md) | [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2) | Done |
| Requirements and acceptance criteria | [06-requirements.md](docs/06-requirements.md) | [DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5), [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) | Done (initial version; marked as assumptions until interviews) |
| Architecture and Component Diagram (draft) | [07-architecture.md](docs/07-architecture.md), [component.png](diagrams/export/component.png) | [DIP-3](https://medexchange-diploma.youtrack.cloud/issue/DIP-3) | Done |
| System Context Diagram | [context.png](diagrams/export/context.png) | [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) | Done |
| Use Case Diagram | [use-case.png](diagrams/export/use-case.png) | [DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5) | Done |
| UX prototype | [08-prototype.md](docs/08-prototype.md), [prototype/](prototype/) | [DIP-6](https://medexchange-diploma.youtrack.cloud/issue/DIP-6) | Done |
| Verification and evaluation plan | [09-verification-plan.md](docs/09-verification-plan.md) | [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) | Done |
| Threat model (STRIDE) | [security/threat-model.md](docs/security/threat-model.md) | [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) | Done |
| Project management, milestones, risks | [10-project-management.md](docs/10-project-management.md) | [DIP-3](https://medexchange-diploma.youtrack.cloud/issue/DIP-3), [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) | Done |
| Supervisor feedback | [11-supervisor-feedback.md](docs/11-supervisor-feedback.md) | [DIP-4](https://medexchange-diploma.youtrack.cloud/issue/DIP-4) | Requested, awaiting response; internal review recorded |

## Selected technologies

| Layer | Technology |
|---|---|
| Main API | Python 3.12, FastAPI, SQLAlchemy, Alembic, Pydantic |
| EDS signing | NCALayer (NCA RK) on the user's workstation |
| EDS verification | Signature service: Java, official KalkanCrypt SDK (NCA RK), OCSP/CRL |
| Database | PostgreSQL |
| File storage | SeaweedFS (S3 API), application-level AES-256-GCM encryption |
| Interaction | REST API (HTTPS, JSON), OpenAPI, JWT |
| Document format | HL7 FHIR R4 `DocumentReference` |
| Deployment | Docker, Docker Compose |

Justification: [docs/05-technology-comparison.md](docs/05-technology-comparison.md) and [architecture decision records](docs/adr/).

## Links

- YouTrack project: https://medexchange-diploma.youtrack.cloud/issues/DIP
- Documentation: [docs/](docs/)
- Architecture decision records: [docs/adr/](docs/adr/)
- Diagrams: [diagrams/src/](diagrams/src/) (editable PlantUML sources), [diagrams/export/](diagrams/export/) (PNG)
- UX prototype: [prototype/src/index.html](prototype/src/index.html) (open in a browser, no server needed), description in [docs/08-prototype.md](docs/08-prototype.md)

## Repository structure

| Path | Content |
|---|---|
| `apps/web/` | Web client |
| `services/api/` | Main API (FastAPI) and its unit/integration tests |
| `services/signature/` | Signature service (Java, KalkanCrypt SDK) — planned |
| `tests/` | End-to-end and security tests |
| `infra/` | Docker and deployment configuration |
| `docs/` | Proposal, literature review, comparisons, requirements, architecture, plans |
| `docs/adr/` | Architecture decision records |
| `docs/security/` | Threat model and security analysis |
| `diagrams/src/` | Editable diagram sources (PlantUML) |
| `diagrams/export/` | Diagram exports (PNG) |
| `prototype/` | Clickable HTML prototype (`src/`) and screenshots |

## Setup, execution and testing

Runnable code is not available yet (analysis and design stage). Setup, run and test instructions will be added here together with the first runnable version (milestone M2, see [project plan](docs/10-project-management.md#3-milestones)).

Configuration will be provided through environment variables; copy [`.env.example`](.env.example) to `.env` and fill in the values. `.env`, keys and certificates must never be committed.

## Workflow

- `main` is the reviewed baseline; changes go through pull requests.
- Branch name: `DIP-<id>-short-description`, e.g. `DIP-2-proposal-and-technology-comparison`.
- Commit message: `DIP-<id>: <description>`.
- Each pull request references its YouTrack task and is reviewed by another team member.
- Each pre-defense submission is tagged; the pre-defense 1 version is tagged `prob-period-1` (names of later tags follow the department submission form).
