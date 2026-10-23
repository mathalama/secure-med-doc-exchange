# ADR 0002: Data storage — PostgreSQL + SeaweedFS

- **Status:** Accepted (revised 2026-10-23: MinIO replaced by SeaweedFS)
- **Date:** 2026-10-19
- **Task:** [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2)

## Context

The system stores structured data (organizations, users, roles, document metadata, statuses, audit log) and files (document contents and CMS signatures). Structured data must stay consistent: a status change and its audit record must be saved together. Files contain medical data and must be stored encrypted. MinIO, the initial candidate for file storage, is no longer maintained as a community edition (no official Docker images since October 2025, maintenance mode since December 2025, repository archived in 2026). Detailed comparison: [05-technology-comparison.md](../05-technology-comparison.md#decision-3-data-storage).

## Decision

- **PostgreSQL** for structured data. FHIR resources are stored in `JSONB` columns. The audit log table is append-only (UPDATE and DELETE are revoked for the application role).
- **SeaweedFS** (S3 API) for files. Files are encrypted by the application with AES-256-GCM before upload. The database stores the object key and the SHA-256 hash of each file.

## Alternatives considered

- **MongoDB** — convenient for JSON, but weaker relational integrity and transactions.
- **MySQL** — relational, but weaker JSON indexing and fewer security features than PostgreSQL.
- **MinIO** — excluded: no longer maintained, no security fixes.
- **Garage** — S3-compatible and simple, but smaller community and AGPL-3.0; kept as a fallback.
- **Files in PostgreSQL (`bytea`)** — simple, but large files bloat the database and backups.

## Consequences

- (+) Transactions and foreign keys guarantee consistency of statuses and audit.
- (+) Confidentiality of files does not depend on the storage product (application-level encryption); integrity is checked by the stored hash.
- (+) Access through the S3 API allows replacing SeaweedFS with Garage or cloud S3 without code changes.
- (–) Encryption keys must be managed by the application (stored outside the database, configured through environment variables in the prototype).
- (–) One more container to deploy and back up.
