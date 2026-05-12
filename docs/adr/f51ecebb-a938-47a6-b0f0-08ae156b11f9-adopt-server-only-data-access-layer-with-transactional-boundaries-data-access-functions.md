# Adopt Server-Only Data Access Layer with Transactional Boundaries: Data Access Functions

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all server-side data access operations. All data mutations and queries that interact with the database MUST follow the patterns defined herein.

## Context

- The codebase exhibits a consistent pattern of isolating data access operations in a dedicated 'server-only' layer, separating business logic from client-facing code
- Multiple files demonstrate transactional data operations (delete-folder, delete-envelope-recipient, delete-api-token-by-id, set-avatar-image) that require atomic database interactions
- The pattern appears across different functional domains (folders, recipients, API tokens, profiles, security settings), indicating a system-wide architectural decision
- The 'server-only' namespace suggests explicit enforcement of server-side execution boundaries to prevent client-side data access vulnerabilities
- The pattern signature (f45f4b322b9b0df9c17437027526907d) was detected with 91.36% confidence across 5 files, indicating strong architectural consistency

## Problem Statement

Without a standardized data access pattern, applications risk inconsistent transaction handling, security vulnerabilities from client-side data access, scattered business logic, and difficulty maintaining data integrity across complex operations. The system needs a clear architectural boundary that ensures all database operations are executed server-side with proper transactional semantics and security controls.

## Decision

1. MAY: Data access functions MAY return typed results or throw typed exceptions to provide clear contracts for calling code

## Policy Block

- MAY Data access functions MAY return typed results or throw typed exceptions to provide clear contracts for calling code

In scope:
- All database queries and mutations across the application
- Server-side API endpoints that interact with persistent storage
- Business logic functions that require transactional data operations
- Authentication and authorization checks for data access
- Data validation and transformation logic before persistence

Out of scope:
- Client-side state management and UI rendering logic
- In-memory data structures and caching layers
- Third-party API integrations that do not touch the database
- Static file serving and asset management
- Frontend validation logic (which should complement, not replace, server-side validation)

Exceptions:
- EX-001: Read-only database queries for public data that requires no authentication may be executed in edge functions or middleware for performance optimization
- EX-002: Database migrations and schema management tools may access the database directly outside the server-only layer

## Rationale

- The pattern was detected across 5 distinct files with 91.36% confidence, demonstrating consistent architectural implementation across multiple domains (folders, recipients, API tokens, profiles, security)
- Centralizing data access in a server-only layer provides a single enforcement point for security policies, transaction management, and data validation
- The pattern aligns with modern web security best practices by preventing client-side code from directly accessing or manipulating database state
- Organizing data access by domain entity (as evidenced by the file structure) improves code maintainability and enables clear ownership boundaries for different parts of the system

## Consequences

Positive:
- Enhanced security posture by eliminating direct client-to-database access paths and enforcing server-side authorization
- Improved data consistency through centralized transaction management and validation logic
- Better code organization and maintainability with clear separation between data access, business logic, and presentation layers
- Easier testing and mocking of data access operations through well-defined service interfaces
- Simplified auditing and monitoring of all database operations through a single architectural layer

Negative:
- Increased latency for data operations due to additional network round-trips between client and server
- Higher server resource utilization as all database operations must execute server-side
- Additional boilerplate code required to create and maintain server-only service functions for each data operation
- Potential performance bottlenecks if server-only layer is not properly scaled or optimized
- Learning curve for developers unfamiliar with strict server-client separation patterns

## Alternatives

- Direct client-side database access using client SDKs with row-level security policies (rejected)
  Rejected because: Exposes database connection details to clients, increases attack surface, and makes it difficult to enforce complex business logic and transaction boundaries consistently
  When valid: May be appropriate for simple CRUD applications with minimal business logic and strong database-level security policies
- GraphQL API with automatic query generation and resolver-based data access (rejected)
  Rejected because: While GraphQL provides good client flexibility, the detected pattern shows explicit service functions for specific operations, suggesting a preference for controlled, operation-specific APIs over flexible query languages
  When valid: Could be adopted for read-heavy workloads where clients need flexible data fetching without compromising the server-only execution boundary
- Repository pattern with ORM abstraction layer (deferred)
  Rejected because: Not explicitly rejected; the current pattern may already implement a repository-like pattern within the server-only layer
  When valid: Could be formalized as an enhancement to provide additional abstraction and testability without changing the core server-only architecture

## Risks

- Developers may bypass the server-only layer by creating direct database connections in client-side code or API routes, undermining the architectural pattern
  Mitigation: Implement linting rules and build-time checks to detect imports of database clients outside the server-only directory; conduct code reviews focused on data access patterns
  Owner: Engineering team and security team
- Performance degradation for data-intensive operations due to server round-trips and lack of client-side caching
  Mitigation: Implement strategic caching at the API layer; use batch operations and optimized queries; monitor performance metrics and optimize hot paths
  Owner: Engineering team and performance engineering
- Inconsistent error handling and transaction management across different server-only modules may lead to data integrity issues
  Mitigation: Establish shared utilities and patterns for transaction management; create comprehensive testing for error scenarios; document standard patterns in developer guidelines
  Owner: Engineering team and architecture team

## Implementation Notes

- Organize server-only data access functions by domain entity in a clear directory structure (e.g., server-only/folder/, server-only/recipient/, server-only/profile/)
- Use TypeScript to define clear input/output types for each data access function, enabling compile-time validation and better IDE support
- Implement a consistent error handling strategy across all server-only functions, using typed exceptions or result types to communicate failures
- Consider creating shared utilities for common patterns like transaction management, authorization checks, and input validation to reduce boilerplate
- Document the contract for each server-only function including preconditions, postconditions, and side effects to aid in testing and maintenance

## Continuation Context


Verify commands:
- grep -r 'import.*prisma\|import.*db' apps/*/app --exclude-dir=node_modules | grep -v 'server-only' | grep -v '.server.' || echo 'No client-side database imports detected'
- find packages/lib/server-only -type f -name '*.ts' | wc -l | awk '{if($1 >= 5) print "PASS: Found " $1 " server-only modules"; else print "FAIL: Expected at least 5 server-only modules"}'
- grep -r 'export.*function.*delete\|export.*function.*create\|export.*function.*update' packages/lib/server-only --include='*.ts' | wc -l | awk '{if($1 >= 3) print "PASS: Found " $1 " data mutation functions"; else print "FAIL: Expected at least 3 mutation functions"}'

Accept when:
- No client-side code directly imports database clients or executes queries; all database access is mediated through server-only layer
- At least 5 server-only data access modules exist, organized by domain entity, each handling specific business operations
- All data mutation operations (create, update, delete) are implemented as exported functions in the server-only layer with proper error handling

## Enforcement

- Verified by: Automated linting rules that flag database client imports outside the server-only directory
- Verified by: Code review checklist requiring verification that new data access code follows the server-only pattern
- Verified by: CI/CD pipeline checks running the verify commands to detect pattern violations before merge
- Verified by: Architecture review for new features that introduce data access patterns
- Violation handling: Build failures for linting violations that detect client-side database access
- Violation handling: Automated PR comments flagging potential violations for reviewer attention
- Violation handling: Required architecture review and remediation plan for merged code that violates the pattern
- Violation handling: Quarterly architecture audits to identify and address systematic violations
- Exception process: Submit exception request to architecture team with detailed justification, security analysis, and performance requirements
- Exception process: Security team review required for any exception involving data access outside server-only layer
- Exception process: Approved exceptions must be documented in ADR addendum with specific scope, duration, and review date
- Exception process: All exceptions subject to quarterly review and must be renewed or remediated