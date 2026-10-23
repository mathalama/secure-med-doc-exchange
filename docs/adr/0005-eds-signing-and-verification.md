# ADR 0005: EDS — signing in the browser via NCALayer, verification in a Java signature service

- **Status:** Accepted
- **Date:** 2026-10-23
- **Task:** [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2)

## Context

Documents must be signed with a legally valid EDS issued by NCA RK (Law of the RK No. 370-II of 07.01.2003). The signer must control their private key. NCA RK provides the KalkanCrypt library officially only for Java and C; the Python wrapper `pykalkan` is a third-party project with reported vulnerabilities. In web applications users sign through NCALayer, an NCA RK application with a local WebSocket API. NCA RK is moving to GOST 34.10-2015. Detailed comparison: [05-technology-comparison.md](../05-technology-comparison.md#decision-1-eds-signing-and-verification).

## Decision

1. **Signing** is done in the browser: the web client sends the document hash or content to NCALayer, the user chooses their key and confirms, and NCALayer returns a CMS signature. Private keys never reach the server.
2. **Verification** is done by a separate **signature service** written in Java on the official KalkanCrypt SDK. It verifies the CMS signature against the document, the certificate chain to NCA RK root certificates and the certificate status (OCSP/CRL), and returns the signer's data (name, IIN/BIN, organization, validity).
3. The main API (FastAPI) accepts a document only if the signature service confirms the signature; the verification result is stored with the document and written to the audit log.
4. The signature service is reachable only inside the Docker network and has no access to the database or file storage.

## Alternatives considered

- **Verification with `pykalkan` inside the Python API** — a single codebase, but relies on an unofficial wrapper with reported vulnerabilities and uncertain support of new NCA RK algorithms.
- **Signing on the server with keys stored on the server** — rejected: the signer does not control the key, all keys are concentrated in one place, non-repudiation is lost.

## Consequences

- (+) Legally correct model: the user signs with their own key; the system only verifies.
- (+) Official, vendor-maintained cryptographic library for verification.
- (+) The EDS logic is isolated and testable on its own.
- (–) An additional Java service to build and deploy.
- (–) NCALayer must be installed on users' workstations and in the demo environment.
- (–) Access to the KalkanCrypt SDK and NCA RK test keys is required before implementation; this is a project risk tracked in the risk register.
