<rule_activation id="35cb8fe4-c2c4-438c-ae9a-70caf17a6dba" title="Adopt Monorepo Package Structure with Scoped Modules: Contracts Openapi Specifications" applies_to="**/*">
These rules are ALWAYS ACTIVE for all files in the monorepo to enforce consistent package structure organization, API contract placement, and server-only code isolation.
</rule_activation>

### Rules

- **R-MONO-001** MUST: API contracts and OpenAPI specifications MUST reside in packages/api with versioned subdirectories (e.g., v1/)
- **R-MONO-002** MUST: All TypeScript/JavaScript modules intended for reuse across multiple application components MUST be organized in the packages/ directory structure
- **R-MONO-003** MUST: Server-only utilities including authentication, authorization, and data access logic MUST be placed in packages/lib/server-only
- **R-MONO-004** MUST: No client-side code or components MAY import from packages/lib/server-only
- **R-MONO-005** SHOULD: Use TypeScript path mapping in tsconfig.json to enable clean imports (e.g., '@packages/api/v1/contract' instead of relative paths)
- **R-MONO-006** SHOULD: Each package SHOULD include a README.md file documenting its purpose and scope
- **R-MONO-007** MAY: Experimental features MAY temporarily deviate from package boundaries during rapid prototyping (see EX-002)

### Verify

```bash
# Count TypeScript files in established package structure
find packages/ -type f -name '*.ts' | grep -E 'packages/(api|lib|app-tests)' | wc -l

# Verify no client-side imports of server-only code
grep -r 'packages/lib/server-only' --include='*.ts' --exclude-dir=node_modules | grep -v 'server-only' || echo 'No client-side imports of server-only code'

# Verify API contracts exist in versioned subdirectories
test -d packages/api/v1 && test -f packages/api/v1/openapi.ts && test -f packages/api/v1/contract.ts
```

**Accept when:**
- All TypeScript files under packages/ directory follow the established structure with clear package boundaries (api, lib, app-tests)
- No imports of packages/lib/server-only code exist in client-side bundles or components
- API contracts and OpenAPI specifications are located in packages/api with versioned subdirectories
- Automated dependency graph validation passes without circular dependency errors
- Build-time checks confirm server-only code is not included in client bundles

<enforcement>
Claude Code MUST verify all rules in this activation block. Violations MUST block code generation or modification. Dependency graph analysis and package boundary checks are mandatory on every interaction. Exceptions require explicit documentation and Tech Lead approval.
</enforcement>