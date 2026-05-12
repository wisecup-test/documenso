# Standardize Structured Logging with Context Propagation for API and Server Operations: Log Entries Not

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all logging operations in API endpoints, server-only functions, and service clients. All new code MUST comply with these logging standards.

## Context

- The codebase contains multiple API endpoints, server-side operations, and service clients that require consistent logging for debugging, monitoring, and operational visibility
- Pattern detected across 3 files with 90.57% confidence indicates a consistent approach to logging in public API contracts and server-only operations
- Evidence shows logging patterns in Hono API framework, organization member invite operations, and license client interactions, suggesting cross-cutting logging concerns
- Structured logging with context propagation is essential for distributed systems to trace requests across service boundaries and understand system behavior in production
- Without standardized logging practices, debugging production issues becomes difficult and operational metrics lack consistency

## Problem Statement

The system lacks a unified approach to logging across API endpoints, server-only functions, and service clients, leading to inconsistent log formats, missing contextual information, and difficulty in tracing requests through the system. This creates operational blind spots and increases mean time to resolution (MTTR) for production incidents.

## Decision

1. MUST_NOT: Log entries MUST NOT contain sensitive information such as passwords, tokens, API keys, or personally identifiable information (PII) unless explicitly redacted

## Policy Block

- MUST_NOT Log entries MUST NOT contain sensitive information such as passwords, tokens, API keys, or personally identifiable information (PII) unless explicitly redacted

In scope:
- All HTTP API endpoints exposed through Hono or similar frameworks
- Server-only business logic functions including organization management, user operations, and license validation
- Service clients for external API integrations and inter-service communication
- Background jobs and scheduled tasks that perform business operations
- Error handling and exception logging across all application layers

Out of scope:
- Third-party library internal logging (unless wrapped by application code)
- Development-only debug logging that is not deployed to production
- Infrastructure-level logging (OS, container runtime, load balancer) managed outside application code
- Database query logging handled by ORM or database driver configuration

Exceptions:
- EXC-001: Performance-critical hot paths where logging overhead is measured to exceed 5% of operation time
- EXC-002: Legacy code modules scheduled for deprecation within 3 months

## Rationale

- Pattern detected with 90.57% confidence across 3 files in critical paths (API layer, organization operations, license client) indicates this is an established architectural pattern worth codifying
- Structured logging with context propagation is an industry best practice for microservices and distributed systems, enabling effective observability and debugging
- Consistent logging standards reduce cognitive load for developers and operators by providing predictable log formats and content across the codebase
- The facet 'api.public.contracts' suggests this pattern is particularly important for public-facing interfaces where operational visibility is critical for SLA compliance

## Consequences

Positive:
- Improved operational visibility enables faster incident detection and resolution through consistent, searchable log data
- Distributed tracing capabilities through correlation ID propagation allow tracking requests across service boundaries
- Structured JSON logs enable automated analysis, alerting, and dashboard creation without custom parsing logic
- Standardized logging patterns reduce onboarding time for new developers and create consistent debugging experiences

Negative:
- Increased log volume from comprehensive logging may require investment in log aggregation infrastructure and storage
- Performance overhead from structured logging serialization may impact high-throughput endpoints (mitigated by sampling)
- Developers must learn and follow logging standards, adding initial complexity to code reviews and development
- Retrofitting existing code to comply with standards requires engineering effort and careful testing

## Alternatives

- Use plain text logging with printf-style formatting (rejected)
  Rejected because: Plain text logs are difficult to parse programmatically, lack structure for automated analysis, and don't support rich contextual metadata needed for distributed systems
  When valid: Only appropriate for simple scripts or local development where log aggregation is not required
- Implement logging only at API gateway/edge layer without internal service logging (rejected)
  Rejected because: Edge-only logging provides insufficient visibility into internal service behavior, making it impossible to debug issues in business logic or service-to-service communication
  When valid: Could be sufficient for simple monolithic applications with no internal service boundaries
- Use distributed tracing framework (OpenTelemetry) instead of structured logging (deferred)
  Rejected because: Not rejected but deferred - distributed tracing complements rather than replaces structured logging. Consider as future enhancement.
  When valid: Should be adopted alongside structured logging for comprehensive observability, particularly for complex distributed systems

## Risks

- Log volume growth may exceed storage capacity or budget constraints, leading to log retention issues or unexpected costs
  Mitigation: Implement log sampling for high-frequency operations, establish log retention policies, and monitor log volume metrics with alerting thresholds
  Owner: Operations team with engineering support
- Sensitive data may be inadvertently logged despite guidelines, creating security and compliance risks
  Mitigation: Implement automated scanning for common sensitive patterns (regex for tokens, emails, etc.), conduct security-focused code reviews, and provide redaction utilities
  Owner: Security team with engineering implementation
- Performance degradation in high-throughput endpoints due to logging overhead
  Mitigation: Benchmark logging performance, implement async logging where possible, use sampling for hot paths, and establish performance budgets in CI
  Owner: Engineering team

## Implementation Notes

- Create a shared logging utility library that encapsulates structured logging format, context propagation, and sensitive data redaction to ensure consistency
- Establish logging level conventions: ERROR for failures requiring intervention, WARN for degraded states, INFO for significant business events, DEBUG for detailed troubleshooting
- Integrate correlation ID generation at API gateway/entry points and ensure it propagates through all downstream calls via request context or headers
- Provide code examples and templates for common logging scenarios (API endpoint logging, error handling, external service calls) in developer documentation
- Configure log aggregation pipeline to parse structured JSON logs and create dashboards for key operational metrics (error rates, latency percentiles, request volumes)

## Continuation Context


Verify commands:
- grep -r 'console.log\|console.error' packages/ --include='*.ts' --include='*.js' | wc -l  # Should be 0 or minimal - use structured logger instead
- grep -r 'logger\.(info|error|warn|debug)' packages/api packages/lib/server-only --include='*.ts' | wc -l  # Should show widespread adoption
- grep -r 'password\|token\|apiKey' packages/ --include='*.ts' -A 2 -B 2 | grep -i 'log' | wc -l  # Should be 0 - no sensitive data in logs

Accept when:
- All API endpoints in packages/api contain structured logging calls with request context at entry and exit points
- Server-only functions in packages/lib/server-only log operation start/completion with entity identifiers for business operations
- No instances of console.log/console.error in production code paths (verified by linting rules)
- Automated security scanning detects no sensitive data patterns in log statements
- Log aggregation dashboard shows consistent structured log format across all services with parseable JSON

## Enforcement

- Verified by: Automated linting rules in CI pipeline to detect console.log usage and enforce structured logger imports
- Verified by: Code review checklist includes verification of logging standards compliance for all API and server-only code changes
- Verified by: Automated security scanning in CI to detect sensitive data patterns in proximity to logging statements
- Verified by: Log aggregation monitoring alerts on malformed or unparseable log entries indicating non-compliant logging
- Violation handling: CI pipeline fails on detection of console.log/console.error in production code paths
- Violation handling: Code review blocks merge if logging standards are not met in new API endpoints or server-only functions
- Violation handling: Security scanner findings for sensitive data in logs are treated as high-severity issues requiring immediate remediation
- Violation handling: Quarterly audit of logging compliance with remediation plans for non-compliant modules
- Exception process: Developer submits exception request with justification (performance impact, legacy code deprecation timeline, etc.) to technical lead
- Exception process: Technical lead reviews with operations/security team as appropriate based on exception type
- Exception process: Approved exceptions are documented in code comments with ticket reference and expiration date
- Exception process: Exception registry is reviewed quarterly to ensure temporary exceptions don't become permanent