# Adopt Server-Only Module Pattern for Backend-Specific Library Code: Server Only Modules

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains backend-specific functionality that should never be bundled or executed in client-side environments, including database access, PDF generation, and API token validation
- A 'server-only' module pattern has been detected across 3 files with 88.43% confidence, indicating a consistent architectural approach to isolating server-side code
- Modern full-stack frameworks like Next.js support both client and server code in the same repository, creating risk of accidentally importing server-only code into client bundles
- The pattern is concentrated in specific functional areas: public API authentication (get-user-by-token.ts), PDF processing helpers (helpers.ts), and document generation (generate-partial-signed-pdf.ts)
- This pattern enables clear separation of concerns and prevents security vulnerabilities from exposing server-side logic or credentials to the client

## Problem Statement

Without explicit architectural boundaries between server-only and universal code, developers may accidentally import backend modules into client-side code, leading to bundle bloat, runtime errors, or security vulnerabilities from exposing server-side logic, database queries, or API credentials in the browser.

## Decision

1. SHOULD: Server-only modules SHOULD be organized by functional domain (e.g., public-api, pdf, database) within the server-only namespace

## Policy Block

- SHOULD Server-only modules SHOULD be organized by functional domain (e.g., public-api, pdf, database) within the server-only namespace

In scope:
- All database access layers and ORM queries
- Authentication and authorization logic including token validation
- PDF generation and document processing utilities
- File system operations and server-side I/O
- API route handlers that access server-only resources
- Server-side configuration and environment variable access

Out of scope:
- Shared TypeScript types and interfaces used for type checking
- Universal utility functions that work in both client and server environments
- Client-side API client libraries that make HTTP requests
- React components and UI logic
- Client-side state management code

Exceptions:
- EXC-001: Type-only imports from server-only modules are needed for client-side type checking

## Rationale

- The pattern was detected across 3 files with 88.43% significance, indicating a deliberate architectural choice rather than coincidental naming
- Server-only isolation prevents accidental exposure of sensitive backend logic, credentials, or database queries in client bundles, reducing security attack surface
- Clear module boundaries improve developer experience by providing compile-time or runtime errors when attempting to import server code in client contexts
- The pattern aligns with modern full-stack framework best practices (Next.js, Remix, SvelteKit) that support server-only module protection

## Consequences

Positive:
- Enhanced security posture by preventing server-side code and credentials from being bundled into client-side JavaScript
- Reduced client bundle size by excluding server-only dependencies from browser builds
- Improved developer experience with clear, enforceable boundaries between client and server code
- Better code organization with explicit functional domains (public-api, pdf) within the server-only namespace

Negative:
- Additional cognitive overhead for developers to understand and maintain the server-only module structure
- Potential for runtime errors if server-only imports are not caught during development or build time
- Requires framework support or additional tooling (like the 'server-only' npm package) to enforce boundaries
- May complicate code sharing between server and client when universal utilities are needed

## Alternatives

- Use file naming conventions (e.g., *.server.ts) without runtime enforcement (rejected)
  Rejected because: Naming conventions alone provide no runtime or build-time enforcement, allowing accidental imports to slip through code review
  When valid: In small projects with a single developer where discipline can be maintained manually
- Separate server and client code into completely different packages or monorepo workspaces (rejected)
  Rejected because: Creates unnecessary complexity for sharing types and utilities, and doesn't align with modern full-stack framework patterns
  When valid: In microservices architectures where frontend and backend are truly separate applications
- Use build-time bundler configuration to exclude server paths from client builds (deferred)
  Rejected because: Can be used as a complementary approach but doesn't provide runtime safety or clear developer intent
  When valid: As an additional layer of defense alongside the server-only module pattern

## Risks

- Developers may forget to place new server-side code in the server-only namespace, creating security vulnerabilities
  Mitigation: Implement automated linting rules and code review checklists to verify server-only placement for database/auth code
  Owner: Engineering team
- Runtime errors from accidental server-only imports may not be caught until production if testing coverage is insufficient
  Mitigation: Add build-time checks using bundler analysis and ensure CI/CD pipeline includes client-side build verification
  Owner: DevOps team
- The 'server-only' package or enforcement mechanism may not work correctly with all bundlers or framework versions
  Mitigation: Document tested framework versions and maintain integration tests that verify server-only enforcement
  Owner: Platform team

## Implementation Notes

- Install the 'server-only' npm package and import it at the top of all server-only module files to enable runtime enforcement
- Organize server-only code under a consistent path structure like 'packages/lib/server-only/{domain}/' where domain represents functional areas (public-api, pdf, database, etc.)
- Configure TypeScript path aliases to make server-only imports explicit and easily identifiable in code reviews
- Add ESLint rules or custom linting to detect and prevent imports of server-only paths in client-side code
- Document the pattern in team onboarding materials with clear examples of what belongs in server-only vs. universal modules

## Continuation Context


Verify commands:
- grep -r "from.*server-only" packages/lib/server-only/ | wc -l
- find packages/lib/server-only -name '*.ts' -exec grep -L "'server-only'" {} \;
- npm run build:client && ! grep -r "server-only" dist/client/

Accept when:
- All files in the server-only directory import the 'server-only' package at the top of the file
- Client-side build output contains no references to server-only module paths or code
- Attempting to import a server-only module in a client component results in a clear error message during development or build

## Enforcement

- Verified by: Automated ESLint rules checking for server-only imports in client code
- Verified by: CI/CD pipeline build verification that analyzes client bundle contents
- Verified by: Code review checklist items for new server-side functionality
- Verified by: Runtime errors from 'server-only' package during development testing
- Violation handling: Build failures when server-only code is detected in client bundles
- Violation handling: Runtime errors thrown by 'server-only' package when imported in browser context
- Violation handling: Pull request blocks from automated linting failures
- Violation handling: Security review triggered for any violations that reach production
- Exception process: Developer submits exception request with justification to tech lead
- Exception process: Tech lead evaluates whether code truly needs server-only isolation or if it can be refactored as universal code
- Exception process: If approved, exception is documented in ADR with specific use case and type-only import pattern
- Exception process: Exception is reviewed quarterly to determine if it can be eliminated through refactoring