# Enforce Server-Only Code Isolation for Runtime Environment Boundaries: Projects Use Framework

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all server-side code modules and applies to runtime environment boundary enforcement across the codebase.

## Context

- The codebase contains modules explicitly designated as 'server-only' (e.g., packages/lib/server-only/*) indicating a deliberate architectural boundary between client and server execution contexts
- Pattern detected across 7 files with 91.40% confidence, including PDF generation, field manipulation, SSO settings, and API implementation modules
- Modern full-stack frameworks (Remix, Next.js) enable code sharing between client and server, creating risk of accidentally exposing server-side logic, secrets, or database access to client bundles
- The pattern shows consistent use of server-only paths for sensitive operations like audit log generation, template field manipulation, and API implementation
- Environment-specific configuration and execution boundaries are critical for security, performance, and proper separation of concerns in distributed applications

## Problem Statement

Without explicit runtime environment boundaries and enforcement mechanisms, server-side code containing sensitive logic, database operations, API keys, and privileged operations can accidentally leak into client-side bundles, creating security vulnerabilities, exposing secrets, and violating the principle of least privilege. The codebase needs a standardized approach to isolate server-only code and prevent its execution or bundling in client contexts.

## Decision

1. MAY: Projects MAY use framework-specific mechanisms (React Server Components, Remix loaders, Next.js server actions) as additional enforcement layers

## Policy Block

- MAY Projects MAY use framework-specific mechanisms (React Server Components, Remix loaders, Next.js server actions) as additional enforcement layers

In scope:
- All modules in 'server-only' directories or with server-only designations
- Database access layers and ORM operations
- PDF generation and document processing services
- Authentication and authorization logic
- API implementation and route handlers
- Environment variable access for secrets and credentials
- Server-side template rendering and field manipulation
- Audit logging and compliance operations

Out of scope:
- Client-side UI components and presentation logic
- Public API client libraries intended for browser use
- Shared utility functions with no environment dependencies
- Type definitions and interfaces used across boundaries
- Client-side state management and routing
- Browser-specific APIs and DOM manipulation

Exceptions:
- EXC-001: Isomorphic utilities that safely operate in both environments without exposing secrets or privileged operations
- EXC-002: Development and testing environments where server-only enforcement is relaxed for debugging purposes

## Rationale

- Pattern detected with 91.40% confidence across 7 critical files including PDF generation, field manipulation, SSO settings, and API implementation, demonstrating consistent architectural intent
- Server-only isolation prevents accidental exposure of sensitive operations, database credentials, and business logic to client bundles, reducing attack surface
- Explicit environment boundaries improve code organization, make security reviews more effective, and reduce cognitive load by clearly separating concerns
- Modern bundlers and frameworks support server-only enforcement, making this pattern practical and enforceable at build time

## Consequences

Positive:
- Significantly reduced risk of accidentally exposing secrets, database connections, or privileged operations to client-side code
- Clearer architectural boundaries make code reviews, security audits, and onboarding more effective
- Smaller client bundle sizes by excluding server-only dependencies from browser builds
- Better separation of concerns enables independent evolution of client and server codebases
- Framework-level enforcement provides compile-time safety rather than relying solely on runtime checks

Negative:
- Additional directory structure and naming conventions increase initial project setup complexity
- Developers must understand and respect environment boundaries, requiring training and documentation
- Some code duplication may occur when similar logic is needed in both environments
- Build tooling configuration becomes more complex to enforce server-only boundaries
- Debugging across environment boundaries may be more challenging without proper tooling

## Alternatives

- Use runtime environment checks only (if (typeof window === 'undefined')) without structural isolation (rejected)
  Rejected because: Runtime checks are error-prone, easy to forget, and don't prevent bundling server code into client builds, leading to larger bundles and potential secret exposure
  When valid: Only acceptable for simple isomorphic utilities with no security implications
- Maintain completely separate codebases for client and server with no shared code (rejected)
  Rejected because: Eliminates code reuse benefits, increases maintenance burden, and prevents sharing of type definitions and business logic that can safely exist in both contexts
  When valid: Appropriate for microservices architectures with completely independent client and server teams
- Rely solely on framework conventions (e.g., Next.js automatic server/client splitting) without explicit server-only markers (deferred)
  Rejected because: Framework conventions vary and may change; explicit markers provide defense in depth and work across framework migrations
  When valid: Can be used as a complementary approach alongside explicit server-only designation

## Risks

- Developers may circumvent server-only boundaries when facing tight deadlines or lacking understanding of security implications
  Mitigation: Implement automated build-time checks, code review checklists, and provide clear documentation with examples. Configure linters to detect violations.
  Owner: Engineering team and security team
- Framework updates or bundler changes may break server-only enforcement mechanisms
  Mitigation: Include integration tests that verify server-only code is not included in client bundles. Monitor framework changelogs and test enforcement after upgrades.
  Owner: Platform engineering team
- Overly strict enforcement may block legitimate use cases for isomorphic code, reducing developer productivity
  Mitigation: Provide clear exception process, maintain list of approved isomorphic utilities, and regularly review exceptions to refine policy boundaries.
  Owner: Architecture team

## Implementation Notes

- Use 'server-only' npm package or framework-specific directives ('use server') at the top of server-only modules to enforce boundaries
- Configure bundler (Webpack, Vite, etc.) to fail builds if server-only modules are imported by client code
- Establish naming conventions: 'server-only/' directories, '.server.ts' suffixes, or 'server/' subdirectories for clear visual identification
- Create shared type definition packages that can be safely imported by both environments without including implementation
- Document API boundaries clearly: use OpenAPI/GraphQL schemas to define client-server contracts
- Set up ESLint rules (e.g., 'no-restricted-imports') to prevent client code from importing server-only paths

## Continuation Context


Verify commands:
- grep -r "from.*server-only" apps/*/client apps/*/components --include="*.ts" --include="*.tsx" || echo 'No client imports of server-only code found'
- find packages/lib/server-only -type f -name '*.ts' -exec grep -L "'server-only'" {} \; | wc -l | grep -q '^0$' && echo 'All server-only files have guards'
- npm run build 2>&1 | grep -i 'server.*client.*bundle' && exit 1 || echo 'Build verification passed'

Accept when:
- No client-side code directly imports from server-only directories or modules
- All server-only modules include appropriate runtime guards or build-time enforcement markers
- Build process successfully detects and rejects attempts to bundle server-only code in client bundles
- Code review checklist includes verification of server-only boundary compliance for new modules

## Enforcement

- Verified by: Automated build-time checks that fail on client-side imports of server-only modules
- Verified by: ESLint rules configured to detect restricted imports in CI pipeline
- Verified by: Code review process with explicit checklist item for environment boundary verification
- Verified by: Bundle analysis in CI to detect unexpected server dependencies in client builds
- Violation handling: Build failures block merge to main branch when server-only violations are detected
- Violation handling: ESLint violations require resolution or explicit suppression with justification
- Violation handling: Security team notification for violations involving credential or secret exposure
- Violation handling: Post-incident review required if server-only code is discovered in production client bundles
- Exception process: Developer submits exception request with justification to architecture team
- Exception process: Security team reviews if exception involves sensitive data or operations
- Exception process: Approved exceptions documented in ADR amendments or exception registry
- Exception process: Exceptions reviewed quarterly to determine if they can be eliminated through refactoring