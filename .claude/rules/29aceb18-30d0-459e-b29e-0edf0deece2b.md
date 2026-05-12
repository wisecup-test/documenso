<rule_activation id="29aceb18-30d0-459e-b29e-0edf0deece2b" title="Enforce Input Validation for Data Access Operations: Functions Modifying Recipients" applies_to="**/*">
These rules are ALWAYS ACTIVE for all data access operations involving user-supplied input parameters, including recipient updates, field modifications, and document operations in server-only modules.
</rule_activation>

### Rules

- **R-VAL-001** MUST: Functions modifying recipients, fields, or document properties MUST verify user authorization before processing input.

### Scope

In scope:
- All server-only data access functions in packages/lib/server-only
- Recipient management operations (create, update, delete)
- Field setting and modification operations
- Document and envelope mutation operations
- Any function accepting user-supplied parameters that interact with the database

Out of scope:
- Read-only query operations that do not modify data
- Internal system operations with hardcoded parameters
- Background jobs with validated input from trusted sources
- Migration scripts and administrative tools

Exceptions:
- EXC-001: Internal administrative functions with pre-validated input from trusted system components

### Verify

```bash
# Count validation/sanitization checks in server-only modules
grep -r 'validate\|sanitize\|check' packages/lib/server-only --include='*.ts' | wc -l

# Verify recipient and field modification functions contain validation
grep -r 'function.*update.*recipient\|function.*set.*field' packages/lib/server-only --include='*.ts' -A 5 | grep -c 'validate\|throw.*Error'

# Check input validation test coverage
npm run test -- --grep 'input validation' --reporter json | jq '.stats.passes'
```

**Accept when:**
- All data access functions in server-only modules contain input validation logic before database operations
- Validation tests exist for each data access function covering valid, invalid, and edge case inputs
- Code review checklist includes verification of input validation for all new data access functions
- No data access functions in packages/lib/server-only lack validation tests

### Implementation Guidance

- Create a centralized validation utility library (e.g., @lib/validation) with reusable validators for common input types
- Implement validation schemas using a library like Zod or Yup for type-safe validation with TypeScript integration
- Add validation checks at the beginning of each data access function before any database operations
- Use parameterized queries or ORM methods in conjunction with input validation for defense-in-depth
- Document validation requirements in function JSDoc comments to guide developers

<enforcement>
Claude Code MUST NOT skip or defer verification of input validation in data access operations. All functions modifying recipients, fields, or document properties MUST include authorization verification and input validation before database operations. Violations must be flagged during code review and CI pipeline checks must fail for non-compliant implementations.
</enforcement>