<rule_activation id="cdc859dc-7f71-4686-bd6f-ea8e4bfcbcbd" title="Enforce Server-Only Code Isolation for Runtime Environment Boundaries: Client Side Code" applies_to="**/*">
These rules are ALWAYS ACTIVE for all server-side code modules and apply to runtime environment boundary enforcement across the codebase.
</rule_activation>

### Rules

- **R-SOI-001** MUST_NOT: Client-side code MUST NOT directly import from server-only modules; all interactions MUST occur through defined API boundaries (e.g., API routes, server actions, loaders)

### Verify

```bash
# Check for client-side imports of server-only code
grep -r "from.*server-only" apps/*/client apps/*/components --include="*.ts" --include="*.tsx" || echo 'No client imports of server-only code found'

# Verify all server-only files have guards
find packages/lib/server-only -type f -name '*.ts' -exec grep -L "'server-only'" {} \; | wc -l | grep -q '^0$' && echo 'All server-only files have guards'

# Verify build does not bundle server code in client bundles
npm run build 2>&1 | grep -i 'server.*client.*bundle' && exit 1 || echo 'Build verification passed'
```

**Accept when:**
- No client-side code directly imports from server-only directories or modules
- All server-only modules include appropriate runtime guards or build-time enforcement markers
- Build process successfully detects and rejects attempts to bundle server-only code in client bundles
- Code review checklist includes verification of server-only boundary compliance for new modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations block merge to main branch and require security team notification if credentials or secrets are exposed.
</enforcement>