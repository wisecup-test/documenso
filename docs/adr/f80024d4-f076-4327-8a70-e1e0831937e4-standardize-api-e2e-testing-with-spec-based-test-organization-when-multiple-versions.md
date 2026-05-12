# Standardize API E2E Testing with Spec-Based Test Organization: When Multiple Versions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application exposes multiple API surfaces including tRPC endpoints and versioned REST APIs (v1, v2) that require comprehensive end-to-end testing
- E2E tests for API endpoints are organized in a dedicated packages/app-tests/e2e/api directory structure with clear separation by API type and version
- Pattern detected across 8 test specification files covering critical API operations including document search, template management, envelope operations, and embedding workflows
- Test organization follows a consistent naming convention (*.spec.ts) and logical grouping by API surface (trpc/, v1/, v2/) to support maintainability and discoverability
- The testing approach supports both legacy (v1) and current (v2) API versions alongside modern tRPC endpoints, indicating a need for structured testing during API evolution

## Problem Statement

As the application evolves with multiple API surfaces (tRPC, REST v1, REST v2) and complex operations (document search, template management, envelope distribution, embedding workflows), there is a need for a standardized approach to organizing and executing end-to-end API tests that ensures comprehensive coverage, maintainability, and clear separation of concerns across different API versions and protocols.

## Decision

1. SHOULD: When multiple API versions exist for the same feature, tests SHOULD be maintained for both versions to ensure backward compatibility

## Policy Block

- SHOULD When multiple API versions exist for the same feature, tests SHOULD be maintained for both versions to ensure backward compatibility

In scope:
- All tRPC API endpoints exposed by the application
- All REST API endpoints in v1 and v2 versions
- End-to-end tests that validate complete API request-response cycles
- Integration tests that verify API behavior with real or test database connections
- Tests for document operations, template management, envelope workflows, and embedding services

Out of scope:
- Unit tests for individual functions or components
- Frontend UI/UX tests that interact with rendered components
- Performance or load testing of API endpoints
- Security penetration testing
- Manual API testing or exploratory testing

## Rationale

- The pattern shows consistent adoption across 8 specification files with 92.27% confidence, indicating this is an established and reliable testing practice
- Organizing tests by API type and version provides clear boundaries that align with the application's API architecture and evolution strategy
- Dedicated spec files per feature enable parallel test execution, easier debugging, and clearer ownership of test maintenance
- The structure supports API versioning strategies by maintaining separate test suites for v1 and v2, ensuring backward compatibility during migrations

## Consequences

Positive:
- Clear, predictable test organization makes it easy for developers to locate and update relevant tests when modifying API endpoints
- Separation by API version enables safe refactoring and migration from v1 to v2 while maintaining test coverage for both
- Consistent naming conventions and directory structure reduce cognitive load and onboarding time for new team members
- Dedicated spec files per feature enable better test isolation, faster test execution through parallelization, and clearer failure diagnostics

Negative:
- Multiple test files for similar functionality across API versions (v1/v2) may lead to duplication of test logic and maintenance overhead
- As the number of API endpoints grows, the flat structure within version directories may become difficult to navigate without further sub-organization
- Strict directory structure may feel rigid for small features or experimental endpoints that don't fit neatly into existing categories
- Maintaining parallel test suites for legacy API versions increases overall test suite size and execution time

## Alternatives

- Organize tests by feature domain (documents/, templates/, envelopes/) rather than by API type (rejected)
  Rejected because: This approach would mix different API protocols and versions within the same directory, making it harder to manage API-specific concerns like versioning, authentication patterns, and protocol-specific testing utilities
  When valid: Could be reconsidered if the application consolidates to a single API surface (e.g., tRPC only) where protocol differences are no longer a primary organizational concern
- Use a single monolithic test file per API version with all endpoints tested together (rejected)
  Rejected because: Monolithic test files become difficult to maintain, slow to execute serially, and create merge conflicts when multiple developers work on different features simultaneously
  When valid: Only appropriate for very small applications with fewer than 5-10 API endpoints where the overhead of multiple files outweighs the benefits
- Co-locate E2E tests with implementation code in the same package/module (rejected)
  Rejected because: E2E tests require different dependencies, execution environments, and deployment considerations than application code; separating them into packages/app-tests provides clearer boundaries
  When valid: May be appropriate for unit or integration tests that are tightly coupled to specific modules, but not for end-to-end API tests that span multiple system components

## Risks

- Test duplication across API versions (v1, v2) leads to maintenance burden and inconsistent test coverage as features evolve
  Mitigation: Create shared test utilities and fixtures that can be reused across versions; establish a deprecation timeline for v1 tests once v2 adoption is complete; use test generators or parameterized tests where appropriate
  Owner: QA Engineering Team
- Flat directory structure within version folders becomes unwieldy as the number of API endpoints grows beyond 20-30 files
  Mitigation: Monitor directory size and introduce sub-categorization (e.g., v2/documents/, v2/templates/) when a version directory exceeds 15-20 spec files; document the sub-organization strategy in testing guidelines
  Owner: Engineering Team
- Developers may be unclear about when to write E2E API tests versus integration or unit tests, leading to inconsistent coverage
  Mitigation: Document clear guidelines on test types and their purposes; provide examples of appropriate E2E test scenarios; include test coverage requirements in code review checklists
  Owner: Engineering Leadership

## Implementation Notes

- When adding a new API endpoint, create a corresponding spec file in the appropriate directory (trpc/, v1/, or v2/) following the {feature-name}.spec.ts naming convention
- Use shared test fixtures and utilities from a common location (e.g., packages/app-tests/e2e/fixtures/) to reduce duplication across spec files
- Ensure each spec file is independently executable and does not rely on execution order or shared state from other spec files
- When deprecating an API version, maintain its test suite until the version is fully removed from production to ensure backward compatibility
- Consider using test tags or metadata to categorize tests by feature area, enabling selective test execution during development

## Continuation Context


Verify commands:
- find packages/app-tests/e2e/api -name '*.spec.ts' -type f | grep -E '(trpc|v1|v2)/' | wc -l
- test -d packages/app-tests/e2e/api/trpc && test -d packages/app-tests/e2e/api/v1 && test -d packages/app-tests/e2e/api/v2
- find packages/app-tests/e2e/api -name '*.spec.ts' | xargs grep -l 'describe\|test\|it' | wc -l

Accept when:
- The packages/app-tests/e2e/api directory exists with subdirectories for trpc/, v1/, and v2/
- All API E2E test files follow the *.spec.ts naming convention and are organized in the appropriate API type subdirectory
- Each spec file contains valid test definitions (describe/test/it blocks) and can be executed independently

## Enforcement

- Verified by: Automated CI pipeline checks that verify test file organization and naming conventions
- Verified by: Code review process ensures new API endpoints include corresponding E2E tests in the correct location
- Verified by: Linting rules or custom scripts that validate test file structure and placement
- Violation handling: CI build fails if E2E tests are placed outside the designated directory structure
- Violation handling: Pull requests are flagged during code review if new API endpoints lack corresponding E2E tests
- Violation handling: Quarterly audits identify and remediate misplaced or missing test files
- Exception process: Exceptions for experimental or internal-only APIs must be documented in a TESTING_EXCEPTIONS.md file with justification and expiration date
- Exception process: Temporary exceptions require approval from QA lead and must include a plan for adding proper E2E tests within one sprint
- Exception process: Legacy endpoints being deprecated may be exempted from new test requirements if deprecation is scheduled within 6 months