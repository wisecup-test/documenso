# Standardize Public API Route Structure and External Integration Patterns: Server Only Utilities

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all public-facing API routes, external integrations, and client-facing endpoints. It applies to both REST APIs and server-rendered routes that expose data or functionality to external consumers.

## Context

- The codebase contains 24 files exhibiting a consistent pattern (signature: eff7ce064ab2e3f219859e6512c7e811) related to public API design and external integrations
- Evidence spans multiple route handlers (authenticated routes, organization settings, team management, inbox), configuration files (lingui.config.ts), and server-side utilities (PDF generation, HTML-to-PDF conversion)
- The pattern appears in both client-facing routes (Remix app routes) and server-only utilities, suggesting a unified approach to external API interactions
- High confidence (89.90%) and significance (89.90%) across diverse file types indicates this is an established architectural pattern rather than coincidental similarity
- The pattern encompasses authentication-aware routes, multi-tenant organization/team structures, and document generation services that likely expose APIs or integrate with external systems

## Problem Statement

Without standardized patterns for public API design and external integrations, the codebase risks inconsistent authentication handling, unpredictable route structures, fragmented error responses, and difficult-to-maintain integration points. This creates friction for API consumers, increases security vulnerabilities, and makes it harder to evolve the API surface over time.

## Decision

1. MUST: Server-only utilities that generate external artifacts (PDFs, exports) MUST be isolated in dedicated server-only packages to prevent client-side exposure

## Policy Block

- MUST Server-only utilities that generate external artifacts (PDFs, exports) MUST be isolated in dedicated server-only packages to prevent client-side exposure

In scope:
- All Remix route handlers under /routes directory
- Server-only API utilities in packages/lib/server-only
- External integration configuration files (lingui.config.ts, API client configs)
- PDF generation and document export endpoints
- Organization, team, and user settings routes
- Authentication-gated public endpoints

Out of scope:
- Internal microservice-to-microservice communication
- Database access layer patterns
- Frontend component libraries without API interaction
- Build and deployment configuration
- Development-only tooling and scripts

Exceptions:
- EX-001: Legacy API endpoints that must maintain backward compatibility with existing external consumers
- EX-002: Third-party webhook receivers that must conform to external service specifications

## Rationale

- Pattern detected across 24 files with 89.90% confidence indicates this is an established, successful architectural approach that should be formalized
- Consistent route structure (authenticated+, organization/team URL parameters) improves API discoverability and reduces cognitive load for developers
- Separation of server-only utilities prevents accidental exposure of sensitive server-side logic or credentials to client bundles
- Standardized patterns enable automated tooling for API documentation generation, testing, and monitoring

## Consequences

Positive:
- Improved API consistency makes it easier for external consumers to predict endpoint behavior and structure
- Clear authentication boundaries reduce security vulnerabilities and make authorization logic easier to audit
- Colocated layouts and route handlers improve developer experience and reduce context switching
- Server-only package isolation prevents accidental client-side exposure of sensitive code or credentials

Negative:
- Strict route structure conventions may feel constraining for developers used to more flexible patterns
- Refactoring existing non-compliant routes to match the standard pattern requires coordination and testing effort
- URL parameter-based multi-tenancy may complicate caching strategies compared to header-based approaches
- Additional directory nesting (_authenticated+, _dynamic_personal_routes+) increases path length and complexity

## Alternatives

- Use header-based authentication and tenant identification instead of URL-based routing (rejected)
  Rejected because: URL-based patterns provide better cacheability, clearer access control boundaries, and more intuitive API structure for external consumers. Headers are less visible in logs and debugging tools.
  When valid: May be appropriate for internal microservice communication where URL structure is less important than performance
- Flatten route structure without authentication namespaces, relying on middleware for protection (rejected)
  Rejected because: Explicit authentication namespaces (_authenticated+) make security boundaries immediately visible in code structure and reduce risk of accidentally exposing protected routes
  When valid: Could work for smaller applications with simpler security requirements
- Separate public API routes into dedicated API gateway service (deferred)
  Rejected because: Current monolithic approach with clear patterns works well at current scale. Gateway adds operational complexity.
  When valid: Should be reconsidered if API traffic grows significantly or if multiple backend services need unified API facade

## Risks

- Inconsistent adoption across teams leads to fragmented API patterns despite documented standards
  Mitigation: Implement automated linting rules to detect non-compliant route structures. Add pre-commit hooks and CI checks. Provide code generation templates for new routes.
  Owner: Platform Engineering Team
- URL parameter-based multi-tenancy could expose tenant enumeration vulnerabilities if not properly validated
  Mitigation: Implement strict authorization checks that verify user access to specified orgUrl/teamUrl. Add rate limiting on tenant discovery endpoints. Log and alert on suspicious enumeration patterns.
  Owner: Security Team
- Server-only package boundaries could be accidentally violated during refactoring or by new developers
  Mitigation: Use build-time checks to prevent server-only imports in client code. Add ESLint rules for package boundaries. Include architecture training in onboarding.
  Owner: Engineering Team

## Implementation Notes

- Use Remix route conventions: _authenticated+ for protected routes, _layout.tsx for shared layouts, $ for dynamic parameters
- Organize server-only utilities under packages/lib/server-only with clear naming (htmltopdf, pdf) to signal their intended usage
- Implement consistent error response format: { error: string, code: string, details?: object } across all public endpoints
- Document all public API routes in OpenAPI/Swagger format, auto-generated from route definitions where possible
- Use TypeScript discriminated unions for route parameters to ensure type safety across organization/team/user contexts

## Continuation Context


Verify commands:
- grep -r "routes/_authenticated+" apps/remix/app/routes/ | wc -l  # Count authenticated routes
- find packages/lib/server-only -name '*.ts' | xargs grep -L 'server-only' | wc -l  # Check server-only markers
- grep -r "\$orgUrl\|\$teamUrl" apps/remix/app/routes/ | wc -l  # Verify multi-tenant URL patterns
- npm run type-check && npm run lint  # Ensure type safety and linting rules pass

Accept when:
- All authenticated routes are organized under _authenticated+ namespace or equivalent authentication boundary
- Server-only utilities are isolated in packages/lib/server-only and not imported by client code
- Multi-tenant routes consistently use URL parameters (orgUrl, teamUrl) for tenant identification
- Type checking and linting pass without errors related to route structure or API patterns

## Enforcement

- Verified by: Automated CI pipeline checks for route structure compliance using custom ESLint rules
- Verified by: Pre-commit hooks validate new routes follow naming conventions and authentication patterns
- Verified by: Code review checklist includes verification of API pattern compliance
- Verified by: Quarterly architecture audits review API consistency across the codebase
- Violation handling: CI pipeline fails if routes violate authentication namespace requirements
- Violation handling: Linting errors block PR merge for non-compliant route structures
- Violation handling: Architecture review required for any exceptions to standard patterns
- Violation handling: Non-compliant code flagged for refactoring in technical debt backlog
- Exception process: Submit exception request to architecture review board with justification and impact analysis
- Exception process: Security team review required for authentication-related exceptions
- Exception process: Document approved exceptions in ADR amendments with expiration date
- Exception process: Exceptions reviewed quarterly for continued validity or migration to standard pattern