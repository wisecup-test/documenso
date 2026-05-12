# Enforce Server-Only Module Isolation for Secrets Management: Implementations That Expose

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all server-side code that handles sensitive configuration, environment variables, or secrets management operations.

## Context

- The application handles sensitive configuration data including API keys, database credentials, and encryption secrets that must never be exposed to client-side code
- Modern full-stack frameworks like Next.js enable code sharing between server and client, creating risk of accidental secret leakage through bundling
- PDF processing operations require access to external services and file system operations that are inherently server-side concerns
- The codebase uses a 'server-only' module pattern to create compile-time guarantees that certain code paths never execute in browser contexts
- Pattern detected across 3 files with 87.60% confidence, indicating consistent architectural approach to secrets isolation

## Problem Statement

Without explicit architectural boundaries, sensitive configuration and secrets management code can accidentally be included in client-side bundles, exposing credentials to end users and creating severe security vulnerabilities. The framework needs a reliable mechanism to enforce server-only execution for modules that handle secrets, environment variables, or privileged operations.

## Decision

1. MUST: API implementations that expose server-side functionality MUST validate that secrets are only accessed in server contexts

## Policy Block

- MUST API implementations that expose server-side functionality MUST validate that secrets are only accessed in server contexts

In scope:
- All modules accessing process.env for secrets or credentials
- PDF processing utilities that require file system or external service access
- API implementation layers that handle authentication or authorization
- Database connection configuration and credential management
- Third-party service integrations requiring API keys or tokens

Out of scope:
- Public configuration values safe for client-side exposure (e.g., API endpoints, feature flags)
- Static assets and resources that don't contain sensitive data
- Client-side validation logic that doesn't require secrets
- Public API response types and interfaces

Exceptions:
- EXC-001: Non-sensitive environment variables explicitly prefixed with PUBLIC_ or NEXT_PUBLIC_ that are intended for client-side use

## Rationale

- Pattern detected with 87.60% confidence across 3 files demonstrates consistent architectural approach to secrets isolation in the codebase
- The 'server-only' package provides compile-time safety guarantees that prevent accidental bundling of sensitive code into client bundles
- Organizing secrets management in dedicated server-only modules creates clear architectural boundaries and reduces cognitive load for developers
- This approach aligns with security best practices for full-stack JavaScript applications and leverages framework-native capabilities

## Consequences

Positive:
- Compile-time protection against accidental secret exposure in client bundles
- Clear architectural boundaries make code review and security audits more effective
- Reduced risk of credential leakage through improved separation of concerns
- Framework-native approach integrates seamlessly with Next.js and similar platforms

Negative:
- Additional directory structure complexity with server-only path segments
- Developers must understand and remember to use server-only imports for sensitive operations
- Refactoring code between client and server contexts requires careful module reorganization
- Build-time errors may initially slow development until patterns are internalized

## Alternatives

- Runtime environment checks only (e.g., typeof window === 'undefined') (rejected)
  Rejected because: Runtime checks provide no compile-time safety and can be bypassed or forgotten, creating security vulnerabilities
  When valid: May be used as additional defense-in-depth alongside compile-time checks
- Separate backend service with API-only access to secrets (rejected)
  Rejected because: Adds significant architectural complexity and latency for applications that benefit from server-side rendering and colocation
  When valid: Appropriate for microservices architectures or when secrets management requires dedicated infrastructure
- Environment variable proxying through API routes only (deferred)
  Rejected because: Can complement this pattern but doesn't eliminate need for server-only module isolation
  When valid: Useful for additional access control and audit logging of secrets usage

## Risks

- Developers may forget to import 'server-only' package in new modules that handle secrets
  Mitigation: Implement linting rules and code review checklists to verify server-only imports; add automated scanning in CI pipeline
  Owner: Engineering team and security team
- Refactoring may accidentally move server-only code into shared modules accessible by client
  Mitigation: Establish clear naming conventions and path structure; use automated tests to verify no server-only imports in client bundles
  Owner: Engineering team
- Third-party dependencies may not respect server-only boundaries
  Mitigation: Carefully vet dependencies that access environment variables; use bundle analysis tools to detect unexpected client-side inclusions
  Owner: Security team and engineering leads

## Implementation Notes

- Create a 'server-only' directory structure under packages/lib/ for all secrets management and privileged operations
- Add 'import "server-only"' as the first import in every module that accesses process.env or handles credentials
- Configure TypeScript path aliases to make server-only imports explicit (e.g., '@/server/*' maps to server-only directories)
- Document the pattern in onboarding materials and architecture guides with clear examples of what belongs in server-only modules
- Set up bundle analysis in CI to detect and alert on unexpected growth in client bundle sizes that might indicate secret leakage

## Continuation Context


Verify commands:
- grep -r "import.*server-only" packages/lib/server-only/ | wc -l
- grep -r "process\.env" packages/ --include="*.ts" --include="*.tsx" | grep -v "server-only" | grep -v "NEXT_PUBLIC" | wc -l
- npm run build && npx @next/bundle-analyzer --analyze

Accept when:
- All files in server-only directories contain 'import "server-only"' statement
- No process.env access for secrets exists outside server-only modules (excluding NEXT_PUBLIC_ prefixed variables)
- Client bundle analysis shows no environment variable access or credential-related code
- Build fails with clear error message when client code attempts to import server-only modules

## Enforcement

- Verified by: Automated linting rules checking for 'server-only' imports in modules accessing process.env
- Verified by: CI pipeline bundle analysis to detect unexpected client-side inclusions
- Verified by: Code review checklist requiring verification of server-only boundaries for security-sensitive changes
- Verified by: Periodic security audits scanning for credential exposure patterns
- Violation handling: Build-time errors prevent deployment when client code imports server-only modules
- Violation handling: Linting violations block PR merges until resolved
- Violation handling: Security team notification for any detected credential exposure in client bundles
- Violation handling: Immediate rollback and incident response for any secrets leaked to production client code
- Exception process: Submit exception request to security team with justification and risk assessment
- Exception process: Document the exception in a central registry with approval date and reviewer
- Exception process: Implement compensating controls (e.g., additional encryption, runtime validation)
- Exception process: Schedule periodic review of all active exceptions to reassess necessity