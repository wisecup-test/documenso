<rule_activation id="bbf7148c-3dea-4bda-aa92-dd3d23fa4e94" title="Standardize Public API Contract Design with Type-Safe Route Handlers and Internationalization: Routes Implement Nested" applies_to="**/*">
These rules are ALWAYS ACTIVE for all Remix route handlers, server-side utilities, and internationalization configurations across the codebase.
</rule_activation>

### Rules

- **R-API-001** MAY: API routes MAY implement nested layouts (_layout.tsx) for shared authentication, authorization, and data loading logic

### Verify

```bash
# Count authenticated route patterns
grep -r '_authenticated+' apps/remix/app/routes/ | wc -l

# Find routes with loader or action functions
find apps/remix/app/routes -name '*.tsx' -exec grep -l 'loader\|action' {} \; | wc -l

# Verify Lingui internationalization configuration
grep -r 'lingui' . --include='*.config.*' --include='*.tsx' | head -5

# Locate PDF generation utilities in server-only packages
find packages/lib/server-only -name '*pdf*.ts' -type f
```

**Accept when:**
- All authenticated routes use _authenticated+ prefix and implement proper authorization checks
- At least 90% of API routes follow the established URL parameter conventions ($orgUrl, $teamUrl)
- Lingui configuration is present and integrated into route handlers for internationalized responses
- PDF generation utilities are isolated in server-only packages with appropriate content-type handling

<enforcement>
Claude Code MUST verify all conditions in the "Accept when" section before approving route implementations. Verification is mandatory and MUST NOT be skipped or deferred.
</enforcement>