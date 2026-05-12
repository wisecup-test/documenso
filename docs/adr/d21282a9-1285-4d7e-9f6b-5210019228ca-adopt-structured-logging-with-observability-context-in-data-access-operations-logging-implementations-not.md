# Adopt Structured Logging with Observability Context in Data Access Operations: Logging Implementations Not

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all data access operations in server-side code. All database queries, data mutations, and access pattern implementations MUST comply with the logging and observability requirements specified herein.

## Context

- The codebase exhibits a consistent pattern of structured logging in data access operations across multiple server-only modules, with 3 files showing this pattern at 91.90% confidence
- Data access operations in server-only contexts (folders, organisations, security settings) require observability for debugging, performance monitoring, and audit trails
- The pattern is specifically associated with the obs.logging facet, indicating a deliberate architectural choice to instrument data access layers with observability primitives
- Modern distributed systems require comprehensive logging at data access boundaries to diagnose issues, track performance, and maintain compliance with data governance requirements
- The pattern appears in critical business operations (folder management, organisation creation, linked account security) suggesting this is a cross-cutting concern for all data access

## Problem Statement

Data access operations in server-side code lack consistent observability instrumentation, making it difficult to debug issues, monitor performance, audit data access patterns, and maintain compliance. Without standardized logging at data access boundaries, teams cannot effectively trace requests, identify bottlenecks, or investigate security incidents involving data operations.

## Decision

1. MUST_NOT: Logging implementations MUST NOT log sensitive data (passwords, tokens, PII) in plain text without appropriate redaction

## Policy Block

- MUST_NOT Logging implementations MUST NOT log sensitive data (passwords, tokens, PII) in plain text without appropriate redaction

In scope:
- All server-only data access modules (database queries, ORM operations, repository patterns)
- CRUD operations for core business entities (folders, organisations, users, security settings)
- Data mutation operations (create, update, delete) that modify persistent state
- Data retrieval operations that access sensitive or business-critical information
- Internal API endpoints that perform data access on behalf of authenticated users

Out of scope:
- Client-side data access code (browser-based operations)
- Public API endpoints with separate logging infrastructure
- Third-party library internals (unless wrapped by application code)
- Development-only utilities and test fixtures
- Static data access (configuration files, constants) that does not involve dynamic queries

Exceptions:
- EXC-001: Performance-critical hot paths where logging overhead is measured and documented to cause >5% performance degradation
- EXC-002: Legacy code modules scheduled for deprecation within 3 months

## Rationale

- The pattern detection identified this approach in 3 critical server-side modules with 91.90% confidence, indicating this is an established architectural practice worth codifying
- Structured logging at data access boundaries provides essential observability for debugging production issues, especially in distributed systems where request context must be preserved across service boundaries
- The obs.logging facet association demonstrates this is a deliberate observability strategy, not an accidental pattern, making it suitable for standardization
- Consistent logging across data access operations enables centralized log aggregation, analysis, and alerting, which are critical for maintaining system reliability and security compliance

## Consequences

Positive:
- Improved debugging capabilities through consistent, structured logs across all data access operations
- Enhanced performance monitoring and bottleneck identification through timing data at data access boundaries
- Better security audit trails for compliance requirements (GDPR, SOC2) through comprehensive logging of data access patterns
- Reduced mean time to resolution (MTTR) for production incidents through correlation identifiers and contextual information
- Foundation for advanced observability practices (distributed tracing, anomaly detection, automated alerting)

Negative:
- Increased storage costs for log retention, especially in high-throughput systems with frequent data access operations
- Potential performance overhead from logging operations, particularly for synchronous logging implementations
- Additional development effort required to implement and maintain logging instrumentation across all data access code
- Risk of logging sensitive data if developers do not properly implement redaction policies
- Increased complexity in data access code with additional logging logic interleaved with business logic

## Alternatives

- Implement logging only at API/controller boundaries without data access layer instrumentation (rejected)
  Rejected because: This approach lacks visibility into internal data access patterns, making it impossible to diagnose issues within complex data operations or identify which specific database queries are causing performance problems
  When valid: May be acceptable for simple CRUD applications with single-layer architectures where API boundaries directly map to database operations
- Use database-level query logging and monitoring tools exclusively without application-level logging (rejected)
  Rejected because: Database logs lack application context (user IDs, request IDs, business operation context) making it difficult to correlate database activity with application behavior and user actions
  When valid: Can be used as a complementary approach for database performance tuning but should not replace application-level logging
- Implement aspect-oriented programming (AOP) or decorators to automatically inject logging into all data access methods (deferred)
  Rejected because: While this could reduce boilerplate, it requires additional infrastructure and may make it harder to customize logging for specific operations. Consider for future optimization once manual logging patterns are well-established
  When valid: Should be reconsidered once the team has established stable logging patterns and can codify them into reusable aspects or decorators

## Risks

- Accidental logging of sensitive data (PII, credentials, tokens) leading to security vulnerabilities and compliance violations
  Mitigation: Implement automated scanning for sensitive data patterns in logs, provide redaction utilities, conduct security training on logging best practices, and enforce code review requirements for data access code
  Owner: Security team and engineering leads
- Performance degradation in high-throughput data access operations due to synchronous logging overhead
  Mitigation: Use asynchronous logging libraries, implement log sampling for high-frequency operations, establish performance budgets, and monitor logging overhead in production
  Owner: Performance engineering team
- Inconsistent implementation across teams leading to gaps in observability coverage
  Mitigation: Provide logging libraries and templates, establish code review checklists, implement linting rules to detect missing logging, and conduct regular audits of data access code
  Owner: Platform team and engineering managers

## Implementation Notes

- Create a shared logging utility library that provides standardized methods for data access logging with built-in redaction for common sensitive fields
- Establish logging templates for common data access patterns (CRUD operations, queries, transactions) to reduce implementation burden and ensure consistency
- Configure log aggregation infrastructure to collect, index, and retain data access logs according to compliance requirements (typically 90 days for operational logs, longer for audit logs)
- Implement correlation ID propagation through middleware to ensure all data access operations within a request share the same correlation identifier
- Use environment-based log level configuration to enable verbose logging in development/staging while keeping production logs focused on INFO and above

## Continuation Context


Verify commands:
- grep -r "logger\|console\.log\|log\." packages/lib/server-only --include="*.ts" | grep -E "(query|find|create|update|delete|fetch)" | wc -l
- find packages/lib/server-only -name "*.ts" -exec grep -l "import.*logger\|import.*log" {} \; | wc -l
- npm run test:logging-coverage 2>&1 | grep "Data access operations with logging" | grep -oE "[0-9]+%"

Accept when:
- At least 90% of data access operations in server-only modules contain structured logging statements with contextual information
- All new data access code passes automated linting checks for required logging instrumentation
- Code review checklist confirms presence of appropriate logging for data access operations before merge approval
- Log aggregation dashboard shows consistent log volume and structure from all data access modules

## Enforcement

- Verified by: Automated linting rules in CI pipeline that detect data access methods without logging statements
- Verified by: Code review checklist requiring reviewers to verify logging implementation in data access code
- Verified by: Periodic automated audits scanning server-only modules for data access patterns and checking for corresponding log statements
- Verified by: Integration tests that verify log output for critical data access operations
- Violation handling: CI pipeline fails if linting rules detect data access methods without required logging
- Violation handling: Pull requests cannot be merged without code review approval confirming logging compliance
- Violation handling: Quarterly audit reports identify non-compliant modules and create remediation tickets assigned to owning teams
- Violation handling: New violations in existing code trigger warnings; violations in new code block deployment
- Exception process: Developer submits exception request documenting the specific operation, rationale for exemption, and alternative observability approach
- Exception process: Architecture review board evaluates exception request within 5 business days
- Exception process: Approved exceptions are documented in code comments with ADR reference and expiration date
- Exception process: Exception registry is reviewed quarterly to ensure exceptions remain valid and are not being overused