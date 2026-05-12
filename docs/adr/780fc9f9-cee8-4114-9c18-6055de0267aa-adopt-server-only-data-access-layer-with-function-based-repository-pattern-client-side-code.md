# Adopt Server-Only Data Access Layer with Function-Based Repository Pattern: Client Side Code

Status: proposed
Date: 2025-01-10
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all server-side data access operations. All database queries and data mutations MUST be implemented through the server-only data access layer.

## Context

- The codebase requires a clear separation between client-side and server-side data access to prevent accidental exposure of database credentials and sensitive operations in client bundles
- Multiple domain entities (fields, folders, API tokens) require consistent CRUD operations with similar patterns for querying, updating, and deleting records
- The application uses a server-only module pattern to ensure database access code is never bundled for client-side execution, improving security and reducing bundle size
- Public API contracts need to be enforced through a well-defined data access layer that provides type-safe interfaces and consistent error handling
- The pattern emerged across 8 files with 91.74% confidence, indicating a deliberate architectural choice for organizing data access operations

## Problem Statement

Without a standardized server-only data access layer, database operations risk being exposed to client bundles, leading to security vulnerabilities and inconsistent data access patterns across the application. Teams need a clear, enforceable pattern for implementing CRUD operations that guarantees server-side execution while maintaining type safety and API contract consistency.

## Decision

1. MUST_NOT: Client-side code MUST NOT directly import or execute database queries; all data access MUST go through the server-only layer

## Policy Block

- MUST_NOT Client-side code MUST NOT directly import or execute database queries; all data access MUST go through the server-only layer

In scope:
- All database queries and mutations for application data
- CRUD operations for domain entities (fields, folders, API tokens, etc.)
- Server-side API route handlers and server components
- Background jobs and scheduled tasks requiring data access
- Authentication and authorization checks requiring database queries

Out of scope:
- Client-side state management and UI logic
- Static data and configuration that doesn't require database access
- Third-party API integrations that don't involve local database operations
- Pure utility functions and business logic that operate on in-memory data
- Frontend data fetching hooks (which should call server endpoints, not database directly)

Exceptions:
- EXC-001: Database migration scripts and seed data operations that run in isolated contexts
- EXC-002: Development and testing utilities that require direct database access for setup/teardown

## Rationale

- The pattern provides strong security guarantees by ensuring database credentials and queries never reach client bundles, reducing attack surface and preventing SQL injection vectors in client code
- Function-based repositories with single-file-per-operation organization improve code discoverability, testability, and maintainability compared to monolithic repository classes
- Consistent naming conventions and directory structure reduce cognitive load for developers and enable automated tooling for pattern detection and enforcement
- The server-only boundary aligns with modern framework patterns (Next.js server components, server actions) and enables better code splitting and bundle optimization

## Consequences

Positive:
- Enhanced security posture with guaranteed server-side execution of all database operations, preventing credential exposure and client-side data manipulation
- Improved code organization with clear separation of concerns and predictable file locations for data access operations
- Better developer experience through consistent patterns, easier code navigation, and reduced decision fatigue when implementing new data access functions
- Smaller client bundle sizes by excluding all database access code from client-side JavaScript bundles
- Easier testing and mocking of data access operations due to isolated, single-purpose functions

Negative:
- Additional indirection layer may increase initial development time for simple CRUD operations compared to direct database access
- Requires discipline to maintain consistency across the codebase and prevent pattern drift as new developers join the team
- May lead to proliferation of small files, potentially making repository navigation more complex without proper tooling
- Server-only boundary requires careful consideration of data serialization and may complicate patterns like optimistic updates

## Alternatives

- Class-based repository pattern with methods for all CRUD operations in a single repository class per entity (rejected)
  Rejected because: Class-based repositories create larger files that are harder to navigate and test, and don't align well with tree-shaking and modern module bundling. The function-based approach provides better code splitting and clearer dependencies.
  When valid: May be reconsidered for entities with complex, interdependent operations that benefit from shared state or transaction management across multiple operations
- ORM-based data access with models containing both data structure and query methods (Active Record pattern) (rejected)
  Rejected because: Active Record pattern couples data structure with data access logic, making it harder to enforce server-only boundaries and potentially leading to accidental client-side imports of database logic
  When valid: Could be used in conjunction with this pattern where ORM models are kept in server-only layer and wrapped by function-based access layer
- GraphQL or tRPC with automatic resolver generation for all database operations (deferred)
  Rejected because: Not rejected but deferred for evaluation. Could complement this pattern by providing the API layer on top of the server-only data access functions
  When valid: Can be adopted as the API transport layer while maintaining the server-only data access pattern as the underlying implementation

## Risks

- Pattern drift over time as developers create inconsistent naming conventions or bypass the server-only layer for perceived convenience
  Mitigation: Implement automated linting rules and CI checks to detect violations. Provide clear documentation and code templates. Conduct regular code reviews focusing on data access patterns.
  Owner: Engineering team leads and architecture review board
- Performance issues from excessive function calls and lack of query optimization when operations are too granular
  Mitigation: Allow for batch operations and internal variants (e.g., find-folders-internal) that can optimize multiple queries. Monitor query performance and refactor hot paths as needed.
  Owner: Backend engineering team and database administrators
- Difficulty in maintaining transaction boundaries across multiple data access function calls
  Mitigation: Provide transaction helper utilities that can wrap multiple data access calls. Document transaction patterns and create higher-level service functions for complex multi-step operations.
  Owner: Backend architecture team

## Implementation Notes

- Create a template or code generator for new data access functions to ensure consistency in structure, error handling, and TypeScript typing
- Use module bundler configuration (e.g., Next.js 'server-only' package) to enforce server-side execution and fail builds if server-only modules are imported in client code
- Organize data access functions by domain entity in subdirectories (field/, folder/, etc.) and use index files to provide clean import paths
- Document common patterns for each operation type (get, find, update, delete) including parameter validation, error handling, and return type conventions
- Consider implementing a base query builder or database client wrapper that all data access functions use to ensure consistent connection handling and query logging

## Continuation Context


Verify commands:
- grep -r 'from.*server-only' --include='*.tsx' --include='*.ts' apps/*/app/**/page.tsx apps/*/app/**/layout.tsx | grep -v '.server.' || echo 'No client-side imports of server-only modules detected'
- find packages/lib/server-only -type f -name '*.ts' | xargs -I {} basename {} | grep -E '^(get|find|update|delete|remove)-.*\.ts$' | wc -l
- test -d packages/lib/server-only && find packages/lib/server-only -type f -name '*.ts' -exec grep -l 'export.*function' {} \; | wc -l

Accept when:
- All database access operations are located in the server-only directory structure with no direct database queries in client-side code
- Each data access function follows the naming convention (get-*, find-*, update-*, delete-*, remove-*) and is isolated in its own file
- Build process successfully prevents client-side imports of server-only modules and fails with clear error messages when violations are detected
- Code review checklist includes verification that new data access operations follow the established pattern

## Enforcement

- Verified by: Automated ESLint rules checking for server-only module imports in client code
- Verified by: CI pipeline verification commands that scan for pattern compliance
- Verified by: Build-time enforcement through bundler configuration (Next.js 'server-only' package)
- Verified by: Code review checklist items for data access pattern compliance
- Verified by: Periodic architecture audits using pattern detection tools
- Violation handling: Build failures for client-side imports of server-only modules (blocking)
- Violation handling: ESLint errors for pattern violations that must be resolved before merge
- Violation handling: Code review rejection for data access operations that don't follow the established pattern
- Violation handling: Automated comments on pull requests identifying potential violations with links to documentation
- Exception process: Developer identifies legitimate need for exception and documents rationale in code comments
- Exception process: Exception request submitted to architecture review board or tech lead with justification
- Exception process: If approved, exception is documented in ADR exceptions section and added to linting ignore rules with explanation
- Exception process: Exceptions are reviewed quarterly to determine if they represent new patterns that should be formalized or can be refactored to comply