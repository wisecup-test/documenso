# Standardize Public API Contract Design with Type-Safe Route Handlers and Internationalization: Public Routes Follow

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 24 files implementing public-facing API routes using Remix framework conventions with authenticated and organization-scoped endpoints
- Routes follow a consistent pattern for team, organization, and personal settings with URL parameter-based routing (e.g., $teamUrl, $orgUrl)
- Internationalization support is integrated via Lingui configuration, indicating multi-language API response requirements
- Server-side PDF generation capabilities (audit logs, certificates) suggest document-oriented API endpoints requiring specialized content-type handling
- The pattern shows strong consistency (89.90% confidence) across authentication-gated routes, settings management, and document generation endpoints

## Problem Statement

Public-facing APIs require consistent contract design, type safety, authentication patterns, and internationalization support across diverse endpoint types including settings management, document generation, and organizational resource access. Without standardized patterns, API consumers face inconsistent interfaces, authentication flows, and response formats, leading to integration difficulties and maintenance overhead.

## Decision

1. MUST: All public API routes MUST follow Remix file-based routing conventions with explicit authentication boundaries (_authenticated+ prefix)

## Policy Block

- MUST All public API routes MUST follow Remix file-based routing conventions with explicit authentication boundaries (_authenticated+ prefix)

In scope:
- All Remix route handlers under apps/remix/app/routes/
- Server-side PDF generation utilities in packages/lib/server-only/
- Internationalization configuration files (lingui.config.ts)
- Authentication-gated endpoints for organizations, teams, and personal settings
- Document generation APIs (certificates, audit logs, rejection stamps)

Out of scope:
- Internal service-to-service APIs not exposed through Remix routes
- WebSocket or real-time communication endpoints
- Static asset serving and CDN integration
- Third-party API integrations and webhooks
- Database migration scripts and schema definitions

Exceptions:
- EXC-001: Public documentation or health check endpoints require unauthenticated access
- EXC-002: Legacy API endpoints require different URL parameter conventions for backward compatibility

## Rationale

- Pattern detected across 24 files with 89.90% confidence indicates strong architectural consistency that should be formalized
- Remix file-based routing provides type-safe, convention-over-configuration approach that reduces boilerplate and improves developer experience
- URL parameter-based resource identification ($orgUrl, $teamUrl) enables clean, RESTful API design with clear resource hierarchies
- Integrated internationalization support ensures API can serve global user base with locale-appropriate responses
- Server-side PDF generation pattern demonstrates need for specialized content-type handling in document-oriented endpoints

## Consequences

Positive:
- Consistent API contract design reduces integration complexity for frontend and external consumers
- Type-safe route handlers with TypeScript catch errors at compile time rather than runtime
- File-based routing conventions make API structure discoverable and self-documenting
- Internationalization support enables seamless expansion to new markets and languages
- Centralized authentication patterns reduce security vulnerabilities and audit complexity

Negative:
- Remix framework coupling creates migration complexity if framework needs to change
- File-based routing conventions may conflict with complex API versioning strategies
- Server-side PDF generation adds computational overhead and potential performance bottlenecks
- Strict URL parameter conventions may require refactoring existing endpoints that use query parameters
- Internationalization overhead increases response processing time and memory usage

## Alternatives

- Use Express.js with manual route registration and middleware chains for API endpoints (rejected)
  Rejected because: Lacks type safety, requires more boilerplate, and doesn't provide integrated data loading patterns that Remix offers
  When valid: For microservices that don't require server-side rendering or tight frontend integration
- Implement GraphQL API with Apollo Server for unified query interface (rejected)
  Rejected because: Adds complexity for document generation endpoints, requires schema duplication, and doesn't align with existing REST patterns
  When valid: For complex data aggregation scenarios or when clients need flexible query capabilities
- Use tRPC for end-to-end type safety without REST conventions (deferred)
  Rejected because: Requires significant refactoring of existing routes and may not support all document generation use cases
  When valid: For new greenfield projects or internal APIs where full TypeScript stack is guaranteed

## Risks

- Remix framework updates may introduce breaking changes to routing conventions or loader/action APIs
  Mitigation: Pin Remix version, maintain comprehensive integration tests, and allocate time for framework upgrades in each sprint
  Owner: Platform Engineering Team
- PDF generation endpoints may become performance bottlenecks under high load
  Mitigation: Implement async job queue for large documents, add caching layer for frequently requested PDFs, and monitor endpoint latency
  Owner: Backend Performance Team
- Internationalization support may be incomplete for all API responses, leading to inconsistent user experience
  Mitigation: Audit all API endpoints for i18n coverage, implement automated tests for locale switching, and establish translation workflow
  Owner: Internationalization Team

## Implementation Notes

- Use Remix's loader functions for GET requests and action functions for mutations, ensuring proper HTTP method semantics
- Implement shared authentication logic in _layout.tsx files to avoid duplication across route handlers
- Configure Lingui with appropriate locale detection strategy (Accept-Language header, user preferences, URL parameters)
- For PDF generation, use streaming responses where possible to reduce memory footprint and improve time-to-first-byte
- Document URL parameter conventions in API specification (OpenAPI/Swagger) for external consumers
- Implement rate limiting and request validation middleware at the route level for public endpoints

## Continuation Context


Verify commands:
- grep -r '_authenticated+' apps/remix/app/routes/ | wc -l
- find apps/remix/app/routes -name '*.tsx' -exec grep -l 'loader\|action' {} \; | wc -l
- grep -r 'lingui' . --include='*.config.*' --include='*.tsx' | head -5
- find packages/lib/server-only -name '*pdf*.ts' -type f

Accept when:
- All authenticated routes use _authenticated+ prefix and implement proper authorization checks
- At least 90% of API routes follow the established URL parameter conventions ($orgUrl, $teamUrl)
- Lingui configuration is present and integrated into route handlers for internationalized responses
- PDF generation utilities are isolated in server-only packages with appropriate content-type handling

## Enforcement

- Verified by: Automated CI checks scanning route file structure for naming convention compliance
- Verified by: TypeScript compilation enforcing type safety on loader and action function signatures
- Verified by: Integration tests validating authentication boundaries and authorization logic
- Verified by: Code review checklist requiring internationalization support verification for new endpoints
- Violation handling: CI pipeline fails if routes don't follow authentication prefix conventions
- Violation handling: Pull requests blocked until TypeScript compilation succeeds without errors
- Violation handling: Security team notified for any unauthenticated routes accessing sensitive data
- Violation handling: Automated issue creation for routes missing internationalization support
- Exception process: Submit exception request to API Governance Committee with business justification
- Exception process: Security team review required for authentication-related exceptions
- Exception process: Document approved exceptions in ADR amendments with expiration dates
- Exception process: Quarterly review of all active exceptions to assess continued validity