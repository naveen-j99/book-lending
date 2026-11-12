<!-- sdlc:auto:start adr -->
# ADR-0002: Use FastAPI 0.115 for Backend

Status: Accepted  
Date: 2026-11-12

## Context and Problem Statement

Which Backend technology does the project use? Decided in architecture v4.

## Considered Options

- FastAPI 0.115
- Django REST Framework
- Flask

## Decision Outcome

Chosen option: FastAPI 0.115, because Approved stack. Expresses the server-side rules (5-loan cap, zero-copy block, 14-day due date, admin-only returns) and auto-generates the OpenAPI docs required by US-018.

## Consequences

Business invariants and session handling are hand-written rather than framework-provided, which is appropriate at this small scope.
<!-- sdlc:auto:end adr -->