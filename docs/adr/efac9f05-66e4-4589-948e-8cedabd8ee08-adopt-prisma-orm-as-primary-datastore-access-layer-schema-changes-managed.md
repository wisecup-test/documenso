# Adopt Prisma ORM as Primary Datastore Access Layer: Schema Changes Managed

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all database access patterns in tRPC server routers and data access layers. All new database queries MUST follow the patterns established herein.

## Context

- The application requires a type-safe, maintainable approach to database access across multiple tRPC router endpoints including envelope management, webhook handling, and attachment operations
- Database queries are distributed across server-side router implementations that need consistent patterns for querying, filtering, and data retrieval
- The codebase demonstrates a pattern of using Prisma Client for database operations in tRPC server routers, particularly in envelope-router and webhook-router modules
- Type safety between database schema and application code is critical for preventing runtime errors in production API endpoints
- The pattern appears in 3 distinct files with 91.20% confidence, indicating a deliberate architectural choice rather than ad-hoc implementation

## Problem Statement

Without a standardized ORM layer, database access patterns become inconsistent across the application, leading to type safety issues, query duplication, migration challenges, and increased maintenance burden. The application needs a unified approach to primary datastore access that provides type safety, query composition, and schema management.

## Decision

1. MUST: Schema changes MUST be managed through Prisma migrations rather than manual database alterations

## Policy Block

- MUST Schema changes MUST be managed through Prisma migrations rather than manual database alterations

In scope:
- All tRPC server router implementations (envelope-router, webhook-router, etc.)
- Database queries for attachments, envelope fields, webhooks, and related entities
- CRUD operations on primary application data models
- Schema migrations and database structure changes
- Type generation for database entities used in API responses

Out of scope:
- Analytics queries that require complex aggregations beyond Prisma's capabilities
- Legacy database systems not supported by Prisma
- Read-only reporting databases or data warehouses
- Caching layers (Redis, Memcached) used for performance optimization
- External API integrations that don't involve the primary datastore

Exceptions:
- EXC-001: Performance profiling demonstrates that Prisma query builder introduces unacceptable latency (>100ms overhead) for specific high-traffic endpoints
- EXC-002: Database-specific features (e.g., PostgreSQL full-text search, advanced JSON operations) are not supported by Prisma's query builder

## Rationale

- Pattern detection shows consistent usage across 3 files in critical data access paths (attachments, webhooks, envelope fields) with 91.20% confidence, indicating this is an established architectural standard
- Prisma provides end-to-end type safety from database schema to TypeScript types, eliminating an entire class of runtime errors in tRPC endpoints
- The ORM layer abstracts database-specific SQL dialects, enabling potential database migrations without rewriting query logic throughout the application
- Prisma's migration system provides version-controlled schema evolution, critical for team collaboration and deployment safety

## Consequences

Positive:
- Type safety across the entire data access layer prevents runtime type errors and improves developer productivity with autocomplete
- Consistent query patterns across all routers reduce cognitive load and make code reviews more efficient
- Automated migration generation and tracking simplifies database schema evolution and deployment processes
- Prisma Studio provides built-in database browsing and debugging capabilities for development workflows

Negative:
- Prisma adds an abstraction layer that may introduce performance overhead for extremely high-throughput queries
- Complex analytical queries or database-specific features may require dropping down to raw SQL, creating inconsistency
- Team members must learn Prisma's query API and migration workflow, adding onboarding complexity
- Prisma Client generation step adds build time and requires regeneration after schema changes

## Alternatives

- Use TypeORM as the primary ORM layer (rejected)
  Rejected because: TypeORM's decorator-based approach is less aligned with functional programming patterns used in tRPC routers, and type safety is not as comprehensive as Prisma's generated client
  When valid: Could be reconsidered if the application migrates away from tRPC or requires ActiveRecord-style patterns
- Use raw SQL with a query builder like Kysely (rejected)
  Rejected because: While Kysely provides excellent type safety, it requires manual schema type definitions and lacks the integrated migration system that Prisma provides
  When valid: Appropriate for microservices that need minimal dependencies or maximum query control
- Use Drizzle ORM for lighter-weight type-safe queries (deferred)
  Rejected because: Drizzle is a newer ORM with a smaller ecosystem; existing codebase has already standardized on Prisma
  When valid: Could be evaluated for new microservices or if Prisma performance becomes a bottleneck across the application

## Risks

- Prisma Client generation failures during CI/CD could block deployments if schema and migrations are out of sync
  Mitigation: Implement pre-commit hooks to validate schema changes and ensure migrations are generated; add CI checks that verify Prisma Client can be generated successfully
  Owner: DevOps and Backend Engineering Team
- Performance degradation on complex queries with deep relation loading could impact API response times
  Mitigation: Establish performance budgets for API endpoints; profile Prisma queries in staging; use dataloader pattern or raw queries for identified bottlenecks
  Owner: Backend Engineering Team
- Prisma version upgrades may introduce breaking changes in query API or migration format
  Mitigation: Pin Prisma versions in package.json; test upgrades in isolated branches; maintain comprehensive integration test coverage for data access patterns
  Owner: Backend Engineering Team

## Implementation Notes

- Initialize Prisma Client as a singleton in the tRPC context to avoid connection pool exhaustion; ensure proper cleanup in serverless environments
- Use Prisma's 'include' and 'select' options strategically to fetch only required relations and fields, optimizing query performance
- Leverage Prisma's middleware feature for cross-cutting concerns like soft deletes, audit logging, or tenant isolation
- Generate Prisma Client after schema changes with 'npx prisma generate' and commit the generated types to version control if using a monorepo structure
- Document complex queries with comments explaining the business logic, especially when using nested writes or transaction blocks

## Continuation Context


Verify commands:
- grep -r 'from @prisma/client' packages/trpc/server --include='*.ts' | wc -l
- grep -r 'prisma\.' packages/trpc/server --include='*.ts' | grep -v 'node_modules' | wc -l
- find packages/trpc/server -name '*.ts' -exec grep -l 'pg\|mysql\|sqlite' {} \; | grep -v prisma | wc -l

Accept when:
- All tRPC router files import PrismaClient and use it for database operations (verify command 1 shows imports in router files)
- Database queries use Prisma's query builder methods (findMany, findUnique, create, update, delete) rather than raw SQL (verify command 2 shows prisma method calls)
- No direct database driver imports (pg, mysql2, sqlite3) exist outside of Prisma configuration files (verify command 3 returns 0 or only shows Prisma-related files)

## Enforcement

- Verified by: Automated code review checks in CI/CD pipeline scanning for direct database driver imports outside Prisma
- Verified by: Pull request reviews by backend team members checking for Prisma usage patterns
- Verified by: Static analysis tools configured to flag raw SQL strings in router implementations
- Verified by: Integration tests that verify database operations use Prisma Client
- Violation handling: CI pipeline fails if direct database driver imports are detected in router files
- Violation handling: Pull requests with non-Prisma database access patterns are blocked until refactored or exception is approved
- Violation handling: Quarterly architecture reviews identify and remediate any violations that bypassed checks
- Violation handling: Violations discovered in production require immediate ticket creation and prioritization for refactoring
- Exception process: Developer documents performance issue or Prisma limitation with specific metrics and use case details
- Exception process: Tech lead reviews exception request and validates that Prisma cannot meet requirements
- Exception process: Architecture team approves exception with documented rationale in ADR or code comments
- Exception process: Exception is tracked in technical debt register with plan for future resolution if applicable