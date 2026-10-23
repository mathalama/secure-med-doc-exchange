# MedExchange — Secure Medical Document Exchange

Development of a secure system for the exchange of digital documents between medical organizations.


> KZ: Медициналық ұйымдар арасында цифрлық құжаттармен алмасудың қауіпсіз жүйесін әзірлеу
> RU: Разработка защищённой системы обмена цифровыми документами между медицинскими организациями

## Problem statement

_To be completed in [DIP-2](https://medexchange-diploma.youtrack.cloud/issue/DIP-2)._

## Team

| Member | GitHub | Role |
|---|---|---|
| Sagyntai Aikyn | [@mathalama](https://github.com/mathalama) | Backend Lead, Team Lead |
| Korazbay Alikhan | [@AlikShal](https://github.com/AlikShal) | Security Analyst & QA Engineer |
| Almasuly Damir | [@smuzyyq](https://github.com/smuzyyq) | Frontend Developer & UX Designer |

**Supervisor:** Bakiyeva Aigerim Muratovna, Assistant Professor

## Project status

Analysis and design stage (pre-defense 1).

## Repository structure

| Path | Content |
|---|---|
| `apps/web/` | Web client |
| `services/api/` | Backend API service and its unit/integration tests |
| `tests/` | End-to-end and security tests |
| `infra/` | Docker and deployment configuration |
| `docs/` | Proposal, literature review, comparisons, requirements, architecture, plans |
| `docs/adr/` | Architecture decision records |
| `docs/security/` | Threat model and security analysis |
| `diagrams/src/` | Editable diagram sources |
| `diagrams/export/` | Diagram exports (PNG) |
| `prototype/` | Prototype screenshots |

## Links

- YouTrack: https://medexchange-diploma.youtrack.cloud/issues/DIP
- Documentation: [docs/](docs/)

## Workflow

- `main` is the reviewed baseline; changes go through pull requests.
- Branch name: `DIP-<id>-short-description`, e.g. `DIP-2-project-proposal`.
- Commit message: `DIP-<id>: <description>`.
