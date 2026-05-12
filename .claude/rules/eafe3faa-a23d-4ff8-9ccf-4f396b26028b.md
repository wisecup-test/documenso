<rule_activation id="eafe3faa-a23d-4ff8-9ccf-4f396b26028b" title="Adopt Server-Only Data Access Layer with Function-Based Repository Pattern: Data Access Functions" applies_to="**/*">
These rules are ALWAYS ACTIVE for all server-side data access operations. All database queries and data mutations MUST be implemented through the server-only data access layer.
</rule_activation>

### Rules

- **R-SODA-001** MUST: Data access functions MUST follow the function-based repository pattern with clear, action-oriented naming (e.g., get-field-by-id, find-folders, update-folder, delete-folder).
- **R-SODA-002** MUST: All database queries and mutations for application data MUST be located in the server-only directory structure with no direct database queries in client-side code.
- **R-SODA-003** MUST: Each data access function MUST be isolated in its own file following the naming convention (get-*, find-*, update-*, delete-*, remove-*).
- **R-SODA-004** MUST: Server-only modules MUST NOT be imported in client-side code, API route handlers, or client components.
- **R-SODA-005** MUST: Build process MUST prevent client-side imports of server-only modules and fail with clear error messages when violations are detected.

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
- No grep results show client-side imports of server-only modules
- Data access function count is greater than zero and follows naming conventions

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All data access operations MUST comply with the server-only pattern before code is considered ready for merge. Build-time enforcement through bundler configuration (Next.js 'server-only' package) is mandatory.
</enforcement>