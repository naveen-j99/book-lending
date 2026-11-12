<!-- sdlc:auto:start adr -->
# ADR-0004: Use AWS for Cloud

Status: Accepted  
Date: 2026-11-12

## Context and Problem Statement

Which Cloud technology does the project use? Decided in architecture v4.

## Considered Options

- AWS
- GCP
- Azure

## Decision Outcome

Chosen option: AWS, because Approved stack. RDS, Fargate and CloudFront give always-on hosting for the SPA, API, worker and database in one place (NFR-008), scaling from launch to thousands of users.

## Consequences

More moving parts than a single PaaS, accepted for reliability and headroom.
<!-- sdlc:auto:end adr -->