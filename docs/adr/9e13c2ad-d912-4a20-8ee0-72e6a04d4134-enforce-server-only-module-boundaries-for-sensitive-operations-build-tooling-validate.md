# Enforce Server-Only Module Boundaries for Sensitive Operations: Build Tooling Validate

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all server-side code modules handling sensitive operations, data access patterns, and environment-specific functionality. It applies to all new and existing server-only modules.

## Context

- The codebase contains modules explicitly marked as 'server-only' (packages/lib/server-only/*) indicating a deliberate architectural boundary between client and server execution contexts
- Sensitive operations such as PDF generation, field manipulation, and audit logging require server-side execution to protect business logic and prevent client-side tampering
- Modern full-stack frameworks (Next.js, Remix) enable code sharing between client and server, creating risk of accidental client-side exposure of server-only logic
- The pattern appears consistently across 5 files with 91.12% confidence, suggesting an established architectural convention
- API implementation and authenticated routes demonstrate integration points where server-only modules are consumed safely

## Problem Statement

Without explicit enforcement of server-only execution boundaries, sensitive business logic, database access patterns, and privileged operations risk being bundled into client-side JavaScript, exposing security vulnerabilities, intellectual property, and creating performance issues from unnecessary code bloat in browser bundles.

## Decision

1. SHOULD: Build tooling SHOULD validate that server-only modules are not included in client bundles and fail builds on violations

## Policy Block

- SHOULD Build tooling SHOULD validate that server-only modules are not included in client bundles and fail builds on violations

In scope:
- All database access and ORM operations
- PDF generation and document processing logic
- Audit logging and compliance tracking
- Field manipulation and template processing with business rules
- Authentication and authorization enforcement logic
- API route handlers and server-side data fetching
- Environment variable access and configuration management
- Third-party service integrations requiring API keys or secrets

Out of scope:
- Client-side UI components and presentation logic
- Browser-based form validation (non-authoritative)
- Client-side routing and navigation
- Public API client libraries and SDK wrappers
- Shared type definitions and interfaces without implementation
- Pure utility functions with no environment dependencies

Exceptions:
- EXC-001: Shared utility functions that are truly environment-agnostic (e.g., date formatting, string manipulation) may exist outside server-only boundaries
- EXC-002: Type definitions and interfaces may be shared between client and server when they contain no implementation logic

## Rationale

- The consistent pattern across 5 files (91.12% confidence) demonstrates this is an established architectural principle, not an accident
- Server-only boundaries prevent security vulnerabilities by ensuring sensitive logic never reaches the client where it can be inspected, modified, or bypassed
- Explicit separation improves code organization and makes it immediately clear which modules have privileged access and which are safe for client execution
- Modern bundlers and frameworks support server-only markers, making enforcement practical without significant development overhead

## Consequences

Positive:
- Enhanced security posture by preventing accidental exposure of sensitive business logic, database queries, and API keys to client bundles
- Reduced client bundle size by excluding server-only code from browser downloads, improving page load performance
- Clearer architectural boundaries make codebase easier to understand and reduce cognitive load for developers
- Build-time validation catches violations early in development cycle before they reach production

Negative:
- Requires discipline and awareness from developers to correctly classify modules as server-only vs. shared
- May create initial friction when refactoring existing code that violates boundaries
- Additional build tooling and configuration needed to enforce boundaries automatically
- Potential for over-classification where truly safe utilities are unnecessarily restricted to server-only

## Alternatives

- Use a single codebase without explicit server/client boundaries, relying on developer discipline and code review (rejected)
  Rejected because: Manual enforcement is error-prone and doesn't scale; security vulnerabilities from accidental client exposure are too high-risk to rely on human vigilance alone
  When valid: Only appropriate for small teams with very simple applications and no sensitive operations
- Separate server and client code into completely different repositories or packages with no shared code (rejected)
  Rejected because: Creates excessive duplication of types, interfaces, and truly shared utilities; increases maintenance burden and reduces developer velocity
  When valid: May be appropriate for microservices architectures with strict service boundaries and different teams
- Use runtime environment checks (if (typeof window === 'undefined')) within shared modules (rejected)
  Rejected because: Still bundles server code into client builds (bloat), makes code harder to reason about, and provides no build-time safety guarantees
  When valid: Acceptable for small utility functions where the overhead is minimal and both implementations are intentionally different

## Risks

- Developers unfamiliar with the pattern may accidentally import server-only modules in client code, causing build failures or runtime errors
  Mitigation: Provide clear documentation, onboarding materials, and IDE/linting configuration that warns on invalid imports; use the 'server-only' npm package for runtime guards
  Owner: Engineering team + Developer Experience team
- Overly aggressive classification may force unnecessary code duplication when safe utilities are marked server-only
  Mitigation: Establish clear guidelines for what constitutes server-only vs. shared code; review exceptions process regularly to identify patterns that should be shared
  Owner: Architecture review team
- Framework or bundler updates may change how server-only boundaries are enforced, breaking existing patterns
  Mitigation: Pin framework versions, test upgrades thoroughly in staging, maintain automated tests that verify server-only modules are not in client bundles
  Owner: Platform engineering team

## Implementation Notes

- Install and import the 'server-only' npm package at the top of server-only modules to get runtime errors if accidentally imported client-side
- Configure bundler (Webpack, Vite, etc.) to fail builds if server-only paths are detected in client bundle analysis
- Use directory structure conventions: place server-only code in /server-only/, /api/, or similar directories; configure ESLint to restrict imports from these paths in client code
- For Next.js: leverage server components and server actions; for Remix: use loaders and actions; ensure framework-specific patterns align with this ADR
- Add pre-commit hooks or CI checks that scan for common violations (e.g., database imports in client directories)

## Continuation Context


Verify commands:
- grep -r "from.*server-only" apps/*/client apps/*/components --include="*.ts" --include="*.tsx" && echo "VIOLATION: Client code importing server-only modules" || echo "PASS"
- npm run build && npx bundle-analyzer --check-server-only || echo "Verify client bundles do not contain server-only code"
- find . -path '*/server-only/*' -name '*.ts' -exec grep -L "'server-only'" {} \; | head -5 && echo "WARNING: Server-only files missing runtime guard" || echo "PASS"

Accept when:
- All modules in server-only directories include 'server-only' package import or equivalent framework marker
- Build process successfully excludes server-only modules from client bundles (verified via bundle analysis)
- No client-side code directly imports from server-only paths (verified via static analysis or linting)
- CI pipeline includes automated checks that fail on server-only boundary violations

## Enforcement

- Verified by: Automated CI checks scanning for server-only imports in client code paths
- Verified by: Bundle analysis tools verifying client bundles do not contain server-only module code
- Verified by: ESLint rules with import restrictions configured for client-side directories
- Verified by: Code review checklist items for new modules requiring classification as server-only or shared
- Violation handling: Build failures block PR merges when server-only code is detected in client bundles
- Violation handling: Linting errors require resolution before code review approval
- Violation handling: Runtime errors (via 'server-only' package) caught in development and staging environments
- Violation handling: Security review triggered for any violations that reach production
- Exception process: Developer documents rationale for exception in PR description with security implications analysis
- Exception process: Tech lead or architect reviews and approves exception with explicit sign-off
- Exception process: Exception documented in code comments with ADR reference and approval date
- Exception process: Exceptions logged in architecture decision log and reviewed quarterly for patterns