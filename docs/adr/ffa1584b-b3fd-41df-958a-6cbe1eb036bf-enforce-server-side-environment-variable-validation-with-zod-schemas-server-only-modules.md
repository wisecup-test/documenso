# Enforce Server-Side Environment Variable Validation with Zod Schemas: Server Only Modules

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all server-side code that accesses environment variables or configuration values. It applies to API routes, server-only utilities, background jobs, and any code executing in Node.js runtime contexts.

## Context

- The codebase processes sensitive operations including PDF generation, SSO configuration, group management, and API schema validation that depend on environment variables and runtime configuration
- Environment variables are a primary attack vector for configuration injection and can lead to runtime failures if malformed or missing
- The pattern appears consistently across 6 files with 91.63% confidence, indicating a deliberate architectural choice for input validation at the configuration layer
- Server-only modules (pdf generation, field manipulation) and authenticated routes (SSO, groups) require strict validation to prevent security vulnerabilities and runtime errors
- The security.input_validation facet indicates this pattern is part of a broader defense-in-depth strategy for validating external inputs at system boundaries

## Problem Statement

Without structured validation of environment variables and configuration inputs, server-side code is vulnerable to runtime failures, type mismatches, injection attacks, and undefined behavior when configuration is missing or malformed. This is particularly critical in security-sensitive contexts like SSO configuration, PDF generation with user data, and API schema enforcement where invalid configuration could lead to data exposure or system compromise.

## Decision

1. MUST: Server-only modules (marked with 'server-only' package or similar) MUST validate all external configuration inputs including environment variables, file paths, and runtime parameters

## Policy Block

- MUST Server-only modules (marked with 'server-only' package or similar) MUST validate all external configuration inputs including environment variables, file paths, and runtime parameters

In scope:
- All server-side TypeScript/JavaScript code executing in Node.js runtime
- API routes and endpoints that process requests
- Background jobs and scheduled tasks
- Server-only utility modules (PDF generation, field manipulation, etc.)
- Authentication and authorization modules (SSO, groups, permissions)
- Database connection and ORM configuration
- Third-party service integrations requiring API keys or credentials

Out of scope:
- Client-side code running in browsers (different validation requirements)
- Build-time configuration (handled by build tools)
- Development-only scripts and tooling (may use relaxed validation)
- Test fixtures and mock data (may bypass validation for testing purposes)

Exceptions:
- EXC-001: Emergency hotfix requires bypassing validation to restore service
- EXC-002: Prototype or proof-of-concept code in isolated feature branch

## Rationale

- The pattern appears in 6 files with 91.63% confidence across security-sensitive modules (SSO, groups, PDF generation, API schemas), indicating this is an established architectural standard
- Zod provides runtime type safety and validation that TypeScript's compile-time checks cannot provide for environment variables and external configuration
- Early validation at startup prevents cascading failures and provides clear error messages before the application begins processing requests
- The security.input_validation facet classification indicates this pattern is part of a defense-in-depth strategy, treating configuration as untrusted input that must be validated at system boundaries

## Consequences

Positive:
- Prevents runtime errors from missing or malformed environment variables before they cause production incidents
- Provides type safety for configuration throughout the application, enabling IDE autocomplete and compile-time checks
- Reduces security vulnerabilities by validating configuration inputs that could be manipulated in deployment environments
- Improves developer experience with clear error messages when configuration is incorrect during development or deployment
- Enables safe refactoring of configuration-dependent code with confidence that validation will catch breaking changes

Negative:
- Adds boilerplate code for defining and maintaining validation schemas alongside configuration usage
- Increases application startup time slightly due to validation overhead (typically negligible)
- Requires developers to learn Zod schema syntax and validation patterns
- May cause application startup failures in environments with incomplete configuration, requiring careful deployment planning

## Alternatives

- Use TypeScript type assertions without runtime validation (rejected)
  Rejected because: TypeScript types are erased at runtime and provide no protection against invalid environment variables or configuration in production
  When valid: Only acceptable for internal type definitions that don't involve external inputs
- Manual validation with if/else checks and custom error handling (rejected)
  Rejected because: Error-prone, inconsistent across modules, lacks type inference, and difficult to maintain as configuration grows
  When valid: May be acceptable for simple single-value validations in isolated scripts
- Use alternative validation libraries (Joi, Yup, io-ts) (deferred)
  Rejected because: Zod is already established in the codebase; migration would require significant effort without clear benefit
  When valid: Could be reconsidered if Zod proves inadequate for specific validation requirements or if the ecosystem shifts significantly

## Risks

- Application fails to start in production due to overly strict validation that wasn't tested in staging
  Mitigation: Implement comprehensive integration tests that validate configuration in CI/CD pipeline; use staging environments that mirror production configuration
  Owner: DevOps and Engineering Teams
- Developers bypass validation by accessing process.env directly to avoid schema maintenance overhead
  Mitigation: Enforce through linting rules (ESLint) that flag direct process.env access; provide clear documentation and examples for adding new configuration
  Owner: Engineering Team
- Sensitive configuration values (API keys, secrets) may be logged in validation error messages
  Mitigation: Configure Zod schemas to redact sensitive values in error messages; implement structured logging that filters secrets
  Owner: Security and Engineering Teams

## Implementation Notes

- Create a centralized configuration module (e.g., config/env.ts) that exports validated configuration objects using Zod schemas
- Use z.object() to define configuration schemas with explicit types (z.string(), z.number(), z.url(), etc.) and validation rules
- Call schema.parse(process.env) at module initialization to validate and transform environment variables into typed configuration objects
- For server-only modules, import 'server-only' package at the top of files to enforce that configuration validation code never runs on the client
- Use Zod's .transform() and .refine() methods for complex validation logic like URL parsing, enum validation, or cross-field dependencies
- Consider using .env.example files with comments that reference the Zod schema to keep documentation in sync with validation rules

## Continuation Context


Verify commands:
- grep -r "process\.env" --include="*.ts" --include="*.tsx" --exclude-dir=node_modules | grep -v "z\.object\|zod\|schema" | wc -l
- find . -name "*.ts" -path "*/server-only/*" -exec grep -L "import.*zod\|from.*zod" {} \; | wc -l
- npm test -- --testPathPattern="config|env" --coverage

Accept when:
- Grep for direct process.env access returns 0 instances in server-side code (excluding validated configuration modules)
- All server-only modules import and use validated configuration objects rather than accessing environment variables directly
- Configuration validation tests achieve 100% coverage of all environment variables used in production code
- Application startup fails fast with clear error messages when required environment variables are missing or invalid

## Enforcement

- Verified by: ESLint rule to flag direct process.env access outside of designated configuration modules
- Verified by: Code review checklist item requiring validation schemas for any new environment variable usage
- Verified by: CI/CD pipeline integration tests that validate configuration in staging environment before deployment
- Verified by: Static analysis tools scanning for 'server-only' imports and corresponding Zod schema usage
- Violation handling: CI build fails if ESLint detects direct process.env access in non-configuration modules
- Violation handling: Pull requests blocked until code review confirms proper validation for new configuration parameters
- Violation handling: Runtime application startup failure with detailed error message indicating which validation rule was violated
- Violation handling: Security review triggered for any exceptions to validation requirements in sensitive modules (auth, payments, etc.)
- Exception process: Developer documents exception rationale in code comments with reference to exception ID (EXC-001, EXC-002)
- Exception process: Exception requires approval from engineering lead or security team depending on module sensitivity
- Exception process: Exception must include follow-up ticket for adding proper validation with defined timeline
- Exception process: Exceptions are reviewed quarterly to ensure temporary bypasses are resolved or properly documented as permanent exceptions