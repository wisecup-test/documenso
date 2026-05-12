<rule_activation id="44ceb7f6-e9f2-4ead-817a-cab8e8ef2cdf" title="Standardize Cache Layer Configuration for Runtime Data Management: Critical Paths Audit" applies_to="**/*">
These rules are ALWAYS ACTIVE for all runtime components that manage data caching, including API handlers, route handlers, and data fetching utilities. All new cache implementations MUST follow these patterns.
</rule_activation>

### Rules

- **R-37-004** MUST: Enforce TTL configuration for all cached data to prevent unbounded cache growth and enable predictable expiration.
- **R-37-006** MUST: Enforce co-location of cache invalidation with mutations to prevent stale data from being served after updates.
- **R-37-001** SHOULD: Critical paths (audit logs, webhooks, documents) SHOULD implement cache warming strategies to prevent cold-start latency.

### Scope

**In scope:**
- All route handlers in `apps/remix/app/routes` that fetch external data
- API utility functions in `packages/trpc/utils` that handle data fetching
- Internal routes for PDF generation and audit logging
- Authenticated routes for team settings, webhooks, and document management
- OpenAPI fetch handlers and tRPC procedure implementations

**Out of scope:**
- Static asset caching handled by CDN or reverse proxy layers
- Browser-side caching controlled by HTTP headers
- Database query result caching managed by ORM or database layer
- Third-party service SDK internal caching mechanisms

**Exceptions:**
- EXC-001: Real-time data requirements prohibit any caching (e.g., live collaboration features, real-time notifications)
- EXC-002: Prototype or experimental features in development branches

### Verify

```bash
# Count cache configuration patterns in route handlers and utilities
grep -r 'cache.*config\|cacheConfig\|Cache.*Options' apps/remix/app/routes packages/trpc/utils --include='*.ts' --include='*.tsx' | wc -l

# Count TTL and maxAge patterns indicating explicit cache configuration
grep -r 'TTL\|timeToLive\|maxAge' apps/remix/app/routes packages/trpc/utils --include='*.ts' --include='*.tsx' | grep -v node_modules | wc -l

# Count cache invalidation patterns in mutation operations
grep -r 'invalidate.*cache\|cache.*invalidate\|revalidate' apps/remix/app/routes packages/trpc/utils --include='*.ts' --include='*.tsx' | wc -l
```

**Accept when:**
- All route handlers and API utilities that fetch data have explicit cache configuration declarations (grep count > 0 for cache config patterns)
- Cache invalidation logic is present alongside mutation operations (grep count shows invalidation patterns in mutation files)
- TTL configuration is explicitly set for cached data rather than using framework defaults (grep count shows TTL patterns in cache configurations)
- Code review checklist includes verification of cache key uniqueness and invalidation strategy
- Common cache TTL patterns are documented for different data types (user profiles: 5min, team settings: 1min, audit logs: 30sec, documents list: 2min)
- Cache metrics are integrated into observability stack with dashboards for hit rate, miss rate, and latency by cache type

<enforcement>
Claude Code MUST verify cache configuration compliance before approving changes to data-fetching code. Verification is mandatory via automated CI checks and manual code review. Runtime monitoring alerts for cache hit rate below 70% indicate misconfiguration requiring investigation. Violations result in CI warnings (non-blocking) for missing configuration and code review rejection for mutations without invalidation logic.
</enforcement>