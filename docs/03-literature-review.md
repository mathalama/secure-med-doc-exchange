# Literature Review

YouTrack: [DIP-7](https://medexchange-diploma.youtrack.cloud/issue/DIP-7) · Owner: Korazbay Alikhan

| Field | Value |
|---|---|
| Version | 0.1 (pre-defense 1) |
| Status | Draft for team review |
| Related | [Proposal](01-proposal.md), [Existing systems](04-existing-systems.md), [Technology comparison](05-technology-comparison.md), [Requirements](06-requirements.md), [Architecture](07-architecture.md) |

This review summarizes what is already known about the secure exchange of medical documents between organizations, what limitations existing approaches have, and which of them are relevant for MedExchange. Existing products and national systems are compared separately in [04-existing-systems.md](04-existing-systems.md).

## 1. Method

**Research questions.**

- RQ1. Which approaches are used to protect medical data exchanged between organizations (encryption, access control, digital signatures, audit)?
- RQ2. What does HL7 FHIR provide for document exchange, and what does it leave to the implementer?
- RQ3. What regulatory and national context (Republic of Kazakhstan) constrains such a system?
- RQ4. Which gaps remain that MedExchange should address?

**Sources searched.** Crossref, PubMed / Europe PMC, Google Scholar, publisher sites (Elsevier, Springer, JMIR, OUP, ACM, MDPI), the official legal database of the RK (Adilet), and official pages of the Ministry of Health of the RK and the Republican Center for Electronic Health (RCEZ).

**Keywords.** "health information exchange security", "FHIR security privacy", "access control electronic health records", "attribute-based encryption health", "digital signature electronic health record", "audit log electronic health record", "tamper-evident log", "healthcare ransomware", "Kazakhstan digital health", "Kazakhstan electronic health record".

**Inclusion criteria.**

- Peer-reviewed journal articles, conference papers and systematic / scoping reviews, published 2021–2026.
- Older works only if they are foundational (a technique MedExchange uses directly) or the main description of a national system.
- Laws, standards and official documents as regulatory context.
- English or Russian.

**Verification.** Bibliographic data of every scientific source was checked against Crossref by DOI, and the claims were taken from the abstract or full text.

**Limitation of the method.** This is a targeted (narrative) review, not a systematic review following PRISMA: the screening was not double-blind and the number of screened records was not recorded. To reduce this bias, reviews (systematic and scoping) are used as the main sources for each theme.

**Result.** 22 sources: 12 scientific publications (10 of them published in 2021–2026) and 10 laws, standards and official documents.

## 2. Thematic review

### 2.1 Threat landscape: why medical data exchange needs dedicated protection

Neprash et al. [9] analyzed 374 ransomware attacks on US health care delivery organizations in 2016–2021. Together these attacks exposed the personal health information of nearly 42 million patients. The annual number of attacks more than doubled (43 → 91), and 44.4% of attacks disrupted care delivery. Over time, attacks were less often recovered from backups and more often exceeded mandatory reporting deadlines. The study is limited to the USA and to publicly reported incidents, so the real numbers are likely higher. Still, it shows the two properties that matter most for MedExchange: **confidentiality** (exposure of patient data) and **integrity and availability** (documents that cannot be trusted or opened).

Implication: medical documents should not be exchanged through general-purpose channels. Stored files should be encrypted and hashed (NFR-02, FR-17), and the system should keep a verifiable record of who did what (FR-15).

### 2.2 Secure sharing architectures and encryption

Xi et al. [3] reviewed blockchain-based approaches to sharing healthcare data: EMR sharing, the Internet of Medical Things and federated learning. They conclude that blockchain and smart contracts suit medical data because they are tamper-proof and traceable. The review mostly describes proposed architectures. It does not compare them with conventional centralized systems on cost, performance or legal validity, and most of the reviewed systems are prototypes.

Imam et al. [4] systematically reviewed attribute-based encryption (ABE) for health information exchange in cloud settings and grouped the works into 10 domains. ABE gives fine-grained, policy-based access to encrypted records. Two limitations matter for this project. First, the authors note that most frameworks remain research proposals. Second, ABE adds key-management and revocation complexity that a small team cannot validate within one academic year.

Critical assessment. Both lines of research solve the "who can decrypt" problem cryptographically. Neither addresses **legally valid authorship**, which in the RK requires an EDS issued by NCA RK [13]. Blockchain also conflicts with the right to delete or correct personal data. For MedExchange, the transferable ideas are tamper evidence and traceability. These can be achieved with a hash-chained audit log (§2.5) and per-file encryption (NFR-02), without a distributed ledger.

### 2.3 Access control

Cobrado et al. [5] analyzed 20 journal articles on access control in EHR systems (PRISMA 2020). They grouped mechanisms into identification, authentication, authorization and accountability (IAAA). ABAC was the most popular authorization model. Unique IDs, digital signatures and access-control logs were the most common mechanisms for identification, authentication and accountability respectively. The research gaps they found were:

- missing multi-factor authentication;
- emergency access;
- patient consent;
- accountability, which only 2 of 20 studies addressed.

The sample is small, and most studies are designs rather than evaluated deployments.

Critical assessment. The gaps line up with the MedExchange requirements:

| Gap | MedExchange requirement |
|---|---|
| Multi-factor authentication | FR-01 (TOTP) |
| Patient consent | FR-05 (consent confirmation) |
| Accountability | FR-15, FR-16 (audit) |
| Isolation between organizations | NFR-06 |

The project uses RBAC (FR-02) with an organization boundary on every request (NFR-06), not full ABAC. Its roles are few and fixed (doctor, registrar, administrator, security officer), so ABAC policy engines would add complexity without a clear benefit at this scope. Emergency ("break-glass") access is out of scope and is recorded as a limitation.

### 2.4 Digital signatures

Felisberto et al. [6] screened 8,834 studies on digital signatures in implemented EHR systems and could include only nine. Five were limited to drug prescriptions, two concerned images, and two did not specify the purpose. XML was the most frequent signature storage format. The authors recommend further research, including the cost of storing signed health files. The small number of included studies shows that signatures on *exchanged* clinical documents are poorly covered by the literature, even though the underlying cryptography (PKI, CMS/XML signatures) is mature.

The Estonian national e-health system [10] is a working example at national scale. Every medical document is digitally signed by the physician or digitally stamped by the institution. Users authenticate with the national ID card / mobile ID / Smart-ID, and the X-Road exchange layer signs and encrypts all outgoing data.

In Kazakhstan, an EDS issued by NCA RK gives an electronic document the same legal force as a handwritten signature [13]. The 2013–2020 e-health concept of the Ministry of Health set the goals that "all users have a digital signature" and "all information systems use EDS where a handwritten signature is required" [16].

Critical assessment. The literature confirms that a document signature (not only TLS between servers) is needed for non-repudiation. It also confirms that the signature must come from a nationally recognized PKI. This supports FR-07 / FR-08 and [ADR 0005](adr/0005-eds-signing-and-verification.md): CMS signatures are created by the user through NCALayer and verified against NCA RK, including certificate status.

### 2.5 Audit logging and tamper evidence

Schneier and Kelsey [8] (foundational) described a computationally cheap method that makes log entries written before a compromise impossible to modify or delete without detection. It chains each entry to the previous one with hashes and evolving keys. This is the basis of the hash-chained audit log in NFR-05.

Rule et al. [7] reviewed 102 studies (2019–2021) that use EHR event logs. Only 8 of them validated their measures, and definitions varied widely between vendors and investigators. The review is about measuring clinician work, not security, but it is relevant here: logs exist in most EHRs, yet without a defined schema they are hard to interpret and compare. MedExchange therefore defines a fixed set of audit events with a fixed structure (FR-15: actor, action, object, time, IP).

In Estonia [10], an "untamperable and unremovable" audit trail is one of six security techniques of the national system. Patients can see in the portal who accessed their data, and suspicious access is investigated.

Critical assessment. Tamper evidence is well understood. The gap is *who reviews the log*: Rule et al. show that logs are rarely validated. MedExchange gives the log to a security officer role with filtering, export and an integrity check (FR-16). A patient-facing access log, as in Estonia, is out of scope (no patient portal, [proposal §8](01-proposal.md#8-project-scope)).

### 2.6 HL7 FHIR for interoperability

Ayaz et al. [1] systematically reviewed 80 FHIR articles (2012–2019) selected from 8,181 records. They identified implementation models, used resources, data migration approaches and open challenges, and concluded that FHIR will play a central role in health care. The review covers the period before the FHIR R4 normative release became widespread, which limits it.

Pavão et al. [2] carried out a scoping review of security and privacy mechanisms combined with FHIR. Of 37 included studies, 19 used blockchain for secure data sharing, secure personal health records, authentication and auditing, or monitoring access to clinical data. In other words, security is usually added *around* FHIR, not provided by it.

This matches the FHIR R4 specification itself [19]. FHIR "is not a security protocol, nor does it define any security related functionality". It recommends TLS for all production exchange and SMART on FHIR / OAuth for authorization, and it provides the `AuditEvent` and `Provenance` resources and reserved places for digital signatures.

Critical assessment. FHIR solves the *format* problem (a structured, machine-readable description of a document). It leaves authentication, authorization, signing and audit to the implementer. This is the division of work in [ADR 0004](adr/0004-document-format.md): a FHIR R4 `DocumentReference` describes the document (FR-12), and security is provided by the system (FR-01, FR-02, FR-07, FR-15, NFR-01, NFR-02).

### 2.7 Kazakhstan: national context and exchange formats

- **National data.** Gusmanov et al. [12] describe the research databases of the Unified National Electronic Health System (UNEHS), which include medical data on 36.4% of the adult population. They are used for epidemiology and real-world evidence. These are centralized registries for analysis, not a channel through which organizations exchange signed documents.
- **Data standards.** Nurgaliyeva et al. [11] catalogued 157 mandatory paper-era medical forms and developed 209 standardized classifications and 83 digital data sets to replace 63 of the forms, guided by ISO 13940. They conclude that "the top-down approach to digital health standardization, combined with open market policies, results in inconsistent data collection practices", and that the national legislation still has to be transformed into a digital-friendly mode.
- **Exchange standard.** The 2013–2020 e-health concept [16] chose **HL7 v3** as the target exchange standard (over HL7 v2 and EN 13606/openEHR) and planned a health interoperability platform with an integration bus [16], [18]. The RCEZ page of the platform does not name a specific standard [18]. The Ministry of Health sets minimum requirements for all medical information systems regardless of ownership (Order No. ҚР ДСМ-80 of 6 August 2021 [17]). **No public document requiring HL7 FHIR in the RK was found during this review**; this confirms the limitation recorded in [05-technology-comparison.md](05-technology-comparison.md#decision-5-document-format).
- **Exchange in practice.** In May 2023 the Government stated that one MIS covers over 80% of the market. It also stated that the lack of a full mechanism for exchanging data between medical information systems made it hard for organizations to change systems. The MIS database was transferred to the Ministry of Health to build the national repository "E-Densaulyk" [22]. See [04-existing-systems.md](04-existing-systems.md) for details.

Implication for ADR 0004: FHIR `DocumentReference` stays the project format because it is an open international standard. National classifications (document types, organization IDs) and a possible mapping to national formats are recorded as future work, not as a requirement for pre-defense 1.

### 2.8 Regulation

| Regulation | What it requires | Effect on MedExchange |
|---|---|---|
| Law of the RK No. 370-II "On Electronic Document and EDS" [13] | An EDS issued by an accredited certification authority (NCA RK) makes an electronic document legally equivalent to a signed paper one | FR-07, FR-08, ADR 0005 |
| Law of the RK No. 94-V "On Personal Data and Their Protection" [14] | Lawful basis and consent for processing; protection measures; storage of personal data in databases located in the RK (Art. 12) | FR-05 (consent), NFR-02, NFR-10; deployment in the RK |
| Code of the RK No. 360-VI, Art. 60 "Digital healthcare" [15] | Processing of personal medical data within digital healthcare by authorized subjects | NFR-10; organization registration (FR-20) |
| Order ҚР ДСМ-80 (2021) [17] | Minimum requirements for medical information systems | To be checked in detail before integration with MIS (FR-18) |
| GDPR, Regulation (EU) 2016/679 [20] (comparison) | Health data is a special category (Art. 9); security of processing including encryption and pseudonymisation (Art. 32) | Confirms the same controls as international practice |
| HIPAA Security Rule, 45 CFR §164.312 [21] (comparison) | Technical safeguards: access control, audit controls, integrity, person or entity authentication, transmission security | Each safeguard maps to FR-02, FR-15, FR-17, FR-01, NFR-01 |

## 3. Analysis table of publications

| # | Publication | Type | Approach | Results | Limitations |
|---|---|---|---|---|---|
| [1] | Ayaz et al., 2021, JMIR Med Inform | Systematic review | Search of 6 databases (2012–2019); 80 of 8,181 articles analyzed | Catalogued FHIR implementation models, resources, migration approaches, challenges; FHIR expected to be central to health interoperability | Covers the period before wide use of FHIR R4; security only one of many topics |
| [2] | Pavão, Bastardo, Rocha, 2023, ICITS (Springer LNNS) | Scoping review | Review of studies integrating security / privacy mechanisms with FHIR; 37 studies | 19 of 37 studies use blockchain for sharing, PHR, authentication, auditing, access monitoring; security is added around FHIR | Short conference paper; dominated by blockchain prototypes; few production evaluations |
| [3] | Xi et al., 2022, Applied Sciences 12(15) | Review | Analysis of blockchain approaches for EMR sharing, IoMT, federated learning | Blockchain and smart contracts give tamper-proof, traceable sharing | No comparison with centralized systems on cost, performance or legal validity; mostly prototypes |
| [4] | Imam et al., 2022, J. King Saud Univ. CIS 34(9) | Systematic review | ABE frameworks for e-health data sharing classified into 10 domains | ABE is a reliable model for fine-grained access to encrypted health data in the cloud | Mostly research proposals; key management and revocation complexity; no legal-signature aspect |
| [5] | Cobrado et al., 2024, Informatics in Medicine Unlocked 49 | Systematic review (PRISMA 2020) | 20 journal articles grouped by IAAA | ABAC most popular; gaps: MFA, emergency access, patient consent, accountability (2 of 20 studies) | Small sample; designs more than evaluated deployments |
| [6] | Felisberto et al., 2024, Health and Technology 14(6) | Scoping review | Digital signatures in implemented EHRs; 9 of 8,834 studies included | Most signature use limited to prescriptions; XML most frequent format | Very few eligible studies; signatures on exchanged documents not covered |
| [7] | Rule, Melnick, Apathy, 2023, JAMIA 30(1) | Scoping review | 102 studies using EHR event logs (2019–2021) | Logs widely used, but definitions inconsistent; only 8 studies validated measures | Focus on clinician workload, not security; US-centred |
| [8] | Schneier, Kelsey, 1999, ACM TISSEC 2(2) | Foundational method | Hash chaining and evolving keys for log entries | Entries before compromise cannot be read, changed or deleted undetectably | Requires protecting the initial secret; detection, not prevention |
| [9] | Neprash et al., 2022, JAMA Health Forum 3(12) | Cohort study | 374 ransomware attacks on US care delivery organizations, 2016–2021 | Attacks doubled; ~42 M patients' PHI exposed; 44.4% disrupted care | US only; publicly reported incidents only |
| [10] | Metsallik et al., 2018, CEUR-WS Vol. 2336 | Case study (national system) | Description of Estonia's EHIS: HL7 CDA, X-Road, national eID | Signed/stamped documents, encrypted records, untamperable log, patient-visible access log; 1,163 institutions connected by 2017 | Older than 5 years; relies on national eID and X-Road; semantic interoperability and staff acceptance reported as problems |
| [11] | Nurgaliyeva et al., 2024, Int J Med Inform 192 | Applied standardization study | Cataloguing 157 mandatory forms; ISO 13940-based data sets | 209 classifications, 83 data sets replacing 63 forms | Data sets not yet adopted into legislation; no exchange format defined |
| [12] | Gusmanov et al., 2023, Int J Med Inform 170 | Review of national databases | Description of UNEHS research databases | Data on 36.4% of adult population usable for epidemiology | Registry and analytics focus; not a document exchange channel; data quality concerns |

## 4. Synthesis: what is known, what is missing, why MedExchange

| # | What is known | Limitation / gap | How MedExchange responds |
|---|---|---|---|
| 1 | Attacks on health data are growing and disrupt care [9] | General channels (e-mail, messengers) give no protection | Dedicated channel, TLS (NFR-01), encrypted storage (NFR-02), hash check on download (FR-17) |
| 2 | Cryptographic sharing (ABE, blockchain) gives fine-grained or tamper-proof sharing [3], [4] | Research prototypes; no legally valid authorship | Simpler RBAC + organization boundary (FR-02, NFR-06); tamper evidence via hash-chained log (NFR-05) |
| 3 | IAAA is the right frame; MFA, consent and accountability are frequent gaps [5] | Gaps remain in most proposed systems | TOTP (FR-01), consent confirmation (FR-05), audit and review (FR-15, FR-16) |
| 4 | Signatures in EHRs are rarely studied beyond prescriptions [6]; national PKI works at scale [10] | No published system for signed document exchange between organizations in the RK | NCA RK EDS through NCALayer, server-side verification incl. OCSP/CRL (FR-07, FR-08, ADR 0005) |
| 5 | Tamper-evident logging is well established [8]; logs are rarely validated [7] | Logs exist but are not reviewed or are ill-defined | Fixed audit schema and a security officer role with integrity check (FR-15, FR-16) |
| 6 | FHIR is the leading interoperability standard but defines no security [1], [2], [19] | Security has to be designed around it | FHIR `DocumentReference` for format (FR-12); system-level security |
| 7 | Kazakhstan has national registries and digitized forms [11], [12]; HL7 v3 was the 2013 target [16]; no FHIR mandate found | Exchange between MIS of different vendors is incomplete [22] | A vendor-neutral exchange with an open format and a REST API for MIS (FR-18) |

**Conclusion.** The individual techniques MedExchange uses (encryption, RBAC, PKI signatures, hash-chained audit, FHIR) are well known. The gap is the lack of a system that *combines* them for document exchange **between** medical organizations in the RK, with:

- NCA RK EDS for legal validity,
- binding of each document to a patient,
- isolation between organizations,
- an audit trail of access to patient documents,
- an open document format.

The existing systems compared in [04-existing-systems.md](04-existing-systems.md) cover parts of this combination, but not all of it.

## Sources

Scientific publications:

1. Ayaz M., Pasha M., Alzahrani M., Budiarto R., Stiawan D. The Fast Health Interoperability Resources (FHIR) Standard: Systematic Literature Review of Implementations, Applications, Challenges and Opportunities. *JMIR Medical Informatics*, 2021, 9(7): e21929. https://doi.org/10.2196/21929
2. Pavão J., Bastardo R., Rocha N. Integrating Security and Privacy Mechanisms with Fast Health Interoperability Resources (FHIR), a Scoping Review. In: *Information Technology and Systems (ICITS 2023)*, Lecture Notes in Networks and Systems, Springer, 2023, pp. 387–395. https://doi.org/10.1007/978-3-031-33261-6_33
3. Xi P., Zhang X., Wang L., Liu W., Peng S. A Review of Blockchain-Based Secure Sharing of Healthcare Data. *Applied Sciences*, 2022, 12(15): 7912. https://doi.org/10.3390/app12157912
4. Imam R., Kumar K., Raza S., Sadaf R., Anwer F., Fatima N., Nadeem M., Abbas M., Rahman O. A systematic literature review of attribute based encryption in health services. *Journal of King Saud University – Computer and Information Sciences*, 2022, 34(9): 6743–6774. https://doi.org/10.1016/j.jksuci.2022.06.018
5. Cobrado U., Sharief S., Regahal N., Zepka E., Mamauag M., Velasco L. Access control solutions in electronic health record systems: A systematic review. *Informatics in Medicine Unlocked*, 2024, 49: 101552. https://doi.org/10.1016/j.imu.2024.101552
6. Felisberto M., de Oliveira J., Mohr E., et al. Digital signatures in electronic health records: a scoping review. *Health and Technology*, 2024, 14(6): 1083–1096. https://doi.org/10.1007/s12553-024-00906-y
7. Rule A., Melnick E., Apathy N. Using event logs to observe interactions with electronic health records: an updated scoping review shows increasing use of vendor-derived measures. *Journal of the American Medical Informatics Association*, 2023, 30(1): 144–154. https://doi.org/10.1093/jamia/ocac177
8. Schneier B., Kelsey J. Secure audit logs to support computer forensics. *ACM Transactions on Information and System Security*, 1999, 2(2): 159–176. https://doi.org/10.1145/317087.317089 (foundational)
9. Neprash H., McGlave C., Cross D., et al. Trends in Ransomware Attacks on US Hospitals, Clinics, and Other Health Care Delivery Organizations, 2016–2021. *JAMA Health Forum*, 2022, 3(12): e224873. https://doi.org/10.1001/jamahealthforum.2022.4873
10. Metsallik J., Ross P., Draheim D., Piho G. Ten Years of the e-Health System in Estonia. In: *Proceedings of the 3rd International Workshop on (Meta)Modelling for Healthcare Systems (MMHS 2018)*, CEUR Workshop Proceedings, Vol. 2336, 2018. https://ceur-ws.org/Vol-2336/MMHS2018_invited.pdf
11. Nurgaliyeva Z., Spatayev Y., Syla S., Yessenbayev B. Paving the way to establishing the digital-friendly health and care information model in Kazakhstan. *International Journal of Medical Informatics*, 2024, 192: 105610. https://doi.org/10.1016/j.ijmedinf.2024.105610
12. Gusmanov A., Zhakhina G., Yerdessov S., et al. Review of the research databases on population-based Registries of Unified electronic Healthcare system of Kazakhstan (UNEHS): Possibilities and limitations for epidemiological research and Real-World Evidence. *International Journal of Medical Informatics*, 2023, 170: 104950. https://doi.org/10.1016/j.ijmedinf.2022.104950

Laws, standards and official documents:

13. Law of the Republic of Kazakhstan No. 370-II of 7 January 2003 "On Electronic Document and Electronic Digital Signature". https://adilet.zan.kz/rus/docs/Z030000370_
14. Law of the Republic of Kazakhstan No. 94-V of 21 May 2013 "On Personal Data and Their Protection". https://adilet.zan.kz/rus/docs/Z1300000094
15. Code of the Republic of Kazakhstan No. 360-VI of 7 July 2020 "On Public Health and the Healthcare System". https://adilet.zan.kz/rus/docs/K2000000360
16. Ministry of Health of the RK. Concept of e-Health Development of the Republic of Kazakhstan for 2013–2020 (approved by Order No. 498 of 3 September 2013). https://www.gov.kz/uploads/2020/3/5/aa3c49fd804b3e8f715f6049fed1ba1b_original.2036350.pdf
17. Order of the Acting Minister of Health of the RK No. ҚР ДСМ-80 of 6 August 2021 "On approval of minimum requirements for medical information systems in the field of healthcare" (registered No. 23926). https://adilet.zan.kz/rus/docs/V2100023926
18. Republican Center for Electronic Health (RCEZ). Health interoperability platform. https://rcez.kz/interoperabilityplatform
19. HL7 International. FHIR R4 (4.0.1): Security. https://hl7.org/fhir/R4/security.html
20. Regulation (EU) 2016/679 (General Data Protection Regulation). https://eur-lex.europa.eu/eli/reg/2016/679/oj
21. 45 CFR §164.312 — HIPAA Security Rule, Technical safeguards. https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C/section-164.312
22. Tengrinews.kz. "Damumed database transferred to the Ministry of Health" (18 May 2023). https://tengrinews.kz/kazakhstan_news/bazu-dannyih-damumed-peredali-minzdravu-499361/

All online sources were accessed on 24.10.2026.
