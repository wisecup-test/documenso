<rule_activation id="109dc3a7-2f0f-4ade-9f34-ae4ec8ae5df6" title="Standardize API E2E Testing with Spec-Based Test Organization: Each Endpoint Feature" applies_to="packages/app-tests/e2e/api/**/*.spec.ts">
These rules are ALWAYS ACTIVE for all API E2E test files in the packages/app-tests/e2e/api directory structure.
</rule_activation>

### Rules

- **R-API-E2E-001** MUST: Each API endpoint or feature MUST have a dedicated spec file that tests its complete functionality end-to-end.
- **R-API-E2E-002** MUST: All API E2E test files MUST follow the `*.spec.ts` naming convention.
- **R-API-E2E-003** MUST: Test files MUST be organized in the appropriate API type subdirectory (trpc/, v1/, or v2/).
- **R-API-E2E-004** MUST: Each spec file MUST be independently executable and not rely on execution order or shared state from other spec files.
- **R-API-E2E-005** MUST: When adding a new API endpoint, create a corresponding spec file in the appropriate directory following the {feature-name}.spec.ts naming convention.
- **R-API-E2E-006** SHOULD: Use shared test fixtures and utilities from a common location (e.g., packages/app-tests/e2e/fixtures/) to reduce duplication across spec files.
- **R-API-E2E-007** SHOULD: When deprecating an API version, maintain its test suite until the version is fully removed from production to ensure backward compatibility.
- **R-API-E2E-008** MAY: Use test tags or metadata to categorize tests by feature area, enabling selective test execution during development.

### Verify

```bash
# Verify spec files exist in correct directory structure
find packages/app-tests/e2e/api -name '*.spec.ts' -type f | grep -E '(trpc|v1|v2)/' | wc -l

# Verify all required API type subdirectories exist
test -d packages/app-tests/e2e/api/trpc && test -d packages/app-tests/e2e/api/v1 && test -d packages/app-tests/e2e/api/v2

# Verify spec files contain valid test definitions
find packages/app-tests/e2e/api -name '*.spec.ts' | xargs grep -l 'describe\|test\|it' | wc -l
```

**Accept when:**
- The packages/app-tests/e2e/api directory exists with subdirectories for trpc/, v1/, and v2/
- All API E2E test files follow the *.spec.ts naming convention and are organized in the appropriate API type subdirectory
- Each spec file contains valid test definitions (describe/test/it blocks) and can be executed independently
- New API endpoints include corresponding E2E tests in the correct location
- Test files are placed within the designated directory structure (packages/app-tests/e2e/api/)

<enforcement>
Claude Code MUST verify test file organization and naming conventions before accepting changes to API E2E tests. CI build MUST fail if E2E tests are placed outside the designated directory structure. Pull requests MUST be flagged during code review if new API endpoints lack corresponding E2E tests. Violations require remediation or documented exceptions approved by QA lead.
</enforcement>