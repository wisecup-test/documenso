<rule_activation id="ade8ade1-27da-4b7c-b8eb-953891edca4f" title="Standardize Environment Variable Configuration Management Pattern: Applications Implement Configuration" applies_to="**/*">
These rules are ALWAYS ACTIVE for all server-side application code requiring runtime configuration, including API implementations, service integrations, PDF processing utilities, database connections, feature flags, and file system paths.
</rule_activation>

### Rules

- **R-CONFIG-001** MAY: Applications MAY implement configuration hot-reloading for non-critical settings that benefit from runtime updates without restart.

### Scope

**In scope:**
- All server-side application code requiring runtime configuration
- API implementations and service integrations
- PDF processing utilities and document generation services
- Database connection strings and service endpoints
- Feature flags and environment-specific behavior toggles
- File system paths and external resource locations

**Out of scope:**
- Build-time constants that never change across environments
- Client-side configuration exposed to browsers (use separate public config mechanism)
- Configuration for development tools and build scripts (may use simpler patterns)
- Test fixtures and mock data (may use inline values for clarity)

**Exceptions:**
- EXC-001: Legacy code modules scheduled for deprecation within 6 months
- EXC-002: Prototype or proof-of-concept code not intended for production deployment

### Verify

```bash
# Count direct process.env access outside configuration modules
grep -r 'process\.env\.' --include='*.ts' --include='*.js' packages/ | grep -v 'config\|environment' | wc -l

# Find centralized configuration modules
find packages/ -name 'config.ts' -o -name 'environment.ts' -o -name 'env.ts' | head -5

# Count configuration imports in API and library code
grep -r 'import.*config.*from' --include='*.ts' packages/api packages/lib | wc -l
```

**Accept when:**
- Direct process.env access is minimal or zero outside of dedicated configuration modules
- At least one centralized configuration module exists that exports typed configuration objects
- Configuration imports are present in API and library code indicating usage of centralized configuration

### Implementation Guidance

- Create a centralized configuration module (e.g., config.ts or environment.ts) that exports typed configuration objects with validation using libraries like zod or joi
- Use environment variable naming conventions (e.g., APP_NAME_FEATURE_SETTING) to avoid conflicts and improve clarity
- Implement configuration loading at application entry point with clear error messages for missing or invalid values
- Provide .env.example files in the repository documenting all required and optional configuration variables with descriptions
- Consider using a configuration library like dotenv for local development and ensure it's only loaded in non-production environments
- Document the configuration schema in README or dedicated configuration documentation with examples for each environment

<enforcement>
Claude Code MUST verify that new configuration implementations follow the centralized pattern. Code review checklist MUST include verification that configuration follows this pattern. ESLint rules MUST be configured to warn or error on direct process.env access outside configuration modules. CI pipeline MUST include verification commands to detect violations. Pull requests with direct environment variable access outside configuration modules MUST be flagged in code review. Linting violations MUST be resolved before merge to main branches.
</enforcement>