# System Architecture

YouTrack: [DIP-3](https://medexchange-diploma.youtrack.cloud/issue/DIP-3)

Status: **draft for pre-defense 1**. The architecture follows the technology decisions in [05-technology-comparison.md](05-technology-comparison.md) and ADRs [0001](adr/0001-server-platform.md)–[0005](adr/0005-eds-signing-and-verification.md).

## 1. Overview

MedExchange is a web system through which medical organizations send each other signed digital documents. Every document is signed by the sender with their own EDS (NCA RK) in the browser, verified by the system before delivery, stored encrypted and described with an HL7 FHIR `DocumentReference`. Every action with a document is written to an audit log.

Architectural style: a **modular monolith** (main API) plus one **dedicated signature service**, deployed with Docker Compose. The monolith keeps development simple for a three-person team; the signature service is separated because the official KalkanCrypt SDK is available only for Java and C (ADR 0005).

## 2. Actors and external systems

These names are used consistently in the System Context diagram ([DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8)) and the Use Case diagram ([DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5)).

| Actor / system | Type | Interaction |
|---|---|---|
| Doctor | User | Creates, signs and sends documents; receives and opens documents |
| Registrar | User | Receives and routes incoming documents, tracks statuses |
| Organization administrator | User | Manages users and roles of their organization |
| Security officer | User | Reviews the audit log of their organization |
| System administrator | User | Registers organizations, manages the platform |
| NCALayer | External software on the user's workstation | Signs data with the user's EDS key (CMS) |
| NCA RK services | External system | OCSP / CRL for certificate status checks |
| External medical information system | External system (future) | Sends and receives documents through the REST API |

## 3. Components

Component diagram: [`diagrams/src/component.puml`](../diagrams/src/component.puml) → `diagrams/export/component.png`. The diagram shows the main components and their connections; the reverse proxy (C2), the internal modules of the API (C3.1–C3.8) and future external systems are described in the tables below.

| # | Component | Technology | Responsibility |
|---|---|---|---|
| C1 | Web client | SPA in the browser (framework selected by the frontend developer) | User interface; calls NCALayer through its local WebSocket API to sign documents; calls the API over HTTPS |
| C2 | Reverse proxy | Nginx | TLS termination, routing to the web client and the API, request size limits |
| C3 | Main API | Python 3.12, FastAPI | Business logic, divided into modules C3.1–C3.8 |
| C3.1 | Auth module | FastAPI, JWT | Authentication, issuing and validating access tokens |
| C3.2 | Organizations & users module | SQLAlchemy | Organizations, users, roles (RBAC), permission checks |
| C3.3 | Documents module | SQLAlchemy | Creating documents, metadata, versions, statuses |
| C3.4 | Exchange module | — | Sending a document to a recipient organization, inbox/outbox, delivery statuses |
| C3.5 | FHIR module | `fhir.resources` | Building and validating `DocumentReference` resources |
| C3.6 | Audit module | PostgreSQL | Append-only audit log with hash chaining of records |
| C3.7 | Storage module | S3 client, AES-256-GCM | Encrypting/decrypting files, writing/reading them in file storage, hashing |
| C3.8 | Signature client | HTTP client | Calls the signature service, stores verification results |
| C4 | Signature service | Java, official KalkanCrypt SDK | Verifies CMS signatures, certificate chains and certificate status (OCSP/CRL); returns signer data |
| C5 | Database | PostgreSQL | Organizations, users, roles, document metadata, FHIR resources (`JSONB`), statuses, audit log |
| C6 | File storage | SeaweedFS (S3 API) | Encrypted document files and CMS signatures |

### Interfaces

| Interface | Provider → consumer | Protocol | Exposed outside |
|---|---|---|---|
| Public REST API (`/api/v1/...`), OpenAPI | C3 → C1, external systems | HTTPS, JSON, JWT | Yes, through C2 |
| NCALayer API | NCALayer → C1 | WebSocket `wss://127.0.0.1:13579` | Only on the user's workstation |
| Verification API (`/verify`, `/certificate-info`) | C4 → C3.8 | HTTP, JSON | No, internal Docker network only |
| SQL | C5 → C3 | PostgreSQL protocol | No |
| S3 API | C6 → C3.7 | HTTP (S3) | No |
| OCSP / CRL | NCA RK → C4 | HTTP | Outbound only |

## 4. Main scenarios (data flows)

### 4.1 Sending a document

1. The doctor fills in the metadata (recipient organization, patient, document type) and attaches a file in the web client (C1).
2. C1 asks NCALayer to sign the file; the doctor selects their EDS key and enters the key password in NCALayer. NCALayer returns a CMS signature. The private key never leaves the workstation.
3. C1 sends the file, the CMS signature and the metadata to `POST /api/v1/documents` (C3).
4. C3 checks the user's permissions (C3.2) and sends the file and signature to the signature service (C4).
5. C4 verifies the signature against the file, the certificate chain and the certificate status at NCA RK (OCSP/CRL), and returns the result and the signer data (name, IIN, organization BIN).
6. If verification fails, the document is rejected and the attempt is logged. If it succeeds, C3 checks that the signer is the current user and belongs to the sender organization.
7. C3.7 computes the SHA-256 hash, encrypts the file and the signature and writes them to file storage (C6).
8. C3.5 creates the FHIR `DocumentReference`; C3.3/C3.4 save the metadata, the verification result and the status `SENT` in PostgreSQL (C5) **in the same transaction** as the audit record (C3.6).

### 4.2 Receiving and opening a document

1. A registrar or doctor of the recipient organization opens the inbox (`GET /api/v1/documents/inbox`); C3 returns only documents addressed to their organization and allowed for their role.
2. On opening, C3.7 reads the encrypted file from C6, decrypts it and compares the SHA-256 hash with the stored value; a mismatch is reported as a tampering incident.
3. The client shows the document and the stored signature verification result (signer, time, certificate status).
4. The status changes to `OPENED`; the action is written to the audit log.
5. Optionally, the recipient confirms receipt by signing an acknowledgement with their EDS (same flow as 4.1, steps 2–6).

### 4.3 Authentication

Proposed approach (to be confirmed by the requirements in DIP-5):

1. The client requests a one-time challenge from C3.
2. The user signs the challenge with their EDS authentication key through NCALayer.
3. C4 verifies the signature; C3 finds the user by the IIN from the certificate and issues a short-lived JWT access token with the user's organization and roles.

### 4.4 Audit review

The security officer requests the audit log of their organization (`GET /api/v1/audit`). Each record contains the time, user, organization, action, document and the hash of the previous record, so that deleting or changing records can be detected.

## 5. Data model (draft)

| Entity | Main attributes |
|---|---|
| Organization | id, name, BIN, status |
| User | id, IIN, full name, organization_id, status |
| Role / UserRole | doctor, registrar, org_admin, security_officer, system_admin |
| Document | id, sender_org_id, recipient_org_id, author_id, patient reference, type, status, file object key, file SHA-256, `fhir_document_reference` (`JSONB`), created_at |
| Signature | id, document_id, signer IIN/BIN, certificate serial, signed_at, verification result, verified_at |
| AuditEvent | id, time, user_id, organization_id, action, document_id, details, previous_hash, hash |

Document statuses: `SENT` → `DELIVERED` → `OPENED` → `ACKNOWLEDGED`, or `REJECTED`.

The detailed ER diagram is prepared at the implementation stage.

## 6. Security architecture

| Concern | Measure | Component |
|---|---|---|
| Authenticity, non-repudiation | User signs with their own EDS via NCALayer; server-side verification with the official KalkanCrypt SDK, OCSP/CRL | C1, C4 |
| Confidentiality in transit | HTTPS (TLS 1.2+) for all external traffic; internal services not exposed | C2 |
| Confidentiality at rest | AES-256-GCM encryption of files by the application; keys outside the database | C3.7 |
| Integrity | SHA-256 hash of each file; CMS signature; hash-chained audit log | C3.6, C3.7 |
| Access control | RBAC + organization isolation checked on every request | C3.2 |
| Accountability | Append-only audit log, written in the same transaction as the action | C3.6 |
| Secrets | Configuration through environment variables; `.env` is not committed | All |

The threat model (STRIDE), trust boundaries and the mapping of threats to requirements are in [security/threat-model.md](security/threat-model.md); the security requirements are NFR-01 – NFR-06 and NFR-12 – NFR-26 in [06-requirements.md](06-requirements.md#3-non-functional-requirements) ([DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8)). The System Context Diagram is [`diagrams/export/context.png`](../diagrams/export/context.png) ([source](../diagrams/src/context.puml)).

## 7. Deployment

Docker Compose (development and demonstration):

| Container | Image / build | Ports |
|---|---|---|
| `proxy` | Nginx | 443 (public) |
| `web` | Web client build | internal |
| `api` | `services/api` (FastAPI) | internal |
| `signature` | `services/signature` (Java, KalkanCrypt SDK) | internal |
| `db` | PostgreSQL | internal |
| `storage` | SeaweedFS (S3 gateway) | internal |

NCALayer is installed on the user's workstation, not in Docker. The KalkanCrypt SDK files are obtained from NCA RK and are not committed to the repository.

## 8. Traceability: components → drivers

Drivers D1–D6 are defined in [05-technology-comparison.md](05-technology-comparison.md#project-requirements-that-drive-the-decisions). Requirement IDs refer to [06-requirements.md](06-requirements.md).

| Component | Drivers | Requirements |
|---|---|---|
| C1 Web client | D4, D5 | FR-04 – FR-11, FR-13, FR-14, FR-16, NFR-08, NFR-09, NFR-14 |
| C2 Reverse proxy | D2, D6 | NFR-01, NFR-13 (body size), NFR-14, NFR-15, NFR-17 |
| C3.1 Auth | D1, D2 | FR-01, FR-19, NFR-03, NFR-04, NFR-12, NFR-15, NFR-22, NFR-23 |
| C3.2 Organizations & users | D2, D4 | FR-02, FR-03, FR-20, NFR-06 |
| C3.3 / C3.4 Documents, exchange | D4 | FR-04 – FR-06, FR-10, FR-11, FR-13, FR-14, FR-18, NFR-06, NFR-13 |
| C3.5 FHIR | D3 | FR-12 |
| C3.6 Audit | D2, D4 | FR-15, FR-16, NFR-05, NFR-25, NFR-26 |
| C3.7 Storage | D2 | FR-17, NFR-02, NFR-18 |
| C3.8 / C4 Signature | D1 | FR-07, FR-08, NFR-16, NFR-24 |
| C5 Database | D4 | NFR-05, NFR-17, NFR-21 |
| C6 File storage | D2, D6 | NFR-02, NFR-17, NFR-21 |

## 9. Open questions

- Final authentication method (EDS challenge–response or login/password + EDS for signing) — depends on interviews (DIP-5).
- Whether recipients must sign an acknowledgement (scenario 4.2, step 5).
- Web client framework — decided by the frontend developer.
- Key management for file encryption beyond the prototype (KMS/HSM) — out of scope for the prototype, described as a limitation.
