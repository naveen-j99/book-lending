<!-- sdlc:auto:start adr -->
# ADR-0001: Use React 19 for Frontend

Status: Accepted  
Date: 2026-11-12

## Context and Problem Statement

Which Frontend technology does the project use? Decided in architecture v4.

## Considered Options

- React 19
- Vue
- SvelteKit

## Decision Outcome

Chosen option: React 19, because Approved stack. Delivers the two responsive views (member browse/borrow/dashboard and admin catalog/loans) as a single-page app that stores the bearer token and calls the API across origins.

## Consequences

Needs a client-side router and an explicit auth header on each request since no shared-origin cookie is available.
<!-- sdlc:auto:end adr -->