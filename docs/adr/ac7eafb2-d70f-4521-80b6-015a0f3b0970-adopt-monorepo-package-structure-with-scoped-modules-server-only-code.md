# Adopt Monorepo Package Structure with Scoped Modules: Server Only Code

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase exhibits a consistent pattern of organizing code into scoped packages under a 'packages/' directory structure, with distinct modules for app-tests, api, and lib components
- Multiple files demonstrate separation of concerns through dedicated packages: packages/app-tests for testing constants, packages/api for API contracts and OpenAPI specifications, and packages/lib for server-only utilities
- The pattern shows clear architectural boundaries between public API functionality (get-user-by-token), organizational logic (get-organisation-claims), and API contract definitions (openapi.ts, contract.ts)
- This structure supports code reusability and modularity across different application layers while maintaining clear dependency boundaries
- The detection of this pattern across 5 files with 87.66% confidence indicates a deliberate architectural choice rather than ad-hoc organization

## Problem Statement

As the application grows in complexity, managing dependencies, ensuring code reusability, and maintaining clear architectural boundaries becomes increasingly difficult without a structured approach to organizing libraries and modules. A consistent package structure is needed to prevent circular dependencies, enable independent testing, and facilitate team collaboration across different functional domains.

## Decision

1. MUST: Server-only code MUST be isolated in packages/lib/server-only to prevent accidental client-side imports

## Policy Block

- MUST Server-only code MUST be isolated in packages/lib/server-only to prevent accidental client-side imports

In scope:
- All TypeScript/JavaScript modules intended for reuse across multiple application components
- API contract definitions, OpenAPI specifications, and versioned API interfaces
- Server-only utilities including authentication, authorization, and data access logic
- Test constants, fixtures, and shared testing utilities
- Shared type definitions and interfaces used across packages

Out of scope:
- Application-specific business logic that belongs to a single service or application
- Third-party dependencies managed through package.json
- Build artifacts and compiled output
- Environment-specific configuration files
- Documentation and non-code assets unless they are package-specific

Exceptions:
- EX-001: Legacy code migration is in progress and immediate refactoring would block critical features
- EX-002: Experimental features require rapid prototyping before architectural boundaries are clear

## Rationale

- The detected pattern across 5 files with 87.66% confidence demonstrates an established architectural convention that should be formalized and enforced
- Separating server-only code (packages/lib/server-only) prevents security vulnerabilities from accidentally exposing server logic to client bundles
- Organizing API contracts in a dedicated package (packages/api) enables contract-first development and facilitates API versioning strategies
- The monorepo package structure enables independent testing, versioning, and potential extraction of packages as standalone libraries if needed

## Consequences

Positive:
- Clear architectural boundaries reduce cognitive load and make it easier for developers to locate and understand code organization
- Server-only code isolation prevents accidental client-side imports, reducing bundle size and improving security
- Independent package structure enables parallel development by multiple teams without conflicts
- Facilitates code reuse across different applications within the monorepo while maintaining clear dependency graphs

Negative:
- Additional overhead in setting up new packages and maintaining package.json configurations for each module
- Potential for over-engineering if package boundaries are created prematurely before patterns stabilize
- Requires discipline and code review to ensure developers place code in the correct package
- May increase build complexity and require tooling configuration for monorepo management (e.g., workspace setup)

## Alternatives

- Flat src/ directory structure with all code in a single location organized by feature (rejected)
  Rejected because: Does not provide clear boundaries between server-only code, API contracts, and test utilities, leading to potential security issues and larger client bundles
  When valid: Only suitable for very small applications with minimal complexity and no server/client separation concerns
- Separate repositories for each package with independent versioning and deployment (rejected)
  Rejected because: Increases coordination overhead, makes cross-package refactoring difficult, and complicates local development setup
  When valid: When packages are truly independent products with different release cycles and external consumers
- Layer-based organization (controllers/, services/, repositories/) without package scoping (rejected)
  Rejected because: Focuses on technical layers rather than functional boundaries, making it harder to understand feature scope and dependencies
  When valid: In traditional MVC applications where layer separation is the primary architectural concern

## Risks

- Developers may not understand package boundaries and place code in incorrect locations, degrading the architecture over time
  Mitigation: Provide clear documentation, enforce through linting rules, and include package structure review in code review checklist
  Owner: Engineering Team Lead
- Circular dependencies between packages may emerge as the codebase grows, causing build failures or runtime issues
  Mitigation: Implement automated dependency graph validation in CI pipeline and use tools like madge or dependency-cruiser
  Owner: DevOps Team
- Package boundaries may become too granular, leading to excessive indirection and difficulty navigating the codebase
  Mitigation: Establish clear criteria for when to create new packages (minimum 3 consumers or clear domain boundary) and review package structure quarterly
  Owner: Architecture Team

## Implementation Notes

- Use TypeScript path mapping in tsconfig.json to enable clean imports (e.g., '@packages/api/v1/contract' instead of relative paths)
- Configure build tools (webpack, vite, etc.) to recognize the packages/ directory structure and handle server-only code exclusion
- Establish naming conventions for packages: packages/api for public contracts, packages/lib for shared utilities, packages/app-* for application-specific code
- Consider using tools like Turborepo, Nx, or Lerna to manage monorepo builds, caching, and dependency orchestration
- Document the purpose and scope of each package in its own README.md file to guide developers

## Continuation Context


Verify commands:
- find packages/ -type f -name '*.ts' | grep -E 'packages/(api|lib|app-tests)' | wc -l
- grep -r 'packages/lib/server-only' --include='*.ts' --exclude-dir=node_modules | grep -v 'server-only' || echo 'No client-side imports of server-only code'
- test -d packages/api/v1 && test -f packages/api/v1/openapi.ts && test -f packages/api/v1/contract.ts

Accept when:
- All TypeScript files under packages/ directory follow the established structure with clear package boundaries (api, lib, app-tests)
- No imports of packages/lib/server-only code exist in client-side bundles or components
- API contracts and OpenAPI specifications are located in packages/api with versioned subdirectories
- Automated dependency graph validation passes without circular dependency errors

## Enforcement

- Verified by: Automated linting rules (ESLint with import restrictions) in CI pipeline
- Verified by: Code review checklist includes verification of correct package placement
- Verified by: Dependency graph analysis tools run on every pull request
- Verified by: Build-time checks ensure server-only code is not included in client bundles
- Violation handling: CI pipeline fails if imports violate package boundaries (e.g., client code importing server-only modules)
- Violation handling: Pull requests are blocked until code is moved to appropriate package location
- Violation handling: Automated comments on PRs highlight violations with suggestions for correct placement
- Violation handling: Quarterly architecture reviews identify and prioritize refactoring of violations that slipped through
- Exception process: Developer documents the exception reason and proposes alternative approach in PR description
- Exception process: Tech Lead or Architecture Team reviews and approves/rejects with written justification
- Exception process: Approved exceptions are documented in a central EXCEPTIONS.md file with expiration date
- Exception process: Exceptions are reviewed monthly and must be resolved or re-justified