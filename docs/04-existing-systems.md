# Existing Systems Comparison

YouTrack: [DIP-7](https://medexchange-diploma.youtrack.cloud/issue/DIP-7) · Owner: Korazbay Alikhan

| Field | Value |
|---|---|
| Version | 0.1 (pre-defense 1) |
| Status | Draft for team review |
| Related | [Literature review](03-literature-review.md), [Proposal](01-proposal.md), [Requirements](06-requirements.md), [ADR 0004](adr/0004-document-format.md), [ADR 0005](adr/0005-eds-signing-and-verification.md) |

This document compares five systems that already solve part of the problem MedExchange addresses. It answers two questions:

1. What can existing solutions already do?
2. Why is a new system still needed?

Each claim about an existing system is supported by a source. Where the reviewed public sources say nothing about a criterion, the cell says **"not publicly documented"**. This means the feature was not found, not that it does not exist.

## 1. Systems selected

| # | System | Country | Type | Why it is relevant |
|---|---|---|---|---|
| S1 | **Documentolog** | Kazakhstan | General-purpose electronic document management (EDM) system | Market leader in EDM in the RK with NCA RK EDS; used by some medical organizations. The committee is likely to ask "why not Documentolog?" |
| S2 | **Damumed** | Kazakhstan | Medical information system (MIS) | The most widely used MIS in the RK; the main system that would send and receive medical documents |
| S3 | **Estonian e-Health (EHIS + X-Road)** | Estonia | National health information exchange | Reference model of signed, logged medical document exchange at national scale |
| S4 | **NHS MESH** | United Kingdom | National secure messaging between health organizations | Closest equivalent of an organization-to-organization document transport |
| S5 | **Epic Care Everywhere** | USA (+ other countries) | Vendor interoperability network of a large EHR | Largest commercial example of record exchange between providers |
| — | **MedExchange** (this project) | Kazakhstan | Secure exchange of signed documents between medical organizations | Planned design, given for comparison; nothing is implemented yet |

## 2. Criteria

The criteria come from the project drivers ([05-technology-comparison.md](05-technology-comparison.md#project-requirements-that-drive-the-decisions)) and the comments of the team on DIP-7.

| # | Criterion | What is checked | Driver |
|---|---|---|---|
| K1 | Legally valid signature (NCA RK EDS) | Can a document be signed with an EDS of NCA RK, and is the signature verified? | D1 |
| K2 | Medical document format / FHIR | Is the document described in a medical interoperability standard (HL7 FHIR, CDA)? | D3 |
| K3 | Patient binding | Is each document linked to a patient (IIN) and searchable by patient? | D2, D4 |
| K4 | Medical roles and data isolation between organizations | Are there medical roles, and can an organization see only documents addressed to it? | D2, D4 |
| K5 | Transport and storage protection | Encryption, mutual authentication, certification | D2 |
| K6 | Audit of access to patient data | Is access to a patient's document recorded, and can it be reviewed? | D2 |
| K7 | Integration with medical information systems | Is there an API or standard interface for MIS? | D3 |
| K8 | Availability in Kazakhstan and cost | Can a medical organization in the RK use it, and at what price? | D5, D6 |

## 3. Comparison

### 3.1 Detailed table

| Criterion | S1 Documentolog | S2 Damumed | S3 Estonia e-Health | S4 NHS MESH | S5 Epic Care Everywhere | MedExchange (planned) |
|---|---|---|---|---|---|---|
| **K1 Signature** | NCA RK EDS supported; Gamma, Russian EDS and Adobe Sign for cross-border documents [8], [5]. The user agreement treats EDS-signed documents as equivalent to handwritten ones, and the validity of EDS keys is checked through NCA RK [6]. Legally significant cross-border exchange with Russia through Kontur.Diadok since 2026 [3], [4] | EDS sign-in through NCALayer exists. An independent review (2021) reported a replay weakness in EDS authentication: the same data block is signed at every login [13]. EDS signing of medical documents: not publicly documented | Every medical document is signed by the physician or stamped by the institution with the Estonian national eID; X-Road signs all outgoing data [14], [15]. Not NCA RK | Certificates authenticate the *mailbox / endpoint*, not the document author [16], [17]. Document signature: not provided | Not publicly documented; no NCA RK support (US-oriented) [19] | User signs a CMS signature with own NCA RK EDS through NCALayer; chain and OCSP/CRL checked before delivery (FR-07, FR-08) |
| **K2 Format / FHIR** | No HL7 / FHIR or medical format found in the product, pricing or release notes [1], [2], [7], [8]. Generic documents (letters, contracts, HR, invoices) | Not publicly documented. The Government stated that a full data-exchange mechanism between MIS is missing [11] | HL7 CDA, DICOM, LOINC [14] | Payload-agnostic: any file up to 100 MB (2017) [16]. NHS ITK3 defines FHIR messaging that can carry any payload [18] | C-CDA documents, IHE XCA / XDS.b, Direct; FHIR R4 (US Core) for targeted resource search [19] | HL7 FHIR R4 `DocumentReference` + original file + CMS signature (FR-12) |
| **K3 Patient binding** | No patient entity or medical-record concept found; the only healthcare case found automates administrative document flow [9] | Patient-centred MIS; feeds the electronic health passport [10] | Health record per resident by personal ID number [14] | Messages are addressed to mailboxes (organization / workflow), not patients [16] | Patient record exchange between providers; 32 M charts exchanged daily [20] | Every document has patient IIN and name; search by patient (FR-04, FR-13) |
| **K4 Roles and isolation** | Generic roles and access levels per employee; separate system instance per client; administrators restricted from reading documents [8]. No medical roles | Not publicly documented | Only licensed professionals with a proven treatment relation can access data; patient can hide documents [14] | Mailbox per organization / service; certificate bound to mailbox ID and organization code [17] | Not publicly documented in the reviewed sources | Medical roles (doctor, registrar, administrator, security officer); server-side organization boundary on every request (FR-02, NFR-06) |
| **K5 Protection** | ST RK ISO/IEC 15408 certificate (EAL4); SIEM; brute-force protection; antivirus for uploads [8] | Replay weakness in EDS sign-in reported (2021) [13]; other measures not publicly documented | X-Road encrypts and signs all outgoing data; database records encrypted; personal data separated from medical data [14] | TLS with mutual certificate authentication; RSA 2048 keys [16], [17] | SAML-based authorization framework for network exchange [19]; storage encryption not publicly documented | TLS 1.2+, AES-256-GCM file encryption, SHA-256 hash check, TOTP (NFR-01, NFR-02, FR-17, FR-01) |
| **K6 Audit of patient data access** | Logging of all user actions [8]; not organized around patients | Not publicly documented | Untamperable audit trail; patient sees who accessed their data [14], [15] | Delivery tracking and reports in MOLES [16]; content access is outside MESH | Not publicly documented in the reviewed sources | Append-only hash-chained log of every action incl. open / download; security officer review and export (FR-15, FR-16, NFR-05) |
| **K7 MIS integration** | Integrations with 1C, Enbek.kz, IS ESF, SAP, e-government exchange (ESEDO) [7], [1]. No MIS integration found | Patient data flows into the electronic health passport on the national interoperability platform [10] | Federated: provider systems connect through X-Road [14], [15] | HTTP API and client for supplier systems [16] | Standard IHE / Direct / FHIR interfaces; Carequality and TEFCA networks [19] | REST API (OpenAPI) for MIS (FR-18, Could) |
| **K8 Availability in the RK and cost** | Available. Start: free (2 users); Business: from 405,000 ₸ / year for 15 users; Large: from 2,700,000 ₸ / year from 25 users [7] | Available; over 80% market share (Government, 2023) [11]; listed in the RCEZ register of medical digital systems [12]. Price not publicly documented | National infrastructure of Estonia; not available as a product in the RK [14] | NHS Digital service for health and social care organizations in England (9,000+ organizations in 2017) [16]; not available outside the NHS | Live in 11 countries [20]; no deployment in Kazakhstan found; enterprise license | Open design for the RK; deployed with Docker Compose (NFR-11) |

### 3.2 Summary matrix

Rating: **++** fully meets the criterion, **+** partly meets, **–** does not meet / not found, **n/a** not applicable.

| Criterion | S1 Documentolog | S2 Damumed | S3 Estonia | S4 MESH | S5 Epic | MedExchange (planned) |
|---|---|---|---|---|---|---|
| K1 NCA RK EDS | ++ | + | – (other national PKI) | – | – | ++ |
| K2 Medical format / FHIR | – | – | + (CDA) | + (payload-agnostic) | ++ | + (DocumentReference only) |
| K3 Patient binding | – | ++ | ++ | – | ++ | ++ |
| K4 Medical roles, isolation | + | – (not documented) | ++ | + | – (not documented) | ++ |
| K5 Protection | ++ | – | ++ | + | + | + (planned, not certified) |
| K6 Audit of patient data access | + | – (not documented) | ++ | + | – (not documented) | ++ |
| K7 MIS integration | – | ++ | ++ | ++ | ++ | + (planned API) |
| K8 Available in the RK | ++ | ++ | – | – | – | + (prototype) |

The MedExchange column describes the *planned* design from the requirements and is not evidence that it works. It is checked in later stages by the verification plan.

## 4. Why not Documentolog?

Documentolog is the strongest existing candidate in Kazakhstan:

- It has been on the market since 2007 [8] and is used by over 130 thousand Kazakh companies (vendor figure reported in February 2026 [3]).
- It supports NCA RK EDS with legal force [6].
- It is certified under ST RK ISO/IEC 15408 [8].
- It is already used for document flow in some medical organizations: City Polyclinic No. 9 (Astana) and No. 4 (Almaty) are listed as clients of its implementation packages [2], and JSC "CMTIS" with 8 subordinate organizations of the Medical Center of the Presidential Administration [9].

These cases automate **administrative** document flow: correspondence, orders, approvals. The CMTIS case describes internal routing and signing of organizational documents and no clinical processes [9]. No healthcare-specific module was found in the product pages, pricing or release notes [1], [2], [7].

Comparison on the criteria raised by the team:

| Criterion | Documentolog | MedExchange | Conclusion |
|---|---|---|---|
| EDS NCA RK | Supported with legal force; cross-border signatures [5], [6], [8] | Supported through NCALayer, verified with KalkanCrypt incl. OCSP/CRL (ADR 0005) | **Equal.** Documentolog is mature; MedExchange does not compete here |
| Medical document format / FHIR | Generic document types; no HL7 / FHIR found [1], [7], [8] | FHIR R4 `DocumentReference` with patient, author, organization, type, hash (FR-12) | MedExchange documents are machine-readable for MIS |
| Patient binding | No patient entity or patient search found [8], [9] | Mandatory patient IIN and name; search by patient (FR-04, FR-13) | Medical documents are tied to a person, not to correspondence |
| Medical roles and isolation between organizations | Generic roles; separate instance per client [8] | Doctor / registrar / administrator / security officer; only the doctor accepts with EDS; organization boundary on every request (FR-02, FR-10, NFR-06) | Roles reflect medical responsibility, not office hierarchy |
| Patient consent | Not found | Sender confirms patient consent before sending (FR-05, Law No. 94-V) | Required for medical personal data |
| Audit of access to patient data | Logging of all user actions [8] | Audit of open / download per document and patient, hash-chained, reviewed by a security officer (FR-15, FR-16, NFR-05) | Answers "who saw this patient's document?" |
| Integration with MIS | Accounting, HR, e-government integrations; no MIS integration found [7] | REST API for MIS (FR-18) | MedExchange is designed to sit between MIS |
| Cost | From 405,000 ₸ / year for 15 users (Business) to 2,700,000 ₸ / year (Large) [7] | Open-source prototype; hosting cost only | Lower entry cost; but no commercial support |

**Answer for the committee.** Documentolog solves *legally valid electronic correspondence* for any organization, and does it well. MedExchange solves a narrower problem, the *exchange of patient documents between medical organizations*. That problem needs things a general-purpose EDM does not provide out of the box:

- each document bound to a patient,
- medical roles and a doctor's acceptance,
- patient consent,
- an audit trail of access to patient data,
- a machine-readable medical format (FHIR) that an MIS can process.

The two are complementary. A future version could exchange administrative letters through EDM systems like Documentolog, while MedExchange carries clinical documents.

## 5. Conclusions

1. **No single system covers all eight criteria in Kazakhstan.**
   - Documentolog covers the legal side (K1, K5, K8) but not the medical side (K2, K3, K6).
   - Damumed covers the medical side inside one MIS, but exchange *between* systems is incomplete. In 2023 the Government named this a barrier to competition and changing systems [11].
2. **The foreign reference systems confirm the design.**
   - Estonia [14] shows that signed documents, patient-linked records and a tamper-proof access log work at national scale.
   - MESH [16] shows the value of a simple organization-to-organization channel with mutual certificate authentication.
   - Epic [19] shows that standard formats (C-CDA, FHIR) are what make exchange between different vendors possible.
   - None of them can be used in the RK: they rely on another national PKI, are restricted to their national health service, or are not deployed in Kazakhstan.
3. **MedExchange fills the gap.** It combines NCA RK EDS (as in Documentolog) with patient binding, medical roles, consent and audit (as in Estonia) and an open format (FHIR, as in Epic). It is limited to document exchange; it is not an MIS and not an EDM.
4. **Limitations of this comparison.**
   - It is based on public sources; vendor documentation for Damumed and Epic security is not public.
   - Documentolog's user count is a vendor figure.
   - The MedExchange column describes plans, not measured results.

## Sources

Documentolog:

1. Documentolog. "Documentolog Business updated to version 3.5" (13.11.2022). https://documentolog.com/en/blog/documentolog-business-obnovlen-do-versii-3-5
2. Documentolog. "Service packages for Documentolog implementation" (13.06.2021). https://documentolog.com/en/blog/pakety-uslug-po-vnedreniyu-sistemy-documentolog
3. CNews. "Kontur and Documentolog launched cross-border EDI between Russia and Kazakhstan" (03.02.2026). https://www.cnews.ru/news/line/2026-02-03_kontur_i_documentolog_zapustili
4. Documentolog. "Documentolog and Kontur.Diadok launched legally significant cross-border EDI between Kazakhstan and Russia" (01.02.2026). https://documentolog.com/en/blog/documentolog-i-kontur-diadok-zapustili-yuridicheski-znachimyy-transgranichnyy-edo-mezhdu-kazahstanom-i-rossiey
5. ComNews. "EDM system for electronic document exchange between CIS countries launched" (10.02.2020). https://www.comnews.ru/content/204510/2020-02-10/2020-w07/zapuschena-sed-dlya-elektronnogo-obmena-dokumentami-mezhdu-stranami-sng
6. Documentolog. User agreement (Business / Personal), clauses 1.6, 2.7, 7.6.1. https://documentolog.com/en/user-agreement-business-personal
7. Documentolog. Tariffs. https://documentolog.com/en/tariffs
8. Documentolog. "Electronic document management system Documentolog" — product presentation (2022). https://documentolog.com/files/presentation/Documentolog%20Platform.pdf
9. Documentolog. "Implementation of Documentolog EDMS at JSC Center for Medical Technologies and Information Systems" (31.07.2014). https://documentolog.com/en/blog/vnedrenie-sed-documentolog-v-ao-tsentr-meditsinskikh-tekhnologiy-i-informatsionnykh-sistem

Damumed and Kazakhstan:

10. Informburo.kz. "The Ministry of Health digitized the data of 17.3 million Kazakhstanis. What does the electronic health passport give?" (29.04.2019). https://informburo.kz/stati/minzdrav-ocifroval-dannye-173-mln-kazahstancev-chto-dayot-elektronnyy-pasport-zdorovya.html
11. Tengrinews.kz. "Damumed database transferred to the Ministry of Health" (18.05.2023). https://tengrinews.kz/kazakhstan_news/bazu-dannyih-damumed-peredali-minzdravu-499361/
12. RCEZ. List of medical and laboratory digital systems used by healthcare entities of the RK. https://rcez.kz/mis
13. Turekhanov V. "EDS Hall of Fame: alm.dmed.kz" (25.04.2021). https://ct.kz/hall-of-fame-eds/dmedkz/

Estonia:

14. Metsallik J., Ross P., Draheim D., Piho G. Ten Years of the e-Health System in Estonia. *CEUR Workshop Proceedings*, Vol. 2336, 2018. https://ceur-ws.org/Vol-2336/MMHS2018_invited.pdf
15. European Commission, Digital Building Blocks. Estonian Central Health Information System and Patient Portal. https://ec.europa.eu/digital-building-blocks/sites/pages/viewpage.action?pageId=533365863

United Kingdom:

16. NHS Digital. Message Exchange for Social Care and Health (MESH) — overview (February 2017). https://nhsconnect.github.io/ITK3-FHIR-Messaging-Distribution/pages/support/MESH_Overview_Generic.pdf
17. NHS England Digital. MESH certificate guidance. https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh/mesh-guidance-hub/certificate-guidance
18. NHS Digital. ITK3 FHIR Messaging Distribution. https://nhsconnect.github.io/ITK3-FHIR-Messaging-Distribution/

USA:

19. Epic. Care Everywhere — Supported Standards (last updated 06.06.2025). https://open.epic.com/Home/CareEverywhereSupportedStandards
20. Epic. Care Everywhere. https://www.epic.com/careeverywhere/

All online sources were accessed on 24.10.2026.
