# Project Proposal

YouTrack: [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2)

## 1. Project title

| Language | Title |
|---|---|
| EN | Development of a secure system for the exchange of digital documents between medical organizations |
| RU | Разработка защищённой системы обмена цифровыми документами между медицинскими организациями |
| KZ | Медициналық ұйымдар арасында цифрлық құжаттармен алмасудың қауіпсіз жүйесін әзірлеу |

Working name: **MedExchange**.

## 2. Team and supervisor

| Member | GitHub | Role | Main responsibilities |
|---|---|---|---|
| Sagyntai Aikyn | [@mathalama](https://github.com/mathalama) | Backend Lead, Team Lead | Proposal, technology choice, architecture, backend API, EDS integration, repository and planning |
| Korazbay Alikhan | [@AlikShal](https://github.com/AlikShal) | Security Analyst & QA Engineer | Literature review, existing systems, threat model, security requirements, verification plan, testing |
| Almasuly Damir | [@smuzyyq](https://github.com/smuzyyq) | Frontend Developer & UX Designer | User interviews, functional requirements, use cases, UX prototype, web client |

**Supervisor:** Bakiyeva Aigerim Muratovna, Assistant Professor.

## 3. Problem statement

Medical organizations regularly exchange documents about patients: referrals, discharge summaries, test results, medical certificates and extracts from medical records. In practice a significant part of this exchange still happens outside a single protected channel: paper documents, scanned copies sent by e-mail, or files sent through messengers.

This creates the following problems:

1. **Confidentiality.** Medical data is a special category of personal data. Sending it through general-purpose channels (personal e-mail, messengers) exposes it to unauthorized access and leaks.
2. **Integrity and authenticity.** A scanned or forwarded document can be modified, and the recipient cannot reliably verify who issued it and whether it was changed after signing.
3. **Lack of accountability.** There is no reliable log of who sent, received, opened or forwarded a document, which makes incident investigation difficult.
4. **Delays and manual work.** Staff spend time on printing, scanning, phone calls and re-entering data; a document may be lost or reach the wrong recipient.
5. **Interoperability.** Organizations use different information systems; documents are not exchanged in a common structured format.

The exchange must comply with the legislation of the Republic of Kazakhstan:

- Law of the RK "On Electronic Document and Electronic Digital Signature" No. 370-II of 7 January 2003 — legal validity of electronic documents signed with an electronic digital signature (EDS);
- Law of the RK "On Personal Data and Their Protection" No. 94-V of 21 May 2013 — collection, processing and protection of personal data;
- Code of the RK "On Public Health and the Healthcare System" No. 360-VI of 7 July 2020, Article 60 "Digital healthcare" — processing of personal medical data in digital healthcare.

In Kazakhstan, EDS keys are issued to individuals and organizations by the National Certification Authority of the RK (NCA RK); users sign documents in web applications through the NCA RK application NCALayer.

> Evidence status: the problem description above is based on publicly available information and the team's assumptions. It will be confirmed or corrected by user interviews ([DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5), `docs/02-evidence/`). The literature review ([03-literature-review.md](03-literature-review.md)) and the existing systems comparison ([04-existing-systems.md](04-existing-systems.md)) support problems 1–3 and 5 from published sources ([DIP-7](https://medexchange-diploma.youtrack.cloud/issue/DIP-7)).

## 4. Target users

| User | Needs |
|---|---|
| Doctor | Send and receive patient documents quickly, sign them with EDS, be sure the received document is authentic |
| Registrar / medical records staff | Route incoming documents, track their status, find documents by patient |
| Organization administrator | Manage users and roles inside the organization, connect the organization to the exchange |
| IT / information security specialist | Control access, review the audit log, investigate incidents |
| Receiving medical organization | Get documents in a structured, verifiable format |

## 5. Aim

To develop a web-based system that allows medical organizations to exchange digital medical documents securely, ensuring confidentiality, integrity, authenticity (EDS of the National Certification Authority of the Republic of Kazakhstan) and full traceability of all actions.

## 6. Objectives

1. Analyze the problem domain, existing systems, regulations and scientific publications on secure medical data exchange.
2. Collect and specify functional, non-functional and security requirements based on user interviews.
3. Design the system architecture, data model and threat model.
4. Implement the backend API: organizations, users and roles, sending and receiving documents, document statuses.
5. Implement signing of documents with the user's own EDS in the browser (NCALayer), server-side signature verification with the official NCA RK KalkanCrypt SDK, and encryption of stored documents.
6. Implement an audit log of all actions with documents.
7. Implement the web client for the key user scenarios.
8. Test the system (functional, integration and security testing) and evaluate it against the baseline (e-mail exchange of scanned documents).

## 7. Expected outcomes

- A working prototype of the system, deployable with Docker.
- Exchange of documents between at least two organizations: the sender signs the document with their EDS via NCALayer, the system verifies the signature and the certificate status before delivery, and the recipient sees the verification result.
- Role-based access control: a user can see only the documents of their organization and role.
- An audit log that records who did what with each document and when.
- Documents described in the HL7 FHIR format (`DocumentReference`) for interoperability.
- Documentation: requirements, architecture, threat model, test report.

## 8. Project scope

### In scope

- Web client (browser) for doctors, registrars and organization administrators.
- Backend REST API.
- Management of organizations, users and roles (RBAC).
- Sending, receiving, viewing and downloading documents (PDF and other files with metadata).
- Signing of documents in the browser with the user's EDS through NCALayer; private keys never leave the user's device.
- Server-side verification of signatures and certificate status (OCSP/CRL) by a signature service based on the official KalkanCrypt Java SDK (NCA RK test certificates during development).
- Encryption of data in transit (TLS) and application-level encryption of stored files.
- Document statuses (sent, delivered, opened, signed, rejected) and notifications in the web client.
- Audit log.
- Document metadata in HL7 FHIR `DocumentReference` format.

### Out of scope

- Issuing EDS keys and certificates (this is done by NCA RK) and storing users' private keys on the server.
- A full FHIR server and full implementation of the FHIR specification.
- Integration with real medical information systems (Damumed, KMIS, etc.) and national eGov services; the API is designed to make such integration possible later.
- Mobile applications.
- Electronic medical records, scheduling, billing and other hospital information system functions.
- Use of real patient data: only test data is used.
- Production certification of the system.

## 9. Engineering complexity

The project combines several non-trivial engineering tasks: client-side signing through NCALayer and server-side verification with the national cryptographic library (KalkanCrypt, NCA RK), a multi-service architecture (Python API + Java signature service), encryption and key management, role-based access control across several organizations, a tamper-evident audit log, a standard data format (HL7 FHIR) and security testing based on a threat model.

## 10. Main risks (summary)

Detailed risk register: `docs/10-project-management.md` ([DIP-8](https://medexchange-diploma.youtrack.cloud/issue/DIP-8)).

- Access to the KalkanCrypt SDK and NCA RK test keys, and the transition of NCA RK to GOST 34.10-2015, may delay the signature service.
- NCALayer must be installed and running on the user's workstation; the demo environment must include it.
- Limited access to real users and organizations for interviews and validation.
- Limited time of a three-person team.
