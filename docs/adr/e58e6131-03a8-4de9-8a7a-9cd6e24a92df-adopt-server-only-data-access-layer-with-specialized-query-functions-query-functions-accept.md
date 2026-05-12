# Adopt Server-Only Data Access Layer with Specialized Query Functions: Query Functions Accept

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all server-side data access operations. All database queries MUST be encapsulated in dedicated server-only modules following the established pattern.

## Context

- The codebase requires a clear separation between client-side and server-side data access to prevent accidental exposure of database credentials and sensitive query logic
- Multiple domain entities (fields, folders, API tokens) require consistent CRUD operations with proper authorization and validation
- The application uses a server-only module pattern to enforce compile-time guarantees that database access code cannot be bundled into client-side JavaScript
- Query operations need to be composable and reusable across different API endpoints and internal services
- The pattern emerged organically across 8 files with 91.74% consistency, indicating strong architectural convergence

## Problem Statement

Without a standardized data access layer, database queries become scattered throughout the application, leading to inconsistent error handling, authorization bypasses, code duplication, and potential security vulnerabilities from client-side exposure of database logic. The system needs a unified approach to encapsulate all data access operations in a secure, maintainable, and testable manner.

## Decision

1. MUST: Query functions MUST accept authorization context (user ID, token, permissions) as parameters and enforce access control within the function

## Policy Block

- MUST Query functions MUST accept authorization context (user ID, token, permissions) as parameters and enforce access control within the function

In scope:
- All database read operations (SELECT queries)
- All database write operations (INSERT, UPDATE, DELETE)
- All data access requiring authentication or authorization
- Query operations for entities: fields, folders, API tokens, and similar domain objects
- Internal service-to-service data access within the server environment

Out of scope:
- Client-side data fetching via API endpoints (these call the server-only functions)
- Static data or configuration that doesn't require database access
- In-memory caching layers (though they may consume server-only functions)
- Third-party API integrations that don't touch the primary database

Exceptions:
- EXC-001: Database migration scripts that require direct database access for schema modifications
- EXC-002: Emergency hotfix scripts for production data repair under incident response

## Rationale

- The pattern was detected across 8 files with 91.74% confidence, demonstrating strong architectural consistency and team alignment on this approach
- Server-only modules provide compile-time guarantees against accidental client-side bundling, preventing credential exposure and reducing bundle size
- Specialized query functions create clear boundaries for testing, monitoring, and performance optimization at the data access layer
- The naming convention (verb-entity-qualifier) provides immediate clarity about function purpose and makes the codebase more navigable for new developers

## Consequences

Positive:
- Enhanced security through enforced separation of server-side database logic from client-side code
- Improved code maintainability with centralized, single-purpose data access functions that are easy to locate and modify
- Better testability through isolated functions that can be mocked or tested against test databases
- Consistent error handling and authorization patterns across all data access operations
- Reduced code duplication as common query patterns are encapsulated in reusable functions

Negative:
- Increased number of files in the codebase as each operation gets its own module
- Potential for over-abstraction if simple queries are wrapped in unnecessary layers
- Learning curve for developers unfamiliar with the server-only module pattern
- May require additional tooling or build configuration to enforce server-only boundaries

## Alternatives

- Use a single repository class per entity with all CRUD methods (rejected)
  Rejected because: Large repository classes become difficult to maintain and test, violating single responsibility principle. The detected pattern shows preference for granular, focused functions.
  When valid: May be appropriate for very simple entities with only 2-3 operations total
- Allow direct database access from API route handlers (rejected)
  Rejected because: Creates tight coupling between HTTP layer and data layer, makes testing difficult, and increases risk of security vulnerabilities through scattered authorization logic
  When valid: Never valid in production code; only acceptable in prototypes or proof-of-concept code
- Use a GraphQL or ORM auto-generated query layer (deferred)
  Rejected because: Not rejected but not currently adopted. May be evaluated in future for specific use cases requiring dynamic query composition.
  When valid: Could be valid for admin panels or internal tools where query flexibility is prioritized over performance optimization

## Risks

- Proliferation of similar query functions leading to maintenance burden and inconsistency
  Mitigation: Establish code review guidelines to identify opportunities for consolidation. Create shared query utilities for common patterns (pagination, filtering, sorting).
  Owner: Engineering team leads
- Performance issues from N+1 queries if functions are composed naively without considering query optimization
  Mitigation: Implement query monitoring and alerting. Provide internal variants of functions that accept pre-loaded data to enable batch operations.
  Owner: Database performance team
- Inconsistent error handling across different data access functions leading to poor user experience
  Mitigation: Create standardized error types and handling utilities. Document error handling patterns in developer guidelines.
  Owner: Platform engineering team

## Implementation Notes

- Use TypeScript's module resolution to enforce server-only imports. Configure bundler (webpack/vite) to error on server-only imports in client code.
- Establish naming conventions: get-* for single record retrieval, find-* for queries returning multiple records, create-*, update-*, delete-* for mutations, *-internal for trusted system operations.
- Each function should export a single default or named function with clear TypeScript types for parameters and return values.
- Include JSDoc comments documenting authorization requirements, expected errors, and usage examples.
- Consider using a shared database client/connection pool that is imported by all data access functions to ensure consistent configuration.

## Continuation Context


Verify commands:
- grep -r 'server-only' --include='*.ts' --include='*.tsx' | grep -v 'node_modules' | wc -l
- find . -path '*/server-only/*' -name '*.ts' | xargs grep -L 'export.*function' | wc -l
- grep -r 'import.*prisma\|import.*db' --include='*.tsx' --exclude-dir='server-only' --exclude-dir='api' | grep -v 'node_modules' | wc -l

Accept when:
- All database access operations are located in server-only directories with no direct database imports in client-side code
- Each data access function follows the verb-entity-qualifier naming pattern and exports a single focused operation
- Code review confirms no database credentials or connection strings are exposed outside the server-only layer
- Automated tests verify that client bundles do not include server-only modules

## Enforcement

- Verified by: Automated linting rules that prevent server-only imports in client-side code
- Verified by: Code review checklist requiring verification of data access layer usage
- Verified by: CI/CD pipeline checks for bundle analysis to detect server-only code in client bundles
- Verified by: Periodic architecture audits reviewing adherence to the pattern
- Violation handling: Build failures for any client-side imports of server-only modules
- Violation handling: Code review rejection for direct database access outside server-only layer
- Violation handling: Security review required for any exceptions to the pattern
- Violation handling: Refactoring tickets created for legacy code that violates the pattern
- Exception process: Developer submits exception request with technical justification to architecture review board
- Exception process: Architecture review board evaluates security implications and maintenance impact
- Exception process: If approved, exception is documented in ADR amendments with expiration date
- Exception process: Exceptions are reviewed quarterly and must be re-justified or remediated