# ADR 0003: Component interaction — REST API over HTTPS

- **Status:** Accepted
- **Date:** 2026-10-19
- **Task:** [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2)

## Context

The web client and, in the future, external systems of medical organizations interact with the backend. The interaction must work from a browser, be compatible with HL7 FHIR, be secure and easy to test. Detailed comparison: [05-technology-comparison.md](../05-technology-comparison.md#decision-4-component-interaction).

## Decision

Use a **REST API over HTTPS with JSON**, described with OpenAPI. Authentication uses JWT access tokens; authorization is role-based (RBAC) and checked on every request. The main API calls the signature service (ADR 0005) over an internal REST interface that is not exposed outside the Docker network. Background tasks (notifications) run inside the backend. The browser communicates with NCALayer through its local WebSocket API.

## Alternatives considered

- **gRPC** — efficient, but not supported directly by browsers and not compatible with the FHIR REST API.
- **Message broker (RabbitMQ)** — good for asynchronous delivery, but needs a gateway for the browser and adds infrastructure that the current load does not require.

## Consequences

- (+) Compatible with FHIR, works directly from the browser, universal for external integrations.
- (+) Easy to test with Postman and automated API tests; the OpenAPI specification serves as the API contract for the frontend.
- (–) Delivery retries and notifications must be implemented in the backend; a broker can be introduced later if needed.
