# Adopt E2E API Testing for Frontend Rendering Model Validation: Tests Validate Response

Status: proposed
Date: 2025-01-10
Deciders: Detection Pipeline (automated)

## Context

- The application requires comprehensive end-to-end testing of API endpoints that serve data to frontend rendering components
- Multiple API versions (v1, v2, tRPC) coexist and must be validated for consistent behavior in supporting UI rendering
- Testing patterns emerged across 8 files covering document search, template management, envelope operations, and embedding workflows
- E2E tests validate the complete request-response cycle including authentication, data transformation, and response formatting required for frontend consumption
- The testing approach ensures API contracts remain stable for frontend rendering logic that depends on specific data structures and response formats

## Problem Statement

Frontend rendering models depend on stable, well-tested API contracts. Without comprehensive E2E API testing, changes to backend services can break frontend rendering logic, cause data transformation issues, or introduce inconsistencies across API versions. The challenge is ensuring all API endpoints that feed frontend rendering components are thoroughly validated for correctness, consistency, and backward compatibility.

## Decision

1. MUST: Tests MUST validate response data structures match the contracts expected by frontend rendering components

## Policy Block

- MUST Tests MUST validate response data structures match the contracts expected by frontend rendering components

In scope:
- All REST API endpoints (v1, v2) that serve frontend applications
- All tRPC procedures that provide data for UI rendering
- API endpoints for document search, retrieval, and management
- API endpoints for template operations and field prefilling
- API endpoints for envelope distribution and validation
- API endpoints for embedding and presigning operations

Out of scope:
- Internal service-to-service APIs not consumed by frontend
- Database integration tests (covered by separate unit/integration testing)
- Frontend component unit tests (covered by component testing strategy)
- Performance load testing (covered by separate performance testing suite)
- Third-party API integrations (covered by integration test suite)

Exceptions:
- EXC-001: API endpoints are deprecated and scheduled for removal within 30 days
- EXC-002: Internal admin APIs with no direct frontend consumption

## Rationale

- Pattern detected across 8 test specification files with 92.27% confidence indicates a consistent, established testing practice
- E2E API testing provides confidence that frontend rendering components receive correctly formatted data across all API versions
- Testing at the API boundary catches integration issues that unit tests miss, including authentication, authorization, and data transformation problems
- Maintaining test coverage across multiple API versions (v1, v2, tRPC) ensures backward compatibility and smooth migration paths for frontend applications

## Consequences

Positive:
- Frontend rendering logic can rely on stable, well-tested API contracts reducing runtime errors
- API changes that break frontend expectations are caught early in the development cycle
- Comprehensive test coverage across API versions enables confident refactoring and version migration
- Clear test specifications serve as living documentation of API behavior and expected response formats

Negative:
- E2E tests are slower to execute than unit tests, potentially increasing CI/CD pipeline duration
- Maintaining E2E tests requires additional effort when API contracts evolve
- Test infrastructure complexity increases with need for test databases, authentication mocking, and environment setup
- Flaky tests due to timing issues or environment dependencies can reduce developer confidence in test suite

## Alternatives

- Contract testing with Pact or similar tools instead of full E2E tests (rejected)
  Rejected because: Contract testing validates schemas but misses integration issues, authentication flows, and actual data transformation logic that E2E tests catch
  When valid: Could be used as a complementary approach for cross-team API contracts where full E2E testing is impractical
- Frontend integration tests that mock API responses (rejected)
  Rejected because: Mocked responses can drift from actual API behavior, creating false confidence and missing real integration issues
  When valid: Appropriate for frontend component testing in isolation, but does not replace API validation
- Manual QA testing of API endpoints (rejected)
  Rejected because: Manual testing is not scalable, not repeatable, and cannot provide continuous validation in CI/CD pipelines
  When valid: Useful for exploratory testing and edge cases not covered by automated tests

## Risks

- E2E test suite becomes slow and blocks rapid development iterations
  Mitigation: Implement parallel test execution, optimize test data setup, and consider tiered testing strategy with smoke tests for quick feedback
  Owner: Engineering team
- Test maintenance burden increases as API surface area grows
  Mitigation: Establish test helpers and shared fixtures, use test generation tools where appropriate, and enforce test-writing as part of API development workflow
  Owner: Engineering team
- Environment-specific issues cause test flakiness and reduce confidence
  Mitigation: Use containerized test environments, implement proper test isolation, add retry logic for known transient failures, and monitor test reliability metrics
  Owner: DevOps and Engineering team

## Implementation Notes

- Organize E2E tests in directory structure matching API versioning: e2e/api/v1/, e2e/api/v2/, e2e/api/trpc/
- Create shared test utilities for common operations like authentication, test data setup, and response validation
- Use descriptive test names that clearly indicate the API endpoint and scenario being tested (e.g., 'search-documents.spec.ts', 'template-field-prefill.spec.ts')
- Implement test data factories to generate consistent, realistic test data for API requests
- Consider using API client libraries in tests to ensure tests exercise the same code paths as production frontend applications
- Document test coverage requirements in API development guidelines and include E2E test creation in definition of done

## Continuation Context


Verify commands:
- find packages/app-tests/e2e/api -name '*.spec.ts' | wc -l | grep -E '^[8-9]|^[1-9][0-9]+$'
- grep -r 'describe\|test\|it' packages/app-tests/e2e/api --include='*.spec.ts' | wc -l
- npm test -- --testPathPattern=e2e/api --listTests | wc -l

Accept when:
- At least 8 E2E API test specification files exist covering major API endpoints
- All API versions (v1, v2, tRPC) have corresponding test coverage for frontend-facing endpoints
- E2E test suite runs successfully in CI/CD pipeline with >90% pass rate
- Test coverage includes document operations, template management, envelope workflows, and search functionality

## Enforcement

- Verified by: CI/CD pipeline runs E2E test suite on every pull request
- Verified by: Code review checklist includes verification of E2E test coverage for API changes
- Verified by: Test coverage reports generated and reviewed in pull request checks
- Verified by: Automated checks verify test file naming conventions and organization structure
- Violation handling: Pull requests without corresponding E2E tests for API changes are blocked from merging
- Violation handling: CI/CD pipeline fails if E2E test suite does not pass
- Violation handling: Test coverage regression triggers automated alerts to engineering team
- Violation handling: Quarterly architecture reviews assess E2E test coverage and identify gaps
- Exception process: Developer submits exception request with justification to engineering lead
- Exception process: Exception request must include alternative validation approach or timeline for adding tests
- Exception process: Engineering lead reviews and approves/rejects within 2 business days
- Exception process: Approved exceptions are documented in ADR exceptions log with expiration date