# Adopt Server-Only Module Pattern for External API Integrations: Server Only Modules

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple server-only modules that handle external API integrations, PDF generation, and sensitive operations that must never execute in client-side contexts
- Pattern detected across 13 files with 89.81% confidence, indicating a consistent architectural approach to isolating server-side logic from client-accessible code
- Files are organized under 'server-only' namespaces (e.g., packages/lib/server-only/pdf/, packages/lib/server-only/recipient/) suggesting intentional architectural boundaries
- The pattern appears in critical security-sensitive operations including PDF rendering, audit log generation, rejection stamps, and recipient turn validation
- Remix framework's authenticated routes interact with these server-only modules, establishing a clear separation between presentation and business logic layers

## Problem Statement

External API integrations and server-side operations require strict isolation from client-side code to prevent security vulnerabilities, API key exposure, and unauthorized access. Without a standardized pattern for server-only modules, developers may inadvertently expose sensitive operations to client bundles, leading to security breaches and compliance violations.

## Decision

1. MAY: Server-only modules MAY expose type definitions and interfaces for client-side consumption while keeping implementation details isolated

## Policy Block

- MAY Server-only modules MAY expose type definitions and interfaces for client-side consumption while keeping implementation details isolated

In scope:
- All external API integrations (third-party services, payment processors, authentication providers)
- PDF generation and document manipulation operations
- Audit log rendering and compliance reporting
- Recipient validation and turn-based workflow logic
- Database queries that expose sensitive data or business logic
- Server-side rendering operations that require privileged access

Out of scope:
- Client-side API calls to internal endpoints (handled by framework routing)
- Public-facing REST or GraphQL endpoints (covered by separate API design patterns)
- Shared utility functions that contain no sensitive logic or credentials
- Type definitions and interface contracts shared between client and server

Exceptions:
- EXC-001: A module contains only type definitions or interfaces with no implementation logic
- EXC-002: Development/testing environments require mock implementations for client-side testing

## Rationale

- Pattern detected with 89.81% confidence across 13 files demonstrates this is an established architectural principle in the codebase
- Server-only isolation prevents accidental exposure of API keys, credentials, and sensitive business logic in client-side JavaScript bundles
- Clear directory structure (server-only namespace) provides immediate visual indication of security boundaries and reduces developer error
- Framework-specific patterns (Remix authenticated routes) already leverage this architecture, indicating successful integration with existing infrastructure

## Consequences

Positive:
- Significantly reduced attack surface by preventing client-side access to sensitive operations and credentials
- Clear architectural boundaries improve code organization and make security reviews more efficient
- Build-time enforcement prevents accidental security violations before code reaches production
- Easier compliance with security standards (SOC2, GDPR) through demonstrable separation of concerns

Negative:
- Requires additional boilerplate for client-server communication through framework-specific mechanisms
- May increase complexity for developers unfamiliar with server-only patterns and framework routing
- Build configuration must be carefully maintained to enforce boundaries, adding tooling complexity
- Testing requires separate strategies for server-only modules versus client-accessible code

## Alternatives

- Use environment variable checks within shared modules to conditionally execute server-only code (rejected)
  Rejected because: Runtime checks are error-prone and still bundle sensitive code in client builds, creating security risks even if code paths are not executed
  When valid: Never recommended for production systems with security requirements
- Implement all external integrations as separate microservices with internal API boundaries (rejected)
  Rejected because: Adds significant operational complexity and latency for operations that can be safely handled within the monolith using server-only patterns
  When valid: Consider for extremely high-scale operations or when external integrations require independent scaling characteristics
- Use framework-agnostic server-only markers (e.g., 'use server' directives) without directory structure conventions (deferred)
  Rejected because: While directives provide build-time enforcement, combining them with directory conventions provides both visual clarity and tooling support
  When valid: May be adopted as complementary approach when framework support matures

## Risks

- Developers may inadvertently import server-only modules in client code if build tooling fails to catch violations
  Mitigation: Implement ESLint rules, TypeScript path restrictions, and CI checks to detect server-only imports in client contexts
  Owner: Platform Engineering Team
- Refactoring may accidentally move server-only code into shared directories, breaking security boundaries
  Mitigation: Require security review for any changes to server-only module locations and implement automated path validation in CI
  Owner: Security Team
- Third-party dependencies imported by server-only modules may contain client-incompatible code that breaks builds
  Mitigation: Use separate package.json dependencies for server-only modules or leverage framework-specific dependency isolation features
  Owner: Engineering Team

## Implementation Notes

- Establish 'packages/lib/server-only/' as the canonical location for all server-only modules, organized by domain (pdf, recipient, auth, etc.)
- Configure build tools (Webpack, Vite, etc.) to fail builds if server-only paths are detected in client bundle analysis
- Create ESLint plugin or rule to detect imports from server-only paths in client-side files (routes, components, hooks)
- Document framework-specific patterns for exposing server-only functionality (Remix loaders/actions, Next.js API routes, tRPC procedures)
- Implement TypeScript path mapping to make server-only imports explicit (e.g., '@server/*' vs '@client/*')
- Add CI verification step that analyzes bundle outputs to ensure no server-only code appears in client chunks

## Continuation Context


Verify commands:
- grep -r "from.*server-only" apps/remix/app/routes --include="*.tsx" --include="*.ts" | grep -v "loader\|action" || echo "No client-side server-only imports detected"
- find packages/lib/server-only -type f -name "*.ts" -exec grep -l "export.*API_KEY\|SECRET\|PASSWORD" {} \; | wc -l
- npm run build && du -sh dist/client/**/*.js | awk '{if($1 ~ /M/ && $1+0 > 5) print "Warning: Large client bundle detected: " $0}'

Accept when:
- All server-only modules reside under designated server-only directory structures and are not imported directly in client-side code
- Build process successfully prevents bundling of server-only code in client JavaScript outputs
- CI pipeline includes automated checks that fail on detection of server-only imports in client contexts
- All external API integrations, PDF operations, and audit log rendering are implemented as server-only modules

## Enforcement

- Verified by: Automated CI checks analyzing import paths and bundle contents
- Verified by: ESLint rules enforcing server-only import restrictions
- Verified by: Code review checklist requiring verification of server-only boundaries for security-sensitive changes
- Verified by: Build-time bundle analysis preventing server-only code in client outputs
- Violation handling: Build failures block deployment when server-only code is detected in client bundles
- Violation handling: ESLint violations require resolution before PR approval
- Violation handling: Security team notification for any violations detected in production builds
- Violation handling: Immediate rollback procedures if server-only code exposure is detected post-deployment
- Exception process: Submit exception request to security team with detailed justification and risk assessment
- Exception process: Tech lead and security team joint review required for approval
- Exception process: Approved exceptions must be documented in ADR amendments with time-bound review periods
- Exception process: All exceptions require compensating controls (e.g., additional monitoring, runtime validation)