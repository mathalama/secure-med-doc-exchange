# Requirements Specification

YouTrack: [DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5)

| Field | Value |
|---|---|
| Version | 0.2 (pre-defense 1) |
| Status | Draft for supervisor review |
| Owner | Almasuly Damir |
| Related | [Proposal](01-proposal.md), [Architecture](07-architecture.md), [ADRs](adr/), [Interviews](02-evidence/interviews.md), [Use case diagram](../diagrams/export/use-case.png), [Prototype](08-prototype.md) |

> **Evidence status.** Every requirement has a *Source* and a *Status*.
> **Assumption**: derived from the proposal, the ADRs or domain analysis, not yet confirmed by users.
> **Verified**: confirmed by findings in [02-evidence/interviews.md](02-evidence/interviews.md) (finding ID in *Source*).
> No interviews are recorded yet, so every item is *Assumption*. Update both columns as interviews are added.
> Security requirements (threat model, controls) are specified separately in [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8); the non-functional requirements below list only what the functional scope depends on.

## 1. Actors and roles

Actor names match [07-architecture.md §2](07-architecture.md#2-actors-and-external-systems), as required for the Use Case, System Context and Component diagrams.

| ID | Actor | Description |
|---|---|---|
| R-DOC | Doctor | Prepares, signs and sends documents about their patients; opens and accepts received documents |
| R-REG | Registrar / medical records staff | Routes incoming documents, tracks statuses, prepares and sends documents with their own EDS |
| R-ADM | Organization administrator | Manages users and roles of one medical organization |
| R-SEC | Security officer | Reviews and exports the audit log of the organization; read-only |
| R-SYS | System administrator | Registers organizations and their first administrator; manages the platform |
| X-NCL | NCALayer | NCA RK application on the user's workstation; signs data with the user's key ([ADR 0005](adr/0005-eds-signing-and-verification.md)) |
| X-NCA | NCA RK services | OCSP / CRL certificate status services used by the signature service |
| X-MIS | External medical information system | Information system of a medical organization; integrates through the REST API (future) |

Out of scope ([proposal §8](01-proposal.md#8-project-scope)): issuing EDS keys, a patient portal, a full FHIR server, integration with real MIS and eGov services, mobile apps, real patient data.

## 2. Functional requirements

Priority: MoSCoW for the final system. **M**ust, **S**hould, **C**ould.

| ID | Requirement | Priority | Source | Status |
|---|---|---|---|---|
| FR-01 | The system shall authenticate users by login and password followed by a one-time code from an authenticator app (TOTP). | M | Proposal §3 | Assumption |
| FR-02 | The system shall restrict every action by role (R-DOC, R-REG, R-ADM, R-SEC) according to the role matrix in section 5. | M | Proposal §7 | Assumption |
| FR-03 | An organization administrator shall invite, deactivate and change roles of users of their own organization only. | M | Proposal §4 | Assumption |
| FR-04 | A user shall upload a document file (PDF, JPEG or PNG, up to 25 MB) with metadata: patient IIN, patient name, document type, title and an optional comment. | M | Proposal §8 | Assumption |
| FR-05 | Before sending, the sender shall confirm that the patient's consent to transfer the document has been obtained; the system shall not send a document without this confirmation. | M | Law No. 94-V; A-05 | Assumption |
| FR-06 | The sender shall choose one recipient organization from the directory of connected organizations, optionally with a recipient department. | M | Proposal §7 | Assumption |
| FR-07 | The sender shall sign the document with their own EDS through NCALayer (CMS signature); the private key shall never leave the user's device. | M | ADR 0005; Law No. 370-II | Assumption |
| FR-08 | Before delivery, the system shall verify the signature, the certificate chain to NCA RK and the certificate status (OCSP/CRL); a document with an invalid signature or certificate shall not be delivered, and the sender shall see the reason. | M | ADR 0005 | Assumption |
| FR-09 | A recipient shall open a received document and see the verification result: signer's name, IIN, organization and BIN, certificate validity, signing time and whether the file is unchanged. | M | Proposal §7; A-03 | Assumption |
| FR-10 | A recipient doctor shall accept a received document by signing an acknowledgement with their EDS, or reject it with a mandatory reason. | M | Proposal §8 | Assumption |
| FR-11 | The system shall track the status of every sent document (sent, delivered, opened, signed, rejected, revoked, expired), show it to the sender and the recipient, and notify users in the web client when a status changes or a new document arrives. | M | Proposal §8; A-02 | Assumption |
| FR-12 | The system shall describe every document with an HL7 FHIR R4 `DocumentReference` resource and let authorized users download it together with the original file and the CMS signature. | M | ADR 0004 | Assumption |
| FR-13 | Users shall search and filter incoming and sent documents by patient IIN or name, document type, organization, period and status. | S | Proposal §4; A-08 | Assumption |
| FR-14 | The sender shall set an access period for a sent document (default 30 days) and revoke access before it ends; after expiry or revocation the recipient can no longer open the file. | S | Proposal §3 | Assumption |
| FR-15 | The system shall record every action with documents and accounts (login, failed login, upload, sign, send, verification result, open, download, accept, reject, revoke, user change) in an audit log with actor, action, object, time and IP address. | M | Proposal §7; A-06 | Assumption |
| FR-16 | A security officer shall filter the audit log by user, action, document and period, export it as CSV, and run an integrity check of the log. | S | Proposal §4 | Assumption |
| FR-17 | The system shall verify the SHA-256 hash of a stored file on every download and block the download if it does not match. | S | ADR 0002 | Assumption |
| FR-18 | The system shall provide a token-based REST API (OpenAPI) for sending and receiving documents from an external MIS. | C | ADR 0003 | Assumption |
| FR-19 | A user shall be able to sign in with an EDS authentication certificate through NCALayer instead of a password and TOTP. | C | Domain practice in the RK; [07-architecture.md §4.3](07-architecture.md#43-authentication) | Assumption |
| FR-20 | A system administrator shall register a medical organization (name, BIN) with its first organization administrator, and suspend an organization. | S | [07-architecture.md §2](07-architecture.md#2-actors-and-external-systems) | Assumption |

## 3. Non-functional requirements

| ID | Category | Requirement | Priority | Source | Status |
|---|---|---|---|---|---|
| NFR-01 | Security | All traffic uses TLS 1.2 or higher; HTTP redirects to HTTPS. | M | ADR 0003 | Assumption |
| NFR-02 | Security | Files are encrypted with AES-256-GCM before they are written to storage; keys are kept outside the database. | M | ADR 0002 | Assumption |
| NFR-03 | Security | Passwords are stored only as salted hashes (Argon2id); no secrets in the repository. | M | — | Assumption |
| NFR-04 | Security | Five consecutive failed logins lock the account for 15 minutes; an idle session ends after 15 minutes. | M | — | Assumption |
| NFR-05 | Security | The audit log is append-only (UPDATE and DELETE revoked) and each record contains the hash of the previous one. | M | ADR 0002 | Assumption |
| NFR-06 | Security | Authorization is checked on the server for every request; a user never gets a document of another organization unless it was sent to that organization. | M | ADR 0003 | Assumption |
| NFR-07 | Performance | List and metadata requests respond within 500 ms (p95) with 50 concurrent users; signature verification completes within 3 s. | S | — | Assumption |
| NFR-08 | Usability | A doctor using the system for the first time sends a signed document in under 2 minutes; the SUS score of the usability test is at least 68. | S | — | Assumption |
| NFR-09 | Usability | The web client works in the current versions of Chrome, Edge and Firefox on a 1366×768 screen and is available in Russian (Kazakh in the final version). | S | — | Assumption |
| NFR-10 | Compliance | Processing of personal and medical data follows Law No. 94-V and Art. 60 of Code No. 360-VI; only test data is used in the project. | M | Proposal §3 | Assumption |
| NFR-11 | Deployability | The whole system starts with `docker compose up` and a documented `.env`. | S | Proposal §7 | Assumption |

## 4. Acceptance criteria

Given–When–Then criteria for the *Must* requirements; each is a test case for the verification plan ([DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8)).

| Req. | Acceptance criterion |
|---|---|
| FR-01 | **Given** a user with TOTP enabled, **when** they enter the correct password and a wrong code, **then** access is denied and a failed-login event is logged; with both correct, a session is created. |
| FR-02 | **Given** a user with role R-REG, **when** they call an administrator endpoint, **then** the response is 403. Each role can perform exactly the actions of the role matrix. |
| FR-03 | **Given** an administrator of organization A, **when** they invite a user, **then** the user belongs to A; any attempt to change a user of organization B returns 403. |
| FR-04 | **Given** a 20 MB PDF with valid metadata, **when** it is uploaded, **then** it appears as a draft. A 30 MB file, an `.exe` file or an IIN that is not 12 digits is rejected with a clear message. |
| FR-05 | **Given** a draft, **when** the sender tries to sign and send it without confirming consent, **then** the action is unavailable in the UI and the API returns 422. |
| FR-06 | **Given** a signed draft, **when** the sender chooses organization B and sends it, **then** the document appears in B's incoming list within 5 s. |
| FR-07 | **Given** NCALayer is running with a test key, **when** the sender signs, **then** the server receives a CMS signature and no private key material. If NCALayer is not running, the user sees how to start it. |
| FR-08 | **Given** a document signed with a revoked or expired test certificate, **when** it is sent, **then** it is not delivered, the sender sees "Certificate revoked" or "Certificate expired", and the result is logged. |
| FR-09 | **Given** a delivered document, **when** a recipient opens it, **then** the signer's name, IIN, organization, BIN, certificate validity, signing time and file integrity are displayed, and the status becomes "opened" for the sender. |
| FR-10 | **Given** an opened document, **when** the recipient signs the acknowledgement, **then** the status becomes "signed"; **when** they reject it without a reason, **then** rejection is not possible. |
| FR-11 | **Given** a document sent to B, **when** its status changes, **then** the sender sees the new status and a notification within 5 s, and the status history shows each change with time and user. |
| FR-12 | **Given** a delivered document, **when** its `DocumentReference` is downloaded, **then** it validates against FHIR R4 (`fhir.resources`) and contains the patient, author, custodian, type, date and the file hash. |
| FR-15 | **Given** any event listed in FR-15, **when** it occurs, **then** a record with actor, action, object, time and IP exists in the audit log within 1 s. |

## 5. Role matrix

| Action | R-DOC | R-REG | R-ADM | R-SEC |
|---|:-:|:-:|:-:|:-:|
| Sign in | ✔ | ✔ | ✔ | ✔ |
| Upload a document and confirm consent | ✔ | ✔ | – | – |
| Sign with EDS and send | ✔ | ✔ | – | – |
| View incoming and sent documents of own organization | ✔ | ✔ | – | – |
| Open a received document and see verification result | ✔ | ✔ | – | – |
| Accept (sign acknowledgement) or reject | ✔ | – | – | – |
| Revoke access to a sent document | ✔ | ✔ | – | – |
| Manage users and roles of own organization | – | – | ✔ | – |
| View the audit log of own organization | – | – | ✔ | ✔ |
| Export the audit log and run the integrity check | – | – | – | ✔ |

The system administrator (R-SYS) works at platform level: registers and suspends organizations (FR-20) and has no access to documents.

## 6. Use cases and traceability

Diagram: [`diagrams/src/use-case.puml`](../diagrams/src/use-case.puml) → [`diagrams/export/use-case.png`](../diagrams/export/use-case.png).

| Use case | Actors | Requirements | Prototype screen |
|---|---|---|---|
| UC-01 Sign in | All users | FR-01, FR-19, NFR-04 | S1 Sign in |
| UC-02 Manage users and roles | R-ADM | FR-02, FR-03 | S7 Users and roles |
| UC-03 Send a document | R-DOC, R-REG, X-NCL | FR-04, FR-05, FR-06, FR-07, FR-14 | S4 New document |
| UC-04 Verify signature (included in UC-03 and UC-06) | X-NCA | FR-08, FR-09 | S4, S3 |
| UC-05 View incoming documents | R-DOC, R-REG | FR-11, FR-13 | S2 Incoming |
| UC-06 Open and verify a document | R-DOC, R-REG | FR-09, FR-12, FR-17 | S3 Document |
| UC-07 Accept or reject a document | R-DOC, X-NCL | FR-10 | S3 Document |
| UC-08 Track and revoke sent documents | R-DOC, R-REG | FR-11, FR-13, FR-14 | S5 Sent |
| UC-09 Review the audit log | R-ADM, R-SEC | FR-15, FR-16, NFR-05 | S6 Audit log |
| UC-10 Exchange through the API | X-MIS | FR-18 | — (future) |
| UC-11 Register organizations | R-SYS | FR-20 | — (not in the prototype) |

## 7. Open questions

To be resolved with the supervisor and through interviews.

1. Is the platform operated centrally for all organizations, or deployed in each organization?
2. Is server-side encryption enough, or do organizations require end-to-end encryption?
3. Is the patient's IIN acceptable as the patient identifier in the exchange?
4. Which sign-in method is primary: password + TOTP (FR-01) or EDS challenge–response through NCALayer, proposed in [07-architecture.md §4.3](07-architecture.md#43-authentication) (FR-19)? The priority of FR-01 and FR-19 will be swapped if interviews confirm that all users have EDS keys.
5. Must the recipient's acknowledgement (FR-10) be signed with EDS, or is a confirmation click enough?
6. Which retention periods apply to documents in transit and to the audit log?
