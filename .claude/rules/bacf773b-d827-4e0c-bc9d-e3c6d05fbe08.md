<rule_activation id="bacf773b-d827-4e0c-bc9d-e3c6d05fbe08" title="Adopt Server-Only Data Access Layer with Specialized Query Functions: Database Connection Strings" applies_to="**/*">
These rules are ALWAYS ACTIVE for all server-side data access operations. All database queries MUST be encapsulated in dedicated server-only modules following the established pattern.
</rule_activation>

### Rules

- **R-SODAL-001** MUST_NOT: Database connection strings, credentials, or raw SQL queries MUST NOT be exposed outside the server-only data access layer.
- **R-SODAL-002** MUST: All database read operations (SELECT queries) be encapsulated in server-only modules.
- **R-SODAL-003** MUST: All database write operations (INSERT, UPDATE, DELETE) be encapsulated in server-only modules.
- **R-SODAL-004** MUST: All data access requiring authentication or authorization be encapsulated in server-only modules.
- **R-SODAL-005** MUST: Each data access function follow the verb-entity-qualifier naming pattern (get-*, find-*, create-*, update-*, delete-*, *-internal).
- **R-SODAL-006** MUST: Each function export a single default or named function with clear TypeScript types for parameters and return values.
- **R-SODAL-007** SHOULD: Include JSDoc comments documenting authorization requirements, expected errors, and usage examples.
- **R-SODAL-008** SHOULD: Use a shared database client/connection pool imported by all data access functions to ensure consistent configuration.
- **R-SODAL-009** MAY: Allow exceptions only under EXC-001 (database migration scripts) and EXC-002 (emergency hotfix scripts under incident response).

### Verify

```bash
# Count server-only module usage
grep -r 'server-only' --include='*.ts' --include='*.tsx' | grep -v 'node_modules' | wc -l

# Find server-only functions missing exports
find . -path '*/server-only/*' -name '*.ts' | xargs grep -L 'export.*function' | wc -l

# Detect direct database imports in client-side code
grep -r 'import.*prisma\|import.*db' --include='*.tsx' --exclude-dir='server-only' --exclude-dir='api' | grep -v 'node_modules' | wc -l
```

**Accept when:**
- All database access operations are located in server-only directories with no direct database imports in client-side code
- Each data access function follows the verb-entity-qualifier naming pattern and exports a single focused operation
- Code review confirms no database credentials or connection strings are exposed outside the server-only layer
- Automated tests verify that client bundles do not include server-only modules
- No server-only imports appear in client-side TypeScript/TSX files outside designated API directories
- All data access functions include JSDoc comments documenting authorization requirements and expected errors

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures occur for any client-side imports of server-only modules. Code review rejection is mandatory for direct database access outside the server-only layer. Security review is required for any exceptions to this pattern.
</enforcement>