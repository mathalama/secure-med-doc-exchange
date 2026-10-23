# Requirements Specification

YouTrack: [DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5)

| Field | Value |
|---|---|
| Version | 0.2 (pre-defense 1) |
| Status | Draft for supervisor review |
| Owner | Almasuly Damir |
| Related | [Proposal](01-proposal.md), [Architecture](07-architecture.md), [ADRs](adr/), [Interviews](02-evidence/interviews.md), [Use case diagram](../diagrams/export/use-case.png), [Prototype](08-prototype.md), [Threat model](security/threat-model.md), [Verification plan](09-verification-plan.md) |
| Security and non-functional requirements | Korazbay Alikhan ([DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8)) |

> **Evidence status.** Every requirement has a *Source* and a *Status*.
> **Assumption**: derived from the proposal, the ADRs or domain analysis, not yet confirmed by users.
> **Verified**: confirmed by findings in [02-evidence/interviews.md](02-evidence/interviews.md) (finding ID in *Source*).
> No interviews are recorded yet, so every item is *Assumption*. Update both columns as interviews are added.
> Security requirements NFR-12 – NFR-26 are derived from the STRIDE threat model ([security/threat-model.md](security/threat-model.md), threats T-xx in *Source*) in [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8). Acceptance criteria of all non-functional requirements are in [§4.2](#42-non-functional-requirements); the tests are described in [09-verification-plan.md](09-verification-plan.md).

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
| NFR-12 | Security | Access tokens are JWTs signed with one fixed algorithm and a key of at least 256 bits from secret configuration, valid for at most 15 minutes. The refresh token is single-use, kept in an `HttpOnly`, `Secure`, `SameSite=Strict` cookie and revoked on logout, password change and user deactivation. | M | T-02, T-32, T-34 | Assumption |
| NFR-13 | Security | Every API request is validated against a strict schema (unknown fields rejected). The file type is determined by content (PDF, JPEG, PNG). The proxy rejects request bodies over 26 MB and the API rejects files over 25 MB. | M | T-13, T-26, T-31 | Assumption |
| NFR-14 | Security | Responses set HSTS (max-age ≥ 1 year), a Content Security Policy without `unsafe-inline` that allows connections only to the own origin and `wss://127.0.0.1:13579`, `X-Content-Type-Options: nosniff`, `frame-ancestors 'none'` and `Referrer-Policy: no-referrer`. User-provided text is output-encoded, and state-changing requests are protected against CSRF. | M | T-02, T-05, T-10, T-11 | Assumption |
| NFR-15 | Security | Login and TOTP endpoints accept at most 10 requests per minute per IP address and per account; other endpoints at most 120 requests per minute per user; excess requests get HTTP 429. | S | T-01, T-26, T-27 | Assumption |
| NFR-16 | Security | A signature is accepted only if the signer IIN equals the IIN of the authenticated user, the signer BIN equals the BIN of the sender organization, the CMS signature covers exactly the uploaded file, and the same signature has not been used for another document. | M | T-03, T-04, T-11; ADR 0005 | Assumption |
| NFR-17 | Security | Only the reverse proxy publishes ports (443; 80 only redirects). The API, signature service, database and file storage are reachable only inside the internal Docker network. The signature service has no access to the database or file storage. The application database role has no rights beyond its tables. | M | T-07, T-21; ADR 0005 | Assumption |
| NFR-18 | Security | Each file is encrypted with its own random 256-bit data key and a unique 96-bit nonce. The data key is stored wrapped by a master key. The master key and the JWT signing key come from secret configuration and never appear in the database, backups, logs or the repository. A key rotation procedure is documented. | M | T-19, T-22, T-23, T-32; ADR 0002 | Assumption |
| NFR-19 | Security | Application logs never contain passwords, TOTP codes or secrets, tokens, document content or full IINs (IINs are masked). | M | T-02, T-20 | Assumption |
| NFR-20 | Security | Dependencies and base images are pinned. CI runs static analysis, dependency, secret and container image scans on every pull request. A milestone is released with no open High or Critical finding, unless an exception is documented. Containers run as a non-root user. | M | T-23, T-31, T-33 | Assumption |
| NFR-21 | Availability | The database and file storage are backed up daily in encrypted form; a restore is tested before each pre-defense; RPO ≤ 24 h, RTO ≤ 4 h. | S | T-19, T-29 | Assumption |
| NFR-22 | Security | Error responses never contain stack traces, SQL, internal paths or component versions. A failed login returns the same message whether or not the login exists. | M | T-20, T-24 | Assumption |
| NFR-23 | Security | A TOTP code is accepted only once and only within its 30-second step (± 1 step). An EDS authentication challenge (FR-19) is random (≥ 128 bits), single-use and valid for at most 60 seconds. | M | T-05, T-06 | Assumption |
| NFR-24 | Security | If the certificate status cannot be determined (OCSP unavailable and no valid CRL), the document is not delivered. Its status is "verification pending", verification is retried at least 3 times within 15 minutes, and then the sender is notified. | M | T-28; ADR 0005 | Assumption |
| NFR-25 | Security | The audit record of sending a document contains the SHA-256 of the file and of the canonical metadata; the integrity check (FR-16) reports every document whose current metadata no longer matches. | S | T-09 | Assumption |
| NFR-26 | Security | The hash of the latest audit record is exported outside the database at least daily; the integrity check compares the chain with the last exported hash. | S | T-12 | Assumption |

## 4. Acceptance criteria

Each criterion is a test case of the verification plan ([09-verification-plan.md](09-verification-plan.md)).

### 4.1 Functional requirements

Given–When–Then criteria for the *Must* functional requirements.

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

### 4.2 Non-functional requirements

Measurable criteria for every non-functional requirement. Measurement conditions (environment, data, repetitions) are defined in [09-verification-plan.md §5](09-verification-plan.md#5-measurement-conditions).

| Req. | Acceptance criterion | Test |
|---|---|---|
| NFR-01 | testssl.sh against the public endpoint reports only TLS 1.2 and 1.3 enabled and no finding of severity High or Critical; `http://` requests return 301 to `https://`. | ST-10 |
| NFR-02 | For 20 uploaded test files, none of the objects in SeaweedFS starts with the file signature (`%PDF`, JPEG, PNG) or contains the original bytes; decryption with a different key fails; the master key is not found in a full database dump. | ST-06 |
| NFR-03 | All password hashes in the database have the `$argon2id$` format with at least m = 19 MiB, t = 2; gitleaks over the full Git history reports 0 findings. | ST-13 |
| NFR-04 | After 5 wrong passwords the 6th attempt with the correct password is rejected; login succeeds again after 15 min (± 1 min). A session without requests for 15 min receives 401 on the next request. | ST-03 |
| NFR-05 | `UPDATE` and `DELETE` on the audit table under the application role fail with "permission denied"; in 20 out of 20 direct modifications or deletions of random records (as database superuser) the integrity check reports the first broken record. | ST-07 |
| NFR-06 | The authorization matrix test (every endpoint × role × own/other organization) produces 0 unexpected allowed responses; 100% of requests with document IDs of another organization return 404. | ST-01, ST-02 |
| NFR-07 | k6 with 50 virtual users for 10 min on the reference dataset: p95 of list and metadata requests ≤ 500 ms; p95 of signature verification ≤ 3 s over at least 100 verifications. | PT-01 |
| NFR-08 | Usability test with at least 5 participants: median time from opening "New document" to a sent, signed document ≤ 2 min; mean SUS ≥ 68. | UT-01 |
| NFR-09 | The e2e suite passes in the current Chrome, Edge and Firefox at 1366×768; no UI string is missing a Russian translation. | E2E |
| NFR-10 | Compliance checklist (§3 of the verification plan) passes 100%: only synthetic data in the repository and test databases, consent recorded for 100% of sent documents, deployment location documented. | CL-01 |
| NFR-11 | On a clean machine with Docker, `docker compose up` with `.env.example` brings all services to "healthy" within 5 min without manual steps beyond the README. | DT-01 |
| NFR-12 | A token with a modified payload, `alg: none`, another key or an expired `exp` returns 401; a refresh token used twice is rejected and revokes the session; after logout or deactivation old tokens return 401 within 15 min (refresh at once). | ST-04 |
| NFR-13 | Requests with unknown fields return 422; a renamed `.exe` with a `.pdf` extension, a 30 MB file and a 27 MB request body are rejected (422 / 413). | ST-08 |
| NFR-14 | The security headers listed in NFR-14 are present on every HTML and API response; a `<script>` payload in title or comment is shown as text; a cross-site POST without the CSRF token is rejected. | ST-09 |
| NFR-15 | The 11th login request within one minute from one IP returns 429; normal use (UI e2e suite) never receives 429. | ST-15 |
| NFR-16 | In the signature test corpus, 100% of documents signed by another user, by a user of another organization, over a different file, or with a reused signature are rejected. | ST-05 |
| NFR-17 | nmap from outside the Docker host finds only ports 80 and 443 open; from the signature service container, connections to the database and storage fail. | ST-11 |
| NFR-18 | Two uploads of the same file produce different ciphertexts and different wrapped data keys; the master key and JWT key are absent from database dumps, backups, logs and the repository (search for their values returns 0 hits); the rotation procedure is executed once on the test environment and all files remain readable. | ST-06, ST-13 |
| NFR-19 | After a full e2e run, a search of all container logs for the test passwords, TOTP secrets, tokens, full test IINs and document text returns 0 hits. | ST-14 |
| NFR-20 | The CI pipeline contains all four scan types and fails on High/Critical findings; all images run with a non-root UID. | ST-13 |
| NFR-21 | A restore from the latest backup to an empty environment completes within 4 h, and 100% of test documents open with valid hashes. | ST-16 |
| NFR-22 | ZAP and the API error tests find no stack trace, SQL text, internal path or version in any response; responses for a non-existent login and a wrong password are identical. | ST-03, ST-12 |
| NFR-23 | A TOTP code reused within its window is rejected; an EDS challenge signed after 60 s or used twice is rejected. | ST-03 |
| NFR-24 | With OCSP blocked and no valid CRL, 100% of sent test documents stay "verification pending" and are not visible to the recipient; after 3 failed retries the sender is notified; after OCSP is restored they are verified and delivered. | ST-05 |
| NFR-25 | After a direct change of patient IIN or recipient in the database, the integrity check reports the changed document in 100% of 10 trials. | ST-07 |
| NFR-26 | After the audit chain is rewritten consistently from some record onward (all hashes recomputed), the integrity check detects the mismatch with the last exported head hash. | ST-07 |

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
