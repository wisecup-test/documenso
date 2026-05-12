<rule_activation id="ffa1584b-b3fd-41df-958a-6cbe1eb036bf" title="Enforce Server-Side Environment Variable Validation with Zod Schemas: Server Only Modules" applies_to="**/*">
These rules are ALWAYS ACTIVE for all server-side code that accesses environment variables or configuration values, including API routes, server-only utilities, background jobs, and any code executing in Node.js runtime contexts.
</rule_activation>

### Rules

- **R-ENV-001** MUST: Server-only modules (marked with 'server-only' package or similar) MUST validate all external configuration inputs including environment variables, file paths, and runtime parameters using Zod schemas.
- **R-ENV-002** MUST: Create a centralized configuration module (e.g., config/env.ts) that exports validated configuration objects using Zod schemas.
- **R-ENV-003** MUST: Use z.object() to define configuration schemas with explicit types (z.string(), z.number(), z.url(), etc.) and validation rules.
- **R-ENV-004** MUST: Call schema.parse(process.env) at module initialization to validate and transform environment variables into typed configuration objects.
- **R-ENV-005** MUST: Import 'server-only' package at the top of server-only configuration files to enforce that validation code never runs on the client.
- **R-ENV-006** SHOULD: Use Zod's .transform() and .refine() methods for complex validation logic like URL parsing, enum validation, or cross-field dependencies.
- **R-ENV-007** SHOULD: Use .env.example files with comments that reference the Zod schema to keep documentation in sync with validation rules.
- **R-ENV-008** MUST: Configure Zod schemas to redact sensitive values in error messages to prevent API keys and secrets from being logged.

### Verify

```bash
# Check for direct process.env access outside of designated configuration modules
grep -r "process\.env" --include="*.ts" --include="*.tsx" --exclude-dir=node_modules | grep -v "z\.object\|zod\|schema" | wc -l

# Find server-only modules missing Zod imports
find . -name "*.ts" -path "*/server-only/*" -exec grep -L "import.*zod\|from.*zod" {} \;

# Run configuration and environment validation tests
npm test -- --testPathPattern="config|env" --coverage
```

**Accept when:**
- Grep for direct process.env access returns 0 instances in server-side code (excluding validated configuration modules)
- All server-only modules import and use validated configuration objects rather than accessing environment variables directly
- Configuration validation tests achieve 100% coverage of all environment variables used in production code
- Application startup fails fast with clear error messages when required environment variables are missing or invalid
- ESLint detects no violations of direct process.env access in non-configuration modules
- All new environment variable usage includes corresponding Zod validation schemas

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this activation are mandatory for server-side code. Violations must be caught during code review and CI/CD pipeline checks before deployment.
</enforcement>