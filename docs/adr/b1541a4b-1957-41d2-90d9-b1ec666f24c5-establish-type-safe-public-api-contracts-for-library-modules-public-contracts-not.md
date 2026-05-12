# Establish Type-Safe Public API Contracts for Library Modules: Public Contracts Not

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all public API contract definitions and library module interfaces within the codebase.

## Context

- The codebase contains multiple packages (app-tests, api, lib) that expose public APIs and shared functionality requiring clear contract definitions
- API versioning (v1) indicates a need for stable, backward-compatible interfaces that can evolve over time
- Server-only modules and public API endpoints require explicit type contracts to prevent runtime errors and ensure type safety across package boundaries
- OpenAPI specifications and contract definitions suggest a formal approach to API documentation and validation
- Cross-package dependencies necessitate well-defined module boundaries with explicit public interfaces

## Problem Statement

Without standardized public API contracts and explicit type definitions for library modules, the codebase risks interface inconsistencies, breaking changes across package boundaries, runtime type errors, and poor developer experience when consuming shared functionality. The lack of formal contracts makes it difficult to maintain API stability, validate inputs/outputs, and generate accurate documentation.

## Decision

1. MUST_NOT: Public API contracts MUST NOT expose internal implementation details or private types

## Policy Block

- MUST_NOT Public API contracts MUST NOT expose internal implementation details or private types

In scope:
- All packages under /packages/ directory
- Public API endpoints in /packages/api/
- Shared library modules in /packages/lib/
- Test constants and fixtures in /packages/app-tests/
- OpenAPI specifications and contract definitions

Out of scope:
- Internal implementation files not exposed as public APIs
- Private utility functions within modules
- Development-only scripts and tooling
- Third-party library type definitions

Exceptions:
- EXC-001: Legacy APIs undergoing migration to typed contracts
- EXC-002: Experimental or internal-only APIs not yet stabilized

## Rationale

- Pattern detected across 5 files with 87.66% confidence indicates consistent adoption of contract-based API design
- Type-safe contracts prevent runtime errors by catching type mismatches at compile time, reducing production bugs
- Explicit contracts improve developer experience by providing autocomplete, inline documentation, and clear API boundaries
- OpenAPI specifications enable automated documentation generation, client SDK generation, and API validation middleware

## Consequences

Positive:
- Improved type safety across package boundaries reduces runtime errors and improves code reliability
- Clear API contracts facilitate parallel development by establishing stable interfaces between teams
- Automated documentation generation from OpenAPI specs reduces documentation drift and maintenance burden
- Better IDE support with autocomplete and type checking improves developer productivity

Negative:
- Additional upfront effort required to define and maintain contract files alongside implementations
- Breaking changes to contracts require careful versioning and migration strategies
- Increased build complexity with type checking and contract validation steps
- Learning curve for developers unfamiliar with contract-first API design patterns

## Alternatives

- Use runtime validation libraries (Zod, Yup) without static TypeScript contracts (rejected)
  Rejected because: Runtime-only validation misses compile-time type safety benefits and increases runtime overhead
  When valid: Acceptable for dynamic APIs with unknown schemas or user-defined data structures
- Generate TypeScript types from OpenAPI specs using code generation tools (deferred)
  Rejected because: Not rejected but deferred for evaluation as complementary approach
  When valid: Can be adopted alongside manual contracts to ensure OpenAPI spec and types stay synchronized
- Use JSDoc type annotations instead of TypeScript interfaces (rejected)
  Rejected because: JSDoc provides weaker type checking and lacks the expressiveness of TypeScript's type system
  When valid: Only for JavaScript-only codebases without TypeScript support

## Risks

- Contract definitions diverge from actual implementation over time
  Mitigation: Implement automated tests that validate implementation against contracts, use integration tests to verify API behavior
  Owner: Engineering team
- Breaking changes to public contracts impact downstream consumers
  Mitigation: Enforce semantic versioning, maintain changelog, use API versioning (v1, v2) for major changes
  Owner: API team
- Incomplete or poorly documented contracts reduce their effectiveness
  Mitigation: Establish contract review checklist, require JSDoc comments on all public interfaces, use linting rules to enforce documentation
  Owner: Engineering team

## Implementation Notes

- Create contract.ts files in each package's public API directory to centralize type definitions
- Use TypeScript's 'export type' and 'export interface' to explicitly mark public contracts
- Leverage OpenAPI decorators or schema generators to keep OpenAPI specs synchronized with TypeScript types
- Implement pre-commit hooks to validate that all public API functions have corresponding contract definitions
- Document migration path for existing APIs without contracts, prioritizing high-traffic endpoints first

## Continuation Context


Verify commands:
- grep -r "export.*contract" packages/*/contract.ts | wc -l
- find packages/api -name 'openapi.ts' -o -name 'contract.ts' | xargs grep -l 'export'
- tsc --noEmit --strict && echo 'Type checking passed'
- grep -r "server-only" packages/lib | grep -v node_modules

Accept when:
- All public API endpoints have corresponding TypeScript contract definitions with complete type coverage
- TypeScript strict mode compilation passes without type errors in contract files
- OpenAPI specifications exist for versioned API endpoints (v1, v2, etc.)
- Server-only modules are properly isolated and cannot be imported by client-side code

## Enforcement

- Verified by: TypeScript compiler in strict mode during CI builds
- Verified by: ESLint rules enforcing contract file presence for public APIs
- Verified by: Code review checklist requiring contract definitions for new APIs
- Verified by: Automated tests validating implementation matches contract specifications
- Violation handling: CI build fails if TypeScript compilation errors occur in contract files
- Violation handling: Pull requests blocked if new public APIs lack contract definitions
- Violation handling: Automated comments on PRs identifying missing or incomplete contracts
- Violation handling: Quarterly audits to identify and remediate contract gaps in existing code
- Exception process: Submit exception request to tech lead with justification and timeline
- Exception process: Document exception in ADR exceptions log with tracking issue
- Exception process: Set reminder for exception review at next architecture review meeting
- Exception process: Require migration plan for temporary exceptions to become compliant