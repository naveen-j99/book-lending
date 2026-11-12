<!-- sdlc:auto:start adr -->
# ADR-0003: Use PostgreSQL 16 for Database

Status: Accepted  
Date: 2026-11-12

## Context and Problem Statement

Which Database technology does the project use? Decided in architecture v4.

## Considered Options

- PostgreSQL 16
- MySQL
- SQLite

## Decision Outcome

Chosen option: PostgreSQL 16, because Approved stack. Enforces the Users/Books/Loans relationships and uses SELECT ... FOR UPDATE row locking so concurrent borrows cannot take the last copy (NFR-006).

## Consequences

A relational store is heavier than a file DB but is required for safe concurrency and thousands-scale growth (NFR-005).
<!-- sdlc:auto:end adr -->