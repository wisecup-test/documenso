<rule_activation id="0ad2ed4d-08ff-46db-ab5a-1ef858a6e6b5" title="Standardize Structured Logging with Context Propagation for API and Server Operations: Service Clients Making" applies_to="**/*">
These rules are ALWAYS ACTIVE for all API endpoints, server-only functions, and service clients. All new code MUST comply with these logging standards.
</rule_activation>

### Rules

- **R-LOG-001** MUST: Service clients making external calls MUST log request/response metadata including endpoint, status code, and duration.

### Verify

```bash
# Detect console.log/console.error usage — should be 0 or minimal
grep -r 'console.log\|console.error' packages/ --include='*.ts' --include='*.js' | wc -l

# Verify widespread adoption of structured logger
grep -r 'logger\.(info|error|warn|debug)' packages/api packages/lib/server-only --include='*.ts' | wc -l

# Detect sensitive data patterns in logs — should be 0
grep -r 'password\|token\|apiKey' packages/ --include='*.ts' -A 2 -B 2 | grep -i 'log' | wc -l
```

**Accept when:**
- All API endpoints in packages/api contain structured logging calls with request context at entry and exit points
- Server-only functions in packages/lib/server-only log operation start/completion with entity identifiers for business operations
- No instances of console.log/console.error in production code paths (verified by linting rules)
- Automated security scanning detects no sensitive data patterns in log statements
- Log aggregation dashboard shows consistent structured log format across all services with parseable JSON

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations are treated as high-severity issues requiring immediate remediation. CI pipeline fails on detection of console.log/console.error in production code paths. Code review blocks merge if logging standards are not met in new API endpoints or server-only functions.
</enforcement>