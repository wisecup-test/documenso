<rule_activation id="4c9c674a-83e9-4e6f-9bdd-a4ecfda9d6ce" title="Standardize Public API Route Structure and External Integration Patterns: Authentication Aware Routes" applies_to="**/*">
These rules are ALWAYS ACTIVE for all public-facing API routes, external integrations, and client-facing endpoints. They apply to both REST APIs and server-rendered routes that expose data or functionality to external consumers.
</rule_activation>

### Rules

- **R-AUTH-001** MUST: Authentication-aware routes MUST be grouped under the _authenticated+ namespace or equivalent authentication boundary.
- **R-AUTH-002** MUST: All authenticated routes are organized under _authenticated+ namespace or equivalent authentication boundary.
- **R-AUTH-003** MUST: Server-only utilities are isolated in packages/lib/server-only and not imported by client code.
- **R-AUTH-004** MUST: Multi-tenant routes consistently use URL parameters (orgUrl, teamUrl) for tenant identification.
- **R-AUTH-005** MUST: Implement consistent error response format: { error: string, code: string, details?: object } across all public endpoints.
- **R-AUTH-006** SHOULD: Use Remix route conventions: _authenticated+ for protected routes, _layout.tsx for shared layouts, $ for dynamic parameters.
- **R-AUTH-007** SHOULD: Organize server-only utilities under packages/lib/server-only with clear naming (htmltopdf, pdf) to signal their intended usage.
- **R-AUTH-008** SHOULD: Document all public API routes in OpenAPI/Swagger format, auto-generated from route definitions where possible.
- **R-AUTH-009** SHOULD: Use TypeScript discriminated unions for route parameters to ensure type safety across organization/team/user contexts.
- **R-AUTH-010** MAY: Legacy API endpoints may maintain backward compatibility with existing external consumers (EX-001).
- **R-AUTH-011** MAY: Third-party webhook receivers may conform to external service specifications (EX-002).

### Verify

```bash
# Count authenticated routes
grep -r "routes/_authenticated+" apps/remix/app/routes/ | wc -l

# Check server-only markers
find packages/lib/server-only -name '*.ts' | xargs grep -L 'server-only' | wc -l

# Verify multi-tenant URL patterns
grep -r "\$orgUrl\|\$teamUrl" apps/remix/app/routes/ | wc -l

# Ensure type safety and linting rules pass
npm run type-check && npm run lint
```

**Accept when:**
- All authenticated routes are organized under _authenticated+ namespace or equivalent authentication boundary
- Server-only utilities are isolated in packages/lib/server-only and not imported by client code
- Multi-tenant routes consistently use URL parameters (orgUrl, teamUrl) for tenant identification
- Type checking and linting pass without errors related to route structure or API patterns
- Consistent error response format is implemented across all public endpoints
- All public API routes are documented in OpenAPI/Swagger format or equivalent

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated CI pipeline checks for route structure compliance using custom ESLint rules are mandatory. Pre-commit hooks validate new routes follow naming conventions and authentication patterns. Code review checklist includes verification of API pattern compliance. Quarterly architecture audits review API consistency across the codebase. Violations block PR merge and require architecture review board approval for exceptions.
</enforcement>