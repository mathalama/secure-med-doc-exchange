# Threat Model (STRIDE)

YouTrack: [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) · Owner: Korazbay Alikhan

| Field | Value |
|---|---|
| Version | 0.1 (pre-defense 1) |
| Status | Draft for team review |
| Method | STRIDE per element and trust boundary |
| Related | [Architecture](../07-architecture.md), [Requirements](../06-requirements.md), [Verification plan](../09-verification-plan.md), [System Context Diagram](../../diagrams/export/context.png), [Component diagram](../../diagrams/export/component.png), [ADR 0002](../adr/0002-data-storage.md), [ADR 0005](../adr/0005-eds-signing-and-verification.md) |

The threat model covers the design in [07-architecture.md](../07-architecture.md):

- web client (C1) with NCALayer,
- reverse proxy (C2),
- main API (C3),
- signature service (C4),
- PostgreSQL (C5),
- SeaweedFS (C6),
- NCA RK OCSP/CRL services,
- the future MIS API.

Each threat has at least one mitigation, and each mitigation is mapped to a requirement in [06-requirements.md](../06-requirements.md) (FR-xx, NFR-xx). The security test that checks it is defined in [09-verification-plan.md](../09-verification-plan.md) (ST-xx).

## 1. Scope and assumptions

- **In scope:** the application, its containers and the data flows between them; user workstations only as far as the web client and NCALayer are concerned.
- **Out of scope:**
  - security of NCA RK itself and of NCALayer as a product;
  - physical security and the host operating system;
  - KMS/HSM key management (a prototype limitation, [07-architecture.md §9](../07-architecture.md#9-open-questions));
  - patient-facing access (there is no patient portal).
- **Assumptions:**
  - A1. Users' EDS keys are issued by NCA RK and protected by a password on the user's device.
  - A2. The server host and the Docker engine are administered by a trusted operator.
  - A3. Only synthetic test data is used in the project (NFR-10).
  - A4. The platform is operated centrally for all connected organizations (open question 1 in [06-requirements.md §7](../06-requirements.md#7-open-questions)). If it is deployed per organization, boundary TB7 becomes a network boundary.

## 2. Assets

| ID | Asset | Where | Security property | Impact if compromised |
|---|---|---|---|---|
| AS-01 | Medical document files | C6 (encrypted), C3 memory during processing | Confidentiality, integrity | Disclosure of medical data (special category of personal data); wrong treatment decisions |
| AS-02 | Document metadata: patient IIN and name, type, sender and recipient, status | C5 | Confidentiality, integrity | Disclosure of the fact of treatment; misrouting |
| AS-03 | CMS signatures and verification results | C6, C5 | Integrity, availability | Loss of legal validity and non-repudiation |
| AS-04 | User credentials: password hashes, TOTP secrets | C5 | Confidentiality | Account takeover |
| AS-05 | JWT signing key and active tokens | C3 configuration, browser | Confidentiality, integrity | Impersonation of any user |
| AS-06 | File encryption master key (KEK) | C3 configuration (environment / Docker secret) | Confidentiality | Decryption of all stored files |
| AS-07 | Audit log | C5 | Integrity, availability | Undetectable misuse, impossible investigation |
| AS-08 | Directory of organizations, users and roles | C5 | Integrity | Privilege escalation, documents sent to a fake organization |
| AS-09 | User's EDS private key | User workstation (NCALayer), never on the server | Confidentiality | Forged signatures in the user's name (outside the system as well) |
| AS-10 | Availability of the exchange service | All components | Availability | Delayed patient care, return to e-mail and paper |

## 3. Trust boundaries

The external actors and systems are shown in the [System Context Diagram](../../diagrams/export/context.png) ([source](../../diagrams/src/context.puml)); internal components are shown in the [Component diagram](../../diagrams/export/component.png).

| ID | Boundary | Crosses between | Data crossing |
|---|---|---|---|
| TB1 | Internet ↔ server | Browser (C1), future MIS → reverse proxy (C2) | Credentials, tokens, documents, signatures, metadata |
| TB2 | Browser ↔ NCALayer | Web client page → local WebSocket `wss://127.0.0.1:13579` | Data to sign, CMS signatures, certificate data |
| TB3 | Reverse proxy ↔ internal Docker network | C2 → C3 | Decrypted HTTP requests |
| TB4 | Main API ↔ signature service | C3.8 → C4 | Files, CMS signatures, verification results |
| TB5 | Main API ↔ data stores | C3 → C5, C6 | SQL, encrypted files |
| TB6 | Signature service ↔ NCA RK | C4 → NCA RK OCSP / CRL (Internet) | Certificate serials, status responses |
| TB7 | Organization A ↔ organization B (logical) | Users of different organizations inside one deployment | Documents addressed to another organization |
| TB8 | Platform roles ↔ organization roles (logical) | R-SYS and server operators ↔ document data | Administrative access |

## 4. Threats and mitigations

STRIDE: **S** spoofing, **T** tampering, **R** repudiation, **I** information disclosure, **D** denial of service, **E** elevation of privilege.
Risk = likelihood × impact (L / M / H) before mitigation.

### 4.1 Spoofing

| ID | Element / boundary | Threat | Risk | Mitigation | Requirements | Test |
|---|---|---|---|---|---|---|
| T-01 | C3.1, TB1 | Password guessing, credential stuffing against the login endpoint | H | Second factor (TOTP); Argon2id hashes; lockout after 5 failures; rate limiting per IP | FR-01, NFR-03, NFR-04, NFR-15 | ST-03, ST-15 |
| T-02 | C1, C3.1 | Theft or reuse of an access token (XSS, token in logs or URL) | H | Short-lived signed JWT, refresh token in `HttpOnly` `Secure` `SameSite=Strict` cookie; CSP; no tokens in URLs or logs | NFR-12, NFR-14, NFR-19 | ST-04, ST-09, ST-14 |
| T-03 | C3, C4 | A user submits a document signed with another person's EDS (e.g. a colleague's key) | M | Signer IIN in the certificate must equal the IIN of the authenticated user; signer BIN must equal the sender organization BIN | FR-08, NFR-16 | ST-05 |
| T-04 | C3.2, TB7 | A fake organization or a user impersonating an organization receives documents | M | Only R-SYS registers organizations with BIN; organization BIN is checked against the signer certificate of its users | FR-20, NFR-16 | ST-01, ST-05 |
| T-05 | TB1 | Phishing site imitating MedExchange collects passwords and TOTP codes | M | HSTS and a single canonical domain; TOTP codes valid once and only for 30 s; EDS sign-in option | NFR-01, NFR-14, NFR-23, FR-19 | ST-03, ST-10 |
| T-06 | TB2, C3.1 | Replay of a captured EDS authentication signature or TOTP code | M | Server-generated single-use challenge with ≤ 60 s lifetime; TOTP code rejected after first use | NFR-23, FR-19 | ST-03 |
| T-07 | TB4, TB6 | Forged verification result from a fake signature service or forged OCSP response | L | Signature service reachable only in the internal network; OCSP responses are signed by NCA RK and verified by KalkanCrypt | NFR-17, FR-08 | ST-05, ST-11 |

### 4.2 Tampering

| ID | Element / boundary | Threat | Risk | Mitigation | Requirements | Test |
|---|---|---|---|---|---|---|
| T-08 | C6, TB5 | A stored file is modified or replaced in file storage | M | AES-256-GCM (authenticated encryption); SHA-256 hash compared on every download; CMS signature | NFR-02, FR-17, FR-08 | ST-06 |
| T-09 | C5 | Metadata (patient, recipient, type) is changed in the database after sending | M | Hash of the file and of the canonical metadata recorded in the hash-chained audit record of the "send" event; integrity check reports mismatches | NFR-05, NFR-25 | ST-07 |
| T-10 | TB1 | Man-in-the-middle modifies traffic between browser and server | M | TLS 1.2+ only, HSTS, HTTP → HTTPS redirect | NFR-01, NFR-14 | ST-10 |
| T-11 | C1, TB2 | Injected script or a modified client makes the user sign or send a different file than shown | M | Strict CSP without inline scripts; output encoding; server verifies that the CMS signature covers exactly the uploaded file | NFR-14, NFR-16, FR-08 | ST-05, ST-09 |
| T-12 | C5, C3.6 | Audit records are changed or deleted to hide misuse | H | Append-only table (UPDATE/DELETE revoked for the application role); hash chain; daily export of the head hash outside the database | NFR-05, NFR-26, FR-16 | ST-07 |
| T-13 | C3 API | Parameter tampering / mass assignment (setting role, status, organization or recipient through extra fields) | M | Strict request schemas with explicit allowed fields; server derives organization and status from the session and state | NFR-13, NFR-06 | ST-01, ST-08 |

### 4.3 Repudiation

| ID | Element / boundary | Threat | Risk | Mitigation | Requirements | Test |
|---|---|---|---|---|---|---|
| T-14 | C3, C4 | The sender denies having sent a document or its content | M | CMS signature with the sender's own NCA RK EDS; stored verification result and time; audit record | FR-07, FR-08, FR-15 | ST-05, ST-07 |
| T-15 | C3 | The recipient denies having received or opened a document | M | Statuses "delivered" and "opened" with time and user; signed acknowledgement; audit record of every open and download | FR-10, FR-11, FR-15 | ST-07 |
| T-16 | C3.2 | An administrator denies changing roles or deactivating a user | L | User and role changes are audit events with actor and time; tamper-evident log | FR-15, NFR-05 | ST-07 |

### 4.4 Information disclosure

| ID | Element / boundary | Threat | Risk | Mitigation | Requirements | Test |
|---|---|---|---|---|---|---|
| T-17 | C3, TB7 | A user of organization A reads a document or metadata of organization B (IDOR, guessed IDs, search) | H | Authorization on every request by role and organization; non-sequential IDs; "not found" for documents of other organizations | NFR-06, FR-02, FR-13 | ST-01, ST-02 |
| T-18 | C3 | A recipient opens a document after the access period ended or after revocation | M | Expiry and revocation checked on the server for every file request | FR-14 | ST-17 |
| T-19 | C6, C5, backups | Storage objects or backups are copied by an attacker | M | Files encrypted by the application; master key stored separately from data and backups; backups encrypted | NFR-02, NFR-18, NFR-21 | ST-06, ST-16 |
| T-20 | C3 logs, responses | Sensitive data in application logs or error messages (stack traces, IINs, tokens) | M | Log hygiene rules; generic error responses without internal details | NFR-19, NFR-22 | ST-12, ST-14 |
| T-21 | TB3–TB5 | Direct access to the database, file storage or signature service from outside | H | Only the reverse proxy publishes ports (443, 80 → redirect); other containers in an internal network | NFR-17 | ST-11 |
| T-22 | TB8 | System administrator or server operator reads medical documents | M | R-SYS has no document permissions; files encrypted with a key outside the database; administrative actions audited. Residual: an operator with root access and the key can decrypt (accepted for the prototype; KMS/HSM is future work) | FR-02, NFR-02, NFR-18, FR-15 | ST-01 |
| T-23 | Repository, images | Secrets (keys, passwords) leak through the Git repository or Docker images | M | No secrets in the repository; secrets scanning in CI; secrets passed through environment / Docker secrets | NFR-03, NFR-18, NFR-20 | ST-13 |
| T-24 | C3.1, C3 | Account or patient enumeration through login errors or search responses | L | Identical error for wrong login and wrong password; search limited to own organization | NFR-22, NFR-06 | ST-02, ST-03 |
| T-25 | C3.4 | A document is sent to another organization without the patient's consent | M | Sending blocked until consent is confirmed; confirmation stored and audited | FR-05, FR-15 | ST-01 |

### 4.5 Denial of service

| ID | Element / boundary | Threat | Risk | Mitigation | Requirements | Test |
|---|---|---|---|---|---|---|
| T-26 | C2, C3, C6 | Very large or many uploads exhaust memory or storage | M | Request size limit at the proxy and in the API (25 MB file); rate limits per user | NFR-13, NFR-15 | ST-08, ST-15 |
| T-27 | C3.1 | Login flood; deliberate lockout of other users' accounts | M | Rate limit per IP; lockout is temporary (15 min) and logged; administrator can unlock | NFR-04, NFR-15, FR-03 | ST-15 |
| T-28 | TB6 | NCA RK OCSP/CRL unavailable, so signatures cannot be checked | M | Fail closed: a document is never delivered unverified; verification retried; last valid CRL used within its validity period | NFR-24, FR-08 | ST-05 |
| T-29 | C5, C6 | Data loss through storage failure or ransomware | M | Encrypted daily backups of the database and file storage; tested restore | NFR-21 | ST-16 |

### 4.6 Elevation of privilege

| ID | Element / boundary | Threat | Risk | Mitigation | Requirements | Test |
|---|---|---|---|---|---|---|
| T-30 | C3.2 | An organization administrator assigns a platform role or changes users of another organization | H | Administrators manage only their own organization and only organization roles; checked on the server | FR-03, NFR-06, NFR-13 | ST-01 |
| T-31 | C3, C4 | Injection (SQL, path, deserialization) or a vulnerable dependency gives code execution | M | ORM with parameterized queries; strict input schemas; SAST and dependency scanning in CI | NFR-13, NFR-20 | ST-08, ST-12, ST-13 |
| T-32 | C3.1 | Forged JWT (`alg: none`, weak or leaked signing key) | M | Fixed algorithm, key of ≥ 256 bits from secret configuration, `exp` and `iss` checked | NFR-12, NFR-18 | ST-04 |
| T-33 | Containers | A compromised container gains host access | L | Containers run as non-root with minimal images; images scanned | NFR-20 | ST-13 |
| T-34 | Future MIS API (FR-18) | A leaked API token of an MIS is used to read or send documents of that organization | M | Tokens scoped to one organization, revocable and audited; same authorization as users. Applies when FR-18 is implemented | FR-18, NFR-06, NFR-12, FR-15 | ST-01, ST-04 |

## 5. Coverage check

| STRIDE | Threats | All threats have a mitigation mapped to a requirement |
|---|---|---|
| Spoofing | T-01 – T-07 | Yes |
| Tampering | T-08 – T-13 | Yes |
| Repudiation | T-14 – T-16 | Yes |
| Information disclosure | T-17 – T-25 | Yes |
| Denial of service | T-26 – T-29 | Yes |
| Elevation of privilege | T-30 – T-34 | Yes |

Security requirements introduced by this threat model: NFR-12 – NFR-26 ([06-requirements.md §3](../06-requirements.md#3-non-functional-requirements)).

## 6. Residual risks (accepted for the prototype)

| Risk | Reason it is accepted | Future measure |
|---|---|---|
| A server operator with root access and the master key can decrypt files (T-22) | No KMS/HSM in the prototype | KMS/HSM, separation of duties, end-to-end encryption (open question 2) |
| A user shares their EDS key and password with a colleague (T-03) | Organizational issue; the system can only bind the signature to the logged-in user | Policy and training; EDS sign-in (FR-19) |
| A compromised user workstation (malware controlling the browser and NCALayer) | Outside the system boundary | Endpoint protection in medical organizations |
| Large-scale DDoS on the public endpoint | No DDoS protection in the demo deployment | Upstream DDoS protection in production |

## 7. Review

The threat model is reviewed when the architecture changes, when a new external interface is added (FR-18, FR-19), and before the final pre-defense. Each review records new threats in §4 and new tests in the verification plan.
