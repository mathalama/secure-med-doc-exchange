# Verification Plan

YouTrack: [DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8) · Owner: Korazbay Alikhan

| Field | Value |
|---|---|
| Version | 0.1 (pre-defense 1) |
| Status | Draft for team review |
| Related | [Proposal](01-proposal.md), [Requirements](06-requirements.md), [Threat model](security/threat-model.md), [Architecture](07-architecture.md), [Project management](10-project-management.md) |

This plan defines how the team will show that the project aim is achieved:

> To develop a web-based system that allows medical organizations to exchange digital medical documents securely, ensuring confidentiality, integrity, authenticity (EDS of NCA RK) and full traceability of all actions ([proposal §5](01-proposal.md#5-aim)).

It defines the test levels, the outcome metrics, the measurement conditions and the baseline the results are compared with. Test cases come from the acceptance criteria in [06-requirements.md §4](06-requirements.md#4-acceptance-criteria) and from the threats in [security/threat-model.md](security/threat-model.md).

## 1. From aim to measurable outcomes

| Aim property | Question to answer | Metric | Section |
|---|---|---|---|
| Confidentiality | Can anyone read a document that was not sent to their organization? | OM-1, OM-4 | §4 |
| Integrity | Is every modification of a document, its metadata or the audit log detected? | OM-2, OM-3 | §4 |
| Authenticity (NCA RK EDS) | Are only documents with a valid signature of the actual sender delivered? | OM-2 | §4 |
| Traceability | Is every action recorded, and is the record tamper-evident? | OM-3 | §4 |
| Practical value | Is the system usable and fast enough compared with current practice? | OM-5 – OM-8 | §4, §6 |

## 2. Test levels

| Level | What is tested | Tools | Environment | When | Exit criterion |
|---|---|---|---|---|---|
| Unit | Functions and classes of the API (auth, RBAC checks, encryption, hashing, audit chain, FHIR builder) and the signature service | pytest, pytest-cov; JUnit 5 | Developer machine, CI | Every commit / PR | All tests pass; line coverage ≥ 80% for security modules (C3.1, C3.2, C3.6, C3.7, C3.8, C4) and ≥ 70% overall |
| Integration | API with real PostgreSQL, SeaweedFS and the signature service; transactions (status + audit record), encryption round-trip, OCSP handling | pytest with Docker Compose test profile | CI, test environment | Every PR | All tests pass |
| API | Every endpoint of the OpenAPI specification: status codes, validation, authorization matrix, error format | pytest + httpx; Schemathesis (property-based tests from OpenAPI) | Test environment | Every PR | 100% of Must FR acceptance criteria automated and passing; no 5xx from Schemathesis |
| End-to-end | Main scenarios UC-01 – UC-09 through the browser | Playwright; NCALayer replaced by a stub WebSocket server that signs with NCA RK test keys; one manual run with real NCALayer | Test environment, Chrome / Edge / Firefox | Before each milestone | All UC scenarios pass in 3 browsers (NFR-09); manual NCALayer run recorded |
| Security | Tests ST-01 – ST-17 (§3), derived from the threat model | See §3 | Test environment (isolated) | Automated part on every PR; full run in M6 | Every threat T-01 – T-34 covered by a passing test; OM-4 met |
| Performance | NFR-07 | k6 | Reference environment (§5) | M4, M6 | NFR-07 criteria met |
| Usability | NFR-08 | Moderated test, SUS questionnaire | Reference environment, participants from target roles | M6 | NFR-08 criteria met |
| Deployment | NFR-11 | Clean VM | Clean VM | M4, M7 | NFR-11 criterion met (DT-01) |

CI (GitHub Actions) runs on every pull request: unit, integration and API tests, SAST, dependency, secret and image scans (NFR-20). A pull request cannot be merged if the pipeline fails.

## 3. Security tests

Each test covers threats from [threat-model.md §4](security/threat-model.md#4-threats-and-mitigations) and checks the acceptance criteria of the listed requirements.

| ID | Test | Method and tools | Requirements | Threats |
|---|---|---|---|---|
| ST-01 | Authorization matrix: every endpoint × role (R-DOC, R-REG, R-ADM, R-SEC, R-SYS, anonymous) × own / other organization, compared with the role matrix | Generated pytest suite | FR-02, FR-03, FR-05, FR-20, NFR-06, NFR-13 | T-04, T-13, T-17, T-22, T-25, T-30, T-34 |
| ST-02 | Cross-organization access (IDOR): document, file, FHIR and audit endpoints with IDs and search terms of another organization | pytest; manual with OWASP ZAP | NFR-06, FR-13 | T-17, T-24 |
| ST-03 | Authentication: lockout, TOTP wrong / reused / expired, EDS challenge replay, identical error messages, idle timeout | pytest | FR-01, FR-19, NFR-04, NFR-22, NFR-23 | T-01, T-05, T-06, T-24 |
| ST-04 | Tokens: modified payload, `alg: none`, wrong key, expired, refresh reuse, after logout and deactivation | pytest with PyJWT | NFR-12 | T-02, T-32, T-34 |
| ST-05 | Signature corpus (§5.3): valid; file modified after signing; signed by another user; by a user of another organization; revoked; expired; untrusted CA; signature reused for another document; OCSP unavailable | pytest + JUnit with NCA RK test certificates | FR-07, FR-08, NFR-16, NFR-24 | T-03, T-04, T-07, T-11, T-14, T-28 |
| ST-06 | Storage: ciphertext check of objects, tampered object detected on download, different ciphertexts for the same file, master key absent from dumps | Integration test + script | NFR-02, NFR-18, FR-17 | T-08, T-19 |
| ST-07 | Audit log: completeness for every FR-15 event, database privileges, random modifications and deletions, metadata change, consistent chain rewrite vs exported head hash | Integration test + SQL scripts | FR-15, FR-16, NFR-05, NFR-25, NFR-26 | T-09, T-12, T-14, T-15, T-16 |
| ST-08 | Input validation: unknown fields, renamed executables, polyglot files, oversize file and body, malformed IIN | pytest, Schemathesis | NFR-13, FR-04 | T-13, T-26, T-31 |
| ST-09 | Web: stored XSS in title / comment / reject reason, CSRF, security headers, CSP violations | Playwright, ZAP | NFR-14 | T-02, T-11 |
| ST-10 | TLS: protocols, ciphers, certificate, HSTS, HTTP redirect | testssl.sh | NFR-01, NFR-14 | T-05, T-10 |
| ST-11 | Network exposure: open ports from outside; connections from the signature service to the database and storage | nmap; `docker exec` checks | NFR-17 | T-07, T-21 |
| ST-12 | Dynamic scan: unauthenticated baseline and authenticated full scan for OWASP Top 10 issues | OWASP ZAP | NFR-22, NFR-13 | T-20, T-31 |
| ST-13 | Static and supply-chain scans: SAST, dependencies, secrets in history, images, non-root containers | Semgrep, Bandit; pip-audit, OWASP Dependency-Check, npm audit; gitleaks; Trivy | NFR-03, NFR-18, NFR-20 | T-23, T-31, T-33 |
| ST-14 | Log hygiene: search of all logs after a full e2e run for test secrets, tokens, full IINs and document text | Script (grep over `docker compose logs`) | NFR-19 | T-02, T-20 |
| ST-15 | Rate limiting and flooding: login flood, upload flood, lockout abuse | k6, pytest | NFR-04, NFR-15 | T-01, T-26, T-27 |
| ST-16 | Backup and restore into an empty environment; check of hashes after restore | Script, timed | NFR-21 | T-19, T-29 |
| ST-17 | Access expiry and revocation: file, FHIR and download endpoints after expiry or revocation | pytest with time control | FR-14 | T-18 |

Severity of findings follows CVSS v3.1: Critical ≥ 9.0, High 7.0–8.9, Medium 4.0–6.9, Low < 4.0.

**Compliance checklist CL-01 (NFR-10):**

1. Only synthetic data in the repository and test databases. Test IINs come from a generator, and the generated list is stored with the tests.
2. Consent is recorded for 100% of sent documents (FR-05).
3. The purposes of processing personal data are listed.
4. The deployment location (RK) is documented.
5. Retention periods are documented (open question 6 in [06-requirements.md §7](06-requirements.md#7-open-questions)).

## 4. Outcome metrics

| ID | Metric | Formula / unit | Target | Source of data |
|---|---|---|---|---|
| OM-1 | Unauthorized access rate | Unexpected "allowed" responses ÷ all denied-by-design requests in ST-01 and ST-02 | 0% | ST-01, ST-02 |
| OM-2 | Detection rate of invalid documents | Rejected invalid cases ÷ all invalid cases of the signature corpus and tampered files; false rejections of valid cases | 100% detected; 0 false rejections out of ≥ 20 valid signatures | ST-05, ST-06 |
| OM-3 | Audit completeness and tamper detection | Events with an audit record ÷ events generated in the e2e run; detected modifications ÷ injected modifications | 100% and 100% | ST-07 |
| OM-4 | Open security findings | Number of open High / Critical findings from ST-01 – ST-17 at the milestone | 0 (Medium: fixed or justified) | Security test report |
| OM-5 | Time to send a signed document | Seconds from "New document" to status "sent", median and p90 over participants | Median ≤ 120 s (NFR-08) | UT-01 |
| OM-6 | Delivery and notification time | Seconds from "sent" to the document in the recipient's inbox, p95 | ≤ 5 s (FR-06, FR-11) | E2E with timestamps |
| OM-7 | API response time | p95 of list / metadata requests; p95 of verification | ≤ 500 ms; ≤ 3 s (NFR-07) | PT-01 |
| OM-8 | Usability | Mean SUS score | ≥ 68 (NFR-08) | UT-01 |

The aim is considered achieved if OM-1 – OM-4 meet their targets (security properties of the aim) and at least three of OM-5 – OM-8 meet theirs.

## 5. Measurement conditions

### 5.1 Reference environment

| Parameter | Value |
|---|---|
| Deployment | Docker Compose as in [07-architecture.md §7](07-architecture.md#7-deployment), one host |
| Host | 4 vCPU, 8 GB RAM, SSD, Ubuntu 24.04 LTS (record actual hardware and versions in the test report) |
| Network | Load generator on the same LAN; NCA RK test OCSP / CRL reachable over the Internet (one run of ST-05 with OCSP blocked) |
| Client | Current Chrome, Edge and Firefox; screen 1366×768; NCALayer current version from pki.gov.kz |
| Versions | Git tag of the tested build, image digests and tool versions recorded in the report |

### 5.2 Test data

- 3 organizations, 20 users covering all roles, 1,000 documents (PDF 100 KB – 25 MB; JPEG, PNG), generated by a seed script.
- All persons, IINs and documents are synthetic (NFR-10).

### 5.3 Signature corpus

- NCA RK test certificates: valid individual and legal-entity keys, a revoked key, an expired key.
- A certificate from an untrusted CA.
- For each case, at least 5 documents. Valid cases: at least 20 signatures.

### 5.4 Procedure

- Performance (PT-01): 2 min warm-up, then 10 min with 50 virtual users. Each measurement is run 3 times; the median of the three p95 values is reported.
- Timing in usability (UT-01):
  - at least 5 participants from the target roles (doctors, registrars or medical students in the absence of staff);
  - one scripted task: send a referral to another organization;
  - each participant does the task once in MedExchange and once in the baseline (§6), in counterbalanced order.
- Security tests: run on a fresh environment created from the tagged build; results are recorded per test ID with pass / fail and evidence (logs, screenshots, tool reports).

## 6. Baseline for comparison

**Baseline:** current practice described in the problem statement ([proposal §3](01-proposal.md#3-problem-statement), assumption A-01 in [02-evidence/interviews.md](02-evidence/interviews.md)). A document is printed or scanned, signed by hand and sent as a PDF attachment by e-mail (a general-purpose mailbox) or through a messenger.

The baseline is reproduced in the test environment and evaluated on the same criteria:

| Criterion | How it is measured on both | Baseline (expected) | MedExchange (target) |
|---|---|---|---|
| Recipient can verify the author and integrity | Signature corpus cases sent through both channels; can the recipient tell which ones are invalid? | Not possible: a scan carries no verifiable signature | 100% of invalid cases rejected (OM-2) |
| Modification detected | Modify the file after sending (in the mailbox / storage) | Not detected | Detected on download (FR-17) |
| Access restricted to the recipient organization | Forwarding and access by other users | Anyone with the file or mailbox access | 0 unauthorized accesses (OM-1) |
| Record of who opened the document | Check available logs | No record available to the sender | 100% of opens audited (OM-3) |
| Patient consent recorded | Check for a consent record | No | Yes, for 100% of documents (FR-05) |
| Time to send | OM-5 in UT-01 | Measured | Not worse than the baseline + 30 s, and ≤ 120 s |
| Delivery status visible to sender | Does the sender know the document was opened? | No | Yes (FR-11) |

A general-purpose electronic document management system (e.g. Documentolog) is discussed as an alternative in the existing systems comparison ([DIP-7](https://medexchange-diploma.youtrack.cloud/issue/DIP-7)). It is not used as the measured baseline, because its license and medical configuration are not available to the team.

## 7. Traceability

| Requirement group | Covered by |
|---|---|
| Must FR (FR-01 – FR-12, FR-15) | API tests (one test per acceptance criterion), E2E for UC-01 – UC-09 |
| Should / Could FR | API tests where implemented; listed as "not implemented" in the report otherwise |
| Security NFR (NFR-01 – NFR-06, NFR-12 – NFR-26) | ST-01 – ST-17 |
| NFR-07 | PT-01 |
| NFR-08 | UT-01 |
| NFR-09 | E2E in three browsers |
| NFR-10 | CL-01 |
| NFR-11 | DT-01 |
| Threats T-01 – T-34 | ST-xx listed per threat in [threat-model.md §4](security/threat-model.md#4-threats-and-mitigations) |

## 8. Schedule and responsibilities

| Milestone ([10-project-management.md §3](10-project-management.md#3-milestones)) | Verification activities | Responsible |
|---|---|---|
| M2 Iteration 1 | CI pipeline with unit tests, SAST, secret and dependency scans; ST-03, ST-04 | Alikhan (CI, tests), Aikyn (API unit tests) |
| M3 Iteration 2 | Signature corpus and ST-05; ST-06; integration tests | Alikhan, Aikyn |
| M4 Pre-defense 2 | E2E for the core scenario; ST-01, ST-02; first PT-01; DT-01 | Alikhan, Damir (E2E) |
| M5 Iteration 3 | ST-07, ST-08, ST-09, ST-14, ST-17 | Alikhan |
| M6 Iteration 4 | Full security run ST-01 – ST-17, ZAP, testssl, PT-01, UT-01 with baseline, CL-01; fixes | Alikhan (lead), all |
| M7 Final pre-defense | Test report with all metrics OM-1 – OM-8 and comparison with the baseline | Alikhan |

## 9. Reporting

Results are recorded in `docs/test-report.md` (from M4), containing:

- the tested build (tag);
- the environment;
- the pass/fail of every test ID;
- the value of every metric OM-1 – OM-8 with raw data;
- open findings with severity;
- the comparison with the baseline.

Failed Must tests and High/Critical findings are tracked as YouTrack bugs linked to the requirement.
