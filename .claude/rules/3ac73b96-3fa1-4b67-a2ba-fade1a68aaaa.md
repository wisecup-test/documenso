<rule_activation id="3ac73b96-3fa1-4b67-a2ba-fade1a68aaaa" title="Adopt Server-Only Data Access Layer with Function-Based Repository Pattern: Database Access Operations" applies_to="**/*">
These rules are ALWAYS ACTIVE for all server-side data access operations. All database queries and data mutations MUST be implemented through the server-only data access layer.
</rule_activation>

### Rules

- **R-SODA-001** MUST: All database access operations MUST be implemented in the server-only directory structure and marked with server-only module boundaries.
- **R-SODA-002** MUST: All database queries and mutations for application data MUST use the server-only data access layer.
- **R-SODA-003** MUST: CRUD operations for domain entities (fields, folders, API tokens, etc.) MUST follow the function-based repository pattern.
- **R-SODA-004** MUST: Server-side API route handlers and server components MUST access data only through the server-only layer.
- **R-SODA-005** MUST: Background jobs and scheduled tasks requiring data access MUST use server-only data access functions.
- **R-SODA-006** MUST: Authentication and authorization checks requiring database queries MUST be implemented in the server-only layer.
- **R-SODA-007** SHOULD: Data access functions SHOULD follow naming conventions (get-*, find-*, update-*, delete-*, remove-*) and be isolated in individual files.
- **R-SODA-008** SHOULD: Each data access function SHOULD be organized by domain entity in subdirectories with clean import paths via index files.
- **R-SODA-009** MAY: Database migration scripts and seed data operations (EXC-001) and development/testing utilities (EXC-002) may access the database directly in isolated contexts.

### Verify

```bash
# Check for client-side imports of server-only modules
grep -r 'from.*server-only' --include='*.tsx' --include='*.ts' apps/*/app/**/page.tsx apps/*/app/**/layout.tsx | grep -v '.server.' || echo 'No client-side imports of server-only modules detected'

# Count data access functions following naming convention
find packages/lib/server-only -type f -name '*.ts' | xargs -I {} basename {} | grep -E '^(get|find|update|delete|remove)-.*\.ts$' | wc -l

# Verify server-only directory exists and contains exported functions
test -d packages/lib/server-only && find packages/lib/server-only -type f -name '*.ts' -exec grep -l 'export.*function' {} \; | wc -l
```

**Accept when:**
- All database access operations are located in the server-only directory structure with no direct database queries in client-side code
- Each data access function follows the naming convention (get-*, find-*, update-*, delete-*, remove-*) and is isolated in its own file
- Build process successfully prevents client-side imports of server-only modules and fails with clear error messages when violations are detected
- Code review checklist includes verification that new data access operations follow the established pattern
- No server-only module imports are detected in client-side page.tsx or layout.tsx files

<enforcement>
Claude Code MUST verify all rules in this activation block. Build-time enforcement through bundler configuration (Next.js 'server-only' package) is mandatory. ESLint rules checking for server-only module imports in client code must pass. Violations result in blocking build failures. Code review rejection is required for data access operations that do not follow the established pattern.
</enforcement>