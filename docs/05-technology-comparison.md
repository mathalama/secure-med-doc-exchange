# Technology Comparison

YouTrack: [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2)

This document compares alternatives for the key technical decisions of the project. Each decision is evaluated against criteria derived from the project requirements, not against general popularity. The final decisions are recorded as ADRs in [`docs/adr/`](adr/).

## Project requirements that drive the decisions

| Code | Driver | Source |
|---|---|---|
| D1 | Documents are signed with the user's EDS issued by NCA RK; signatures and certificate status are verified before a document is accepted | Law of the RK No. 370-II of 07.01.2003 "On Electronic Document and EDS"; proposal §5 |
| D2 | Medical data must be protected: encryption, access control, audit | Law of the RK No. 94-V of 21.05.2013 "On Personal Data and Their Protection"; Code of the RK No. 360-VI of 07.07.2020, Art. 60; proposal §3 |
| D3 | Documents must be exchanged in a standard structured format (interoperability) | Proposal §3, §7 |
| D4 | Several organizations, users and roles; consistent data (statuses, audit) | Proposal §8 |
| D5 | Working prototype must be built by a 3-person team within one academic year | Project constraints |
| D6 | Simple deployment for demonstration and testing (Docker); maintained components only | Verification plan |

> The drivers will be linked to requirement IDs from `docs/06-requirements.md` after [DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5) and [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) are completed.

Rating scale used in the tables: **++** strong advantage, **+** suitable, **–** weakness, **– –** serious limitation.

---

## Decision 1. EDS signing and verification

### Background

- NCA RK provides the KalkanCrypt library officially for **Java and C** [1]. There is no official Python SDK; `pykalkan` is a third-party wrapper (latest version 1.2.0), and the Safety vulnerability database marks it as having known vulnerabilities [2].
- In web applications users sign documents with their own keys through **NCALayer**, an NCA RK application that runs on the user's workstation and exposes a local WebSocket API (`wss://127.0.0.1:13579`) to the browser [3].
- NCA RK is moving to the GOST 34.10-2015 algorithm; test user keys are issued on GOST algorithms [4].

### Alternatives

1. **Signing in the browser through NCALayer + verification in a separate signature service based on the official KalkanCrypt Java SDK**
2. **Signing in the browser through NCALayer + verification inside the Python API with `pykalkan`**
3. **Signing on the server with users' keys stored on the server**

### Comparison

| Criterion | Driver | 1. NCALayer + Java service | 2. NCALayer + `pykalkan` | 3. Server-side signing |
|---|---|---|---|---|
| Legal validity / non-repudiation (the signer controls the key) | D1 | ++ | ++ | – – — the key is held by the system, not by the signer |
| Protection of private keys | D2 | ++ — keys never leave the user's device | ++ | – – — keys of all users in one place |
| Official vendor support of the cryptographic library | D1 | ++ — official SDK | – — third-party wrapper | + — depends on the library used |
| Security of dependencies | D2 | + | – — known vulnerabilities reported | + |
| Support of NCA RK algorithms incl. GOST 34.10-2015 | D1 | ++ — updated by NCA RK | – — depends on the wrapper maintainer | + |
| Implementation effort | D5 | – — an additional Java service | + — one codebase | – — key storage and HSM-level protection required |
| Deployment | D6 | + — one more container | ++ | – |

### Decision

**Alternative 1.** The user signs the document in the browser with their own EDS through NCALayer (CMS signature). The backend sends the document and the signature to a small **signature service** written in Java on the official KalkanCrypt SDK, which verifies the signature, the certificate chain and the certificate status (OCSP/CRL) and returns the signer's data. The service has a narrow internal REST interface (`verify`, `certificate-info`) and no access to the database.

ADR: [0005-eds-signing-and-verification.md](adr/0005-eds-signing-and-verification.md)

---

## Decision 2. Main server platform (API)

The main API implements organizations, users, roles, documents, statuses, audit and FHIR resources. EDS verification is delegated to the signature service (Decision 1), so the EDS library no longer constrains the choice of the main platform.

### Alternatives

1. **Python 3.12 + FastAPI**
2. **Java 21 + Spring Boot**
3. **Node.js + NestJS (TypeScript)**

### Comparison

| Criterion | Driver | FastAPI | Spring Boot | NestJS |
|---|---|---|---|---|
| Direct use of the official KalkanCrypt SDK | D1 | – — not available; through the signature service | ++ — official Java SDK | – — not available; through the signature service |
| Security features (authN/authZ, validation) | D2 | + — OAuth2/JWT utilities, Pydantic validation; RBAC implemented in code | ++ — Spring Security, mature RBAC | + — Passport, guards |
| API documentation | D3, D5 | ++ — OpenAPI generated automatically | + — springdoc-openapi | + — Swagger module |
| FHIR libraries | D3 | + — `fhir.resources` | ++ — HAPI FHIR | – |
| Development speed / amount of code | D5 | ++ | – — more configuration and code | + |
| Team experience | D5 | ++ — main language of the backend lead | + | – |
| Performance for the expected load (tens of organizations) | D4 | + — async, sufficient | ++ | + |

### Decision

**Python + FastAPI** for the main API.

Spring Boot would allow using the KalkanCrypt SDK directly in one service, but with EDS isolated in the signature service this advantage is limited to one small component. FastAPI gives a faster development cycle, automatic OpenAPI documentation and matches the experience of the team (driver D5). The cost of this decision is one additional Java service, which is accepted.

ADR: [0001-server-platform.md](adr/0001-server-platform.md)

---

## Decision 3. Data storage

Two kinds of data are stored: **structured data** (organizations, users, roles, document metadata, statuses, audit log) and **files** (document contents and CMS signatures).

### 3a. Database for structured data

#### Alternatives

1. **PostgreSQL**
2. **MongoDB**
3. **MySQL**

#### Comparison

| Criterion | Driver | PostgreSQL | MongoDB | MySQL |
|---|---|---|---|---|
| ACID transactions across entities (status + audit in one transaction) | D4 | ++ | + — multi-document transactions exist but are more limited | ++ |
| Relational integrity (organizations → users → documents) | D4 | ++ | – — references are not enforced | ++ |
| Storing FHIR JSON resources | D3 | ++ — `JSONB` with indexes | ++ — native JSON | + — `JSON` type, weaker indexing |
| Security features (roles, row-level security, encryption extensions) | D2 | ++ — RLS, `pgcrypto` | + | + |
| Append-only audit log | D2 | ++ — triggers, revoked UPDATE/DELETE | + | + |
| Python ecosystem (SQLAlchemy, Alembic) | D5 | ++ | + — Motor/Beanie | ++ |

#### Decision

**PostgreSQL.** It combines strict relational integrity and transactions (needed for organizations, roles, statuses and audit) with `JSONB` for storing FHIR resources, so a separate document database is not needed.

### 3b. File storage

Files are encrypted by the application (AES-256-GCM) **before** they are written to storage, so confidentiality does not depend on the storage product. The storage is accessed through the S3 API so that it can be replaced without code changes.

**MinIO** was considered first, but it is excluded: the community edition stopped receiving official Docker images in October 2025, went into maintenance mode in December 2025 and its repository was archived in 2026, so it no longer receives security fixes [5]. This contradicts driver D6.

#### Alternatives

1. **SeaweedFS** (S3-compatible, Apache 2.0)
2. **Garage** (S3-compatible, AGPL-3.0)
3. **Files inside PostgreSQL (`bytea`)**

#### Comparison

| Criterion | Driver | SeaweedFS | Garage | PostgreSQL `bytea` |
|---|---|---|---|---|
| Active maintenance and maturity | D6 | ++ — mature, widely used [6] | + — considered production-stable, smaller community [7] | ++ |
| S3 API (replaceable with any S3 storage) | D6 | ++ — high S3 compatibility [6] | + — core S3 API | – – |
| Access control (separate credentials and buckets per service) | D2 | + | + | + — only through the API |
| Scalability and DB size | D4 | ++ | + — masterless design limits scaling [7] | – — large files bloat the DB and backups |
| Licence | D5 | ++ — Apache 2.0 | + — AGPL-3.0 | ++ |
| Deployment simplicity | D6 | + — one container in single-node mode | ++ — single binary | ++ — no extra service |

#### Decision

**SeaweedFS** with application-level encryption. PostgreSQL stores only metadata, the object key and the SHA-256 hash of each file to detect tampering. Garage remains a fallback, as both are accessed through the same S3 API.

ADR: [0002-data-storage.md](adr/0002-data-storage.md)

---

## Decision 4. Component interaction

Interaction between the web client and the backend, between the backend and the signature service, and between the system and external systems of medical organizations.

### Alternatives

1. **REST API (HTTPS + JSON)**
2. **gRPC**
3. **Message broker (RabbitMQ) as the main interaction method**

### Comparison

| Criterion | Driver | REST | gRPC | RabbitMQ |
|---|---|---|---|---|
| Compatibility with HL7 FHIR (FHIR defines a RESTful API) | D3 | ++ | – – | – |
| Browser support (web client) | D4 | ++ | – — needs a gRPC-Web proxy | – – — needs a gateway |
| Integration with external medical systems | D3 | ++ — universal | – — both sides need generated stubs | + — both sides need the broker |
| Security (TLS, OAuth2/JWT, standard tooling) | D2 | ++ | + | + |
| Testability (Postman, automated API tests) | D5, D6 | ++ | + | – |
| Asynchronous delivery and retries | D4 | – — must be implemented | + — streaming | ++ |
| Implementation effort | D5 | ++ | + | – |

### Decision

**REST API over HTTPS with JSON**, documented with OpenAPI, for the web client, external systems and the internal call to the signature service. It is required by FHIR, works directly from the browser and is the simplest option for integrations and testing. Asynchronous tasks (notifications) are handled inside the backend with background tasks; a message broker can be added later if the load requires it. The browser talks to NCALayer through its local WebSocket API (Decision 1).

ADR: [0003-component-interaction.md](adr/0003-component-interaction.md)

---

## Decision 5. Document format

### Alternatives

1. **HL7 FHIR R4 `DocumentReference` (metadata) + original file (PDF)**
2. **HL7 CDA R2 (XML documents)**
3. **Custom JSON format**

### Comparison

| Criterion | Driver | FHIR DocumentReference | CDA R2 | Custom JSON |
|---|---|---|---|---|
| International standard, current development | D3 | ++ — current HL7 standard; for example, Uzbekistan's Ministry of Health publishes a FHIR implementation guide [8] | + — HL7 standard of an earlier generation | – – |
| Ease of implementation | D5 | + — only one resource type is needed | – – — complex XML structure | ++ |
| Fits a REST API and JSON storage | D3, D4 | ++ | – | ++ |
| Support for any file type (PDF, images) | D4 | ++ — `attachment` | + | + |
| Libraries in Python | D5 | + — `fhir.resources` | – | ++ |

### Decision

**FHIR R4 `DocumentReference`** for document metadata (patient, author, organization, type, date, hash, link to the file), with the original file and its CMS signature stored in file storage. A full FHIR server is out of scope; only the resources needed for exchange are implemented.

Limitation: no national requirement of the Ministry of Health of the RK to use HL7 FHIR was found during this analysis; FHIR is chosen as an international interoperability standard. National exchange formats are reviewed in the literature review ([03-literature-review.md §2.7](03-literature-review.md#27-kazakhstan-national-context-and-exchange-formats)): the 2013–2020 e-health concept targeted HL7 v3, and no public FHIR requirement was found.

ADR: [0004-document-format.md](adr/0004-document-format.md)

---

## Summary of the selected stack

| Layer | Technology |
|---|---|
| Main API | Python 3.12, FastAPI, SQLAlchemy, Alembic, Pydantic |
| EDS signing | NCALayer on the user's workstation (CMS signature in the browser) |
| EDS verification | Signature service: Java, official KalkanCrypt SDK (NCA RK), OCSP/CRL checks |
| Database | PostgreSQL |
| File storage | SeaweedFS (S3 API), application-level AES-256-GCM encryption |
| Interaction | REST API (HTTPS, JSON), OpenAPI, JWT authentication |
| Document format | HL7 FHIR R4 `DocumentReference` |
| Web client | Defined in the architecture (DIP-3) together with the frontend developer |
| Deployment | Docker, Docker Compose |

## Sources

1. NCA RK. KalkanCrypt certificate of conformity (Java and C providers). https://pki.gov.kz/docs/razrabotka/certificate_kaz.pdf
2. Safety DB. pykalkan changelog and vulnerability status. https://data.safetycli.com/changelogs/pykalkan
3. NCA RK. NCALayer documentation. https://pki.gov.kz/docs/nl_eng/
4. NCA RK. News, February 2022: updated SDK and test keys for GOST 34.10-2015. https://pki.gov.kz/en/2022/02/
5. MinIO ends active development of the community edition. https://linuxiac.com/minio-ends-active-development/
6. RustFS vs SeaweedFS vs Garage: which MinIO alternative should you pick? https://blog.elest.io/rustfs-vs-seaweedfs-vs-garage-which-minio-alternative-should-you-pick/
7. Open edX: replacing MinIO in tutor-minio — why Garage was chosen. https://discuss.openedx.org/t/replacing-minio-in-tutor-minio-why-we-chose-garage-and-what-comes-next/18868
8. Uzbekistan Digital Health Platform FHIR Implementation Guide. https://build.fhir.org/ig/uzinfocom-org/digital-health-ig/
