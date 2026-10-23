# ADR 0001: Main server platform — Python + FastAPI

- **Status:** Accepted (revised 2026-10-23: EDS moved to a separate signature service, see ADR 0005)
- **Date:** 2026-10-19
- **Task:** [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2)

## Context

The main API serves several medical organizations: users and roles, documents and statuses, audit and FHIR resources. The prototype must be built by a three-person team within one academic year. NCA RK provides the KalkanCrypt library officially only for Java and C, so EDS verification is placed in a separate Java signature service (ADR 0005) and no longer constrains the choice of the main platform. Detailed comparison: [05-technology-comparison.md](../05-technology-comparison.md#decision-2-main-server-platform-api).

## Decision

Use **Python 3.12 with FastAPI**, SQLAlchemy (ORM), Alembic (migrations), Pydantic (validation) and `fhir.resources` (FHIR models) for the main API. EDS verification is requested from the signature service over an internal REST interface.

## Alternatives considered

- **Java + Spring Boot** — could use the official KalkanCrypt SDK directly and has Spring Security and HAPI FHIR, but requires more code and configuration and slower development for the team.
- **Node.js + NestJS** — no official EDS SDK and less experience in the team.

## Consequences

- (+) Fast development, automatic OpenAPI documentation, matches the experience of the backend lead.
- (+) Async request handling is sufficient for the expected load.
- (–) The system consists of two backend services in two languages (Python API and Java signature service), which must be built, deployed and tested together.
