# Standardize tRPC Router Pattern for Primary Datastore Access: Router Procedures Handle

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all tRPC router implementations that interact with primary datastores. All new routers and router modifications MUST comply with the patterns defined herein.

## Context

- The codebase uses tRPC as the primary API framework for type-safe client-server communication, requiring consistent patterns for datastore access
- Multiple router implementations (envelope-router, webhook-router) access primary datastores through similar patterns, indicating an emergent architectural standard
- Pattern detected across 3 files with 91.20% confidence suggests this is an established practice rather than an isolated implementation
- The api.public.contracts facet indicates these routers expose public-facing APIs that require stable, consistent datastore interaction patterns
- Standardizing datastore access patterns in tRPC routers ensures type safety, maintainability, and consistent error handling across the API surface

## Problem Statement

Without a standardized approach to accessing primary datastores through tRPC routers, the codebase risks inconsistent data access patterns, duplicated logic, varying error handling strategies, and potential type safety gaps. This creates maintenance burden and increases the likelihood of bugs when different teams implement similar datastore operations.

## Decision

1. MUST: Router procedures MUST handle datastore errors consistently and return appropriate tRPC error codes (NOT_FOUND, INTERNAL_SERVER_ERROR, etc.)

## Policy Block

- MUST Router procedures MUST handle datastore errors consistently and return appropriate tRPC error codes (NOT_FOUND, INTERNAL_SERVER_ERROR, etc.)

In scope:
- All tRPC router procedures that perform read operations on primary datastores
- All tRPC router procedures that perform write operations (create, update, delete) on primary datastores
- Router implementations in the envelope-router, webhook-router, and similar API boundary modules
- Service layer functions called directly from tRPC router handlers

Out of scope:
- Internal service-to-service communication that does not use tRPC
- Background job processors that access datastores outside the request-response cycle
- Database migration scripts and schema management tools
- Direct database access in test fixtures and seed data scripts

Exceptions:
- EXC-001: Performance-critical operations require inline datastore access to minimize function call overhead
- EXC-002: Prototype or experimental features in feature-flagged code paths

## Rationale

- Pattern detected across 3 files (find-attachments, webhook-router, get-envelope-field) with 91.20% confidence indicates this is an established, successful pattern in the codebase
- tRPC's type-safety guarantees are maximized when datastore access follows consistent patterns, enabling end-to-end type inference from database to client
- Separating datastore access into focused procedures improves testability, as individual operations can be tested in isolation with mocked datastores
- The api.public.contracts facet classification indicates these patterns are exposed to external consumers, making consistency and stability critical for API contract reliability

## Consequences

Positive:
- Improved type safety across the entire API surface, reducing runtime errors and improving developer experience
- Consistent error handling patterns make debugging easier and provide predictable behavior for API consumers
- Modular, focused procedures enable easier testing, refactoring, and maintenance of datastore access logic
- Clear separation of concerns between API layer (tRPC routers) and data access layer facilitates future architectural changes

Negative:
- Additional abstraction layers may introduce slight performance overhead compared to direct inline datastore access
- Developers must learn and follow the established pattern, increasing onboarding time for new team members
- Refactoring existing non-compliant code to match the standard pattern requires investment of engineering time
- Overly rigid adherence to the pattern may create unnecessary complexity for simple, one-off datastore operations

## Alternatives

- Allow inline datastore access directly within tRPC router procedure definitions without abstraction layers (rejected)
  Rejected because: Inline access reduces testability, creates tight coupling between API and data layers, and makes it difficult to enforce consistent error handling and type safety patterns
  When valid: Only for throwaway prototypes or one-time scripts that will not be maintained
- Use a generic CRUD router that automatically generates procedures for all datastore entities (rejected)
  Rejected because: Auto-generated CRUD operations lack business logic validation, expose internal data models directly, and provide insufficient control over authorization and data transformation
  When valid: For internal admin tools where direct database access is acceptable and business logic is minimal
- Implement a repository pattern with dependency injection for all datastore access (deferred)
  Rejected because: While providing excellent testability and abstraction, full repository pattern with DI may be over-engineering for current codebase size; can be adopted incrementally as complexity grows
  When valid: When the codebase scales to require multiple datastore implementations or complex transaction management across services

## Risks

- Existing non-compliant router implementations may continue to diverge from the standard pattern, creating inconsistency
  Mitigation: Implement linting rules and code review checklists to catch non-compliant patterns; create migration guide for refactoring existing code
  Owner: Engineering team leads
- Performance overhead from abstraction layers could impact high-throughput API endpoints
  Mitigation: Establish performance benchmarks for critical paths; allow exceptions for proven performance-critical operations with documented justification
  Owner: Platform engineering team
- Pattern may become outdated as tRPC evolves or if the team migrates to a different API framework
  Mitigation: Review ADR annually or when major framework updates occur; ensure abstraction layers are framework-agnostic where possible
  Owner: Architecture review board

## Implementation Notes

- Start by creating a reference implementation guide with examples from the detected pattern files (find-attachments, webhook-router, get-envelope-field)
- Establish a shared utilities module for common datastore error handling and type conversion functions used across routers
- Use TypeScript path aliases to keep import statements clean when accessing service layer functions from routers (e.g., @services/attachments)
- Consider implementing custom tRPC middleware for cross-cutting concerns like logging, metrics, and transaction management for datastore operations

## Continuation Context


Verify commands:
- grep -r 'export.*router' packages/trpc/server --include='*.ts' | xargs -I {} sh -c 'grep -l "prisma\|db\|datastore" {} || true'
- find packages/trpc/server -name '*.ts' -type f -exec grep -l 'procedure.*query\|procedure.*mutation' {} \; | xargs grep -L 'import.*from.*service\|import.*from.*repository' || true
- npm run type-check -- --noEmit packages/trpc/server

Accept when:
- All tRPC router files that access datastores follow the established directory structure pattern (router-name/entity/operation.ts)
- Type-checking passes without errors for all router implementations, confirming end-to-end type safety
- Code review checklist confirms datastore access is properly abstracted and error handling follows tRPC conventions
- No direct database client usage (e.g., prisma.model.findMany) appears inline within router procedure definitions

## Enforcement

- Verified by: Automated CI checks running grep patterns to detect non-compliant inline datastore access
- Verified by: TypeScript compilation enforcing type safety across router and datastore boundaries
- Verified by: Code review process with explicit checklist items for datastore access patterns
- Verified by: Periodic architecture audits reviewing router implementations for pattern compliance
- Violation handling: CI pipeline fails if grep patterns detect inline datastore access without proper abstraction
- Violation handling: Pull requests with non-compliant patterns are blocked until refactored or exception is approved
- Violation handling: Existing violations are tracked in technical debt backlog with prioritization based on API criticality
- Violation handling: Quarterly reviews identify and prioritize refactoring of non-compliant legacy code
- Exception process: Developer submits exception request with performance benchmarks or technical justification
- Exception process: Tech lead and architecture reviewer evaluate exception against policy criteria (EXC-001, EXC-002)
- Exception process: Approved exceptions are documented in code with comments linking to approval and rationale
- Exception process: All exceptions are reviewed quarterly to determine if they can be eliminated or if the pattern needs adjustment