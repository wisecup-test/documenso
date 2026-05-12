# Adopt Prisma ORM as Primary Datastore Access Layer: Raw Sql Queries

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all database access patterns in tRPC server routers and data access layers. All new database queries MUST follow the patterns established herein.

## Context

- The codebase contains multiple tRPC server routers that require database access for operations like finding attachments, managing webhooks, and retrieving envelope fields
- A consistent pattern of database interaction has been detected across 3 files with 91.20% confidence, indicating a standardized approach to data access
- The application architecture separates API layer (tRPC routers) from data access layer, requiring a robust ORM solution to bridge these concerns
- Type safety between TypeScript application code and database schema is critical for maintaining data integrity and developer productivity
- The detected pattern suggests use of Prisma ORM based on typical query patterns found in envelope-router and webhook-router implementations

## Problem Statement

The application requires a standardized, type-safe mechanism for accessing the primary datastore across multiple tRPC routers and service layers. Without a consistent ORM pattern, teams risk introducing inconsistent query patterns, losing type safety guarantees, and creating maintenance challenges as the schema evolves. The system needs a single source of truth for database schema that generates type-safe client code and provides a consistent query interface.

## Decision

1. MUST_NOT: Raw SQL queries MUST NOT be used except when Prisma's query capabilities are demonstrably insufficient for the use case

## Policy Block

- MUST_NOT Raw SQL queries MUST NOT be used except when Prisma's query capabilities are demonstrably insufficient for the use case

In scope:
- All tRPC router implementations requiring database access
- Service layer functions that perform CRUD operations
- Background jobs and scheduled tasks that interact with the primary datastore
- Data migration scripts and seed files
- API endpoints that query or mutate application data

Out of scope:
- Analytics queries against read replicas or data warehouses
- Legacy systems integration where raw SQL is required for compatibility
- Database administration scripts for operational maintenance
- Performance testing and benchmarking utilities
- Third-party service integrations with their own data access patterns

Exceptions:
- EXC-001: A specific query pattern requires database-specific features (e.g., PostgreSQL full-text search, JSON operators) not yet supported by Prisma's query builder
- EXC-002: Performance profiling demonstrates that a Prisma query is at least 3x slower than an equivalent optimized raw SQL query for a critical path operation

## Rationale

- Pattern detection across 3 files (find-attachments, webhook-router, get-envelope-field) with 91.20% confidence indicates this is an established architectural standard in the codebase
- Prisma provides compile-time type safety that prevents entire classes of runtime errors related to schema mismatches and type coercion
- Using a single ORM standard reduces cognitive load for developers moving between different parts of the codebase and enables code reuse
- Prisma's migration system provides version control for database schema changes, enabling safe deployments and rollbacks

## Consequences

Positive:
- Type safety across the entire data access layer eliminates runtime type errors and improves IDE autocomplete support
- Consistent query patterns across all routers reduce onboarding time for new developers and improve code maintainability
- Prisma's generated client automatically stays in sync with schema changes, preventing drift between application code and database structure
- Built-in connection pooling and query optimization in Prisma Client improve application performance without manual tuning

Negative:
- Teams must learn Prisma's query API and may face a learning curve if coming from raw SQL or other ORMs
- Some advanced database features may not be accessible through Prisma's query builder, requiring workarounds or raw queries
- Prisma adds an additional dependency and abstraction layer that could complicate debugging of database performance issues
- Generated Prisma Client increases build time and repository size, particularly for large schemas

## Alternatives

- Use TypeORM as the primary ORM solution (rejected)
  Rejected because: TypeORM uses decorators and class-based models which don't align with the functional programming patterns evident in tRPC routers. Prisma's generated client approach provides better type inference and requires less boilerplate.
  When valid: Consider for projects that heavily use class-based architecture or require ActiveRecord patterns
- Use raw SQL with a query builder like Kysely (rejected)
  Rejected because: While Kysely provides excellent type safety, it requires manual type definitions that can drift from actual schema. Prisma's schema-first approach with automatic type generation provides stronger guarantees.
  When valid: Valid for teams with strong SQL expertise who need maximum control over query generation
- Use Drizzle ORM for a lighter-weight alternative (rejected)
  Rejected because: Pattern detection shows Prisma is already established in the codebase with 91.20% confidence. Switching would require significant refactoring with minimal benefit.
  When valid: Consider for new greenfield projects where bundle size is a critical constraint

## Risks

- Prisma may not support all required database features, forcing use of raw SQL and undermining consistency
  Mitigation: Maintain a documented exception process for raw SQL usage. Regularly review Prisma release notes for new features that could replace raw SQL workarounds. Contribute to Prisma's open-source development for critical missing features.
  Owner: Engineering team / Database working group
- Performance issues with Prisma-generated queries could impact application responsiveness
  Mitigation: Establish performance benchmarks for critical queries. Use Prisma's query logging and APM integration to identify slow queries. Maintain exception process for performance-critical raw SQL when justified by data.
  Owner: Engineering team / Performance team
- Schema changes could break existing queries if not properly tested
  Mitigation: Implement comprehensive integration tests for all database access patterns. Use Prisma's migration preview feature to validate changes. Require database migration review in pull request process.
  Owner: Engineering team / QA team

## Implementation Notes

- Initialize Prisma in the project with 'npx prisma init' and define the schema.prisma file with all models, relations, and indexes
- Configure Prisma Client generation in the build pipeline to run after schema changes and before TypeScript compilation
- Create a shared database context or service that instantiates and exports a singleton Prisma Client instance for use across all routers
- Establish naming conventions for Prisma models that align with existing database table names to minimize migration effort
- Document common query patterns (filtering, pagination, relation loading) in team wiki or code examples to accelerate adoption

## Continuation Context


Verify commands:
- grep -r 'from @prisma/client' packages/trpc/server --include='*.ts' | wc -l
- test -f prisma/schema.prisma && echo 'Schema file exists' || echo 'Schema file missing'
- grep -r 'executeRaw\|queryRaw' packages/trpc/server --include='*.ts' | wc -l

Accept when:
- All tRPC router files in the detected pattern (find-attachments, webhook-router, get-envelope-field) import and use PrismaClient
- A prisma/schema.prisma file exists at the project root defining all database models
- Raw SQL usage (if any) is documented with inline comments explaining the exception and has corresponding tracking issues

## Enforcement

- Verified by: Automated CI checks that verify Prisma Client is imported in all files matching database access patterns
- Verified by: Code review checklist requiring justification for any raw SQL usage
- Verified by: Static analysis rules that flag direct database driver imports (e.g., 'pg', 'mysql2') outside of Prisma configuration
- Violation handling: CI pipeline fails if database access code is detected without Prisma Client import
- Violation handling: Pull requests with raw SQL require tech lead approval and exception documentation
- Violation handling: Quarterly architecture reviews identify and remediate non-compliant patterns
- Exception process: Developer documents the specific Prisma limitation or performance issue in a GitHub issue
- Exception process: Tech lead or architect reviews the justification and approves/rejects with written rationale
- Exception process: Approved exceptions are documented in code with inline comments linking to the approval issue
- Exception process: Exception registry is maintained and reviewed quarterly to identify patterns that should be escalated to Prisma team