# Adopt Remix File-Based Routing with URL Parameter Conventions for Authenticated Routes: Admin Routes Prefixed

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The application uses Remix framework's file-based routing system to define authenticated routes with dynamic URL parameters for organization and team contexts
- Multiple route files follow a consistent naming pattern (_authenticated+) indicating a shared authentication boundary for all routes within this segment
- Routes are organized hierarchically with URL parameters ($orgUrl, $teamUrl) that provide context-specific resource access for organizations and teams
- The pattern appears across 9 files handling settings, members, teams, groups, templates, folders, documents, and admin functionality, indicating a standardized approach to route organization
- This routing structure serves as the public API surface for the web application, defining how external users interact with organizational and team resources

## Problem Statement

How should we structure and organize authenticated routes in a Remix application to provide consistent, context-aware access to organizational and team resources while maintaining clear API boundaries and predictable URL patterns for external users?

## Decision

1. SHOULD: Admin routes SHOULD be prefixed with admin+ to clearly distinguish administrative functionality from user-facing routes

## Policy Block

- SHOULD Admin routes SHOULD be prefixed with admin+ to clearly distinguish administrative functionality from user-facing routes

In scope:
- All authenticated web application routes in the Remix application
- Organization-scoped resource access patterns
- Team-scoped resource access patterns
- Administrative interface routes
- Settings, members, teams, groups, templates, folders, and document management routes

Out of scope:
- Unauthenticated public routes
- API routes that don't serve HTML (REST/GraphQL endpoints)
- Static asset routes
- Authentication flow routes (login, signup, password reset)
- Third-party integrations or webhooks

Exceptions:
- EX-001: Legacy routes that predate this convention and require backward compatibility
- EX-002: Experimental features under feature flags that may use alternative routing patterns

## Rationale

- Pattern detected across 9 files with 90% confidence and 90% significance, indicating this is an established and intentional architectural decision
- Remix's file-based routing provides type-safe URL parameter extraction and automatic code splitting, improving developer experience and application performance
- Consistent URL parameter conventions ($orgUrl, $teamUrl) create predictable API patterns that external users and integrations can rely on
- The _authenticated+ boundary provides a clear security perimeter, ensuring all routes within this segment enforce authentication requirements automatically

## Consequences

Positive:
- Predictable and consistent URL structure makes the application easier to navigate and understand for both developers and end users
- File-based routing provides automatic code splitting and optimized bundle sizes for each route
- Type-safe parameter extraction reduces runtime errors and improves developer productivity
- Clear authentication boundaries reduce the risk of accidentally exposing protected resources
- Parallel naming structures (org vs team routes) make it easy to find and maintain related functionality

Negative:
- File naming conventions with special characters (+, $, .) may be unfamiliar to developers new to Remix
- Refactoring route structures requires file renames which can be disruptive to version control history
- Deep nesting of routes can lead to long file paths that may exceed filesystem limits on some operating systems
- Tight coupling to Remix framework makes migration to other frameworks more difficult

## Alternatives

- Use a centralized route configuration file (like React Router's route config object) instead of file-based routing (rejected)
  Rejected because: Loses automatic code splitting benefits, requires manual route-to-component mapping, and doesn't leverage Remix's conventions
  When valid: When framework-agnostic routing is required or when migrating from a non-Remix application
- Use path-based parameters (/org/:orgId/) instead of URL-friendly slugs ($orgUrl) (rejected)
  Rejected because: Numeric IDs are less user-friendly in URLs and don't provide SEO benefits; URL slugs are more readable and shareable
  When valid: For internal admin tools where URL readability is not a priority
- Flatten route structure by removing nested segments and using query parameters for context (rejected)
  Rejected because: Query parameters don't provide the same hierarchical clarity and are harder to reason about for nested resources; also loses Remix layout nesting benefits
  When valid: For simple applications with minimal resource hierarchy

## Risks

- Route naming conventions may diverge over time as different developers add new routes without following established patterns
  Mitigation: Implement linting rules to enforce file naming conventions and conduct code reviews focused on route structure consistency
  Owner: Engineering team
- URL parameter collisions could occur if $orgUrl and $teamUrl values overlap or are not properly scoped
  Mitigation: Implement loader validation to ensure URL parameters resolve to the correct resource type and return 404 for invalid combinations
  Owner: Engineering team
- Deep route nesting may lead to performance issues with excessive layout re-renders
  Mitigation: Use Remix's shouldRevalidate function to optimize data fetching and prevent unnecessary re-renders; monitor route performance metrics
  Owner: Engineering team

## Implementation Notes

- Use Remix's useParams() hook to extract $orgUrl and $teamUrl parameters in loaders and components
- Implement shared layout components at the _authenticated+, o.$orgUrl, and t.$teamUrl+ boundaries to provide consistent navigation and context
- Create utility functions for generating type-safe route URLs (e.g., orgSettingsUrl(orgUrl, 'members')) to avoid hardcoding paths
- Document the route structure in a central location (e.g., ROUTES.md) with examples of each pattern and when to use them
- Consider using a route manifest generator to automatically create TypeScript types for all routes and their parameters

## Continuation Context


Verify commands:
- find apps/remix/app/routes/_authenticated+ -name '*.tsx' | grep -E 'o\.\$orgUrl|t\.\$teamUrl' | wc -l
- grep -r 'export.*loader' apps/remix/app/routes/_authenticated+ | grep -c 'params'
- find apps/remix/app/routes/_authenticated+ -name '*_index.tsx' | wc -l

Accept when:
- All authenticated routes are located within the _authenticated+ directory segment
- Organization routes use o.$orgUrl pattern and team routes use t.$teamUrl+ pattern consistently
- Index routes use _index.tsx suffix and layout boundaries use + suffix appropriately
- Route loaders properly extract and validate URL parameters before accessing resources

## Enforcement

- Verified by: Code review checklist includes verification of route naming conventions
- Verified by: ESLint custom rules to validate file naming patterns in the routes directory
- Verified by: CI pipeline checks for route structure compliance using grep patterns
- Violation handling: CI build fails if routes are created outside _authenticated+ that access protected resources
- Violation handling: Pull requests with non-compliant route names are flagged for revision during code review
- Violation handling: Quarterly audits identify and remediate any routes that don't follow conventions
- Exception process: Developer documents the reason for deviation in a comment at the top of the route file
- Exception process: Engineering lead reviews and approves the exception with a documented rationale
- Exception process: Exception is tracked in a central registry with a review date for potential future alignment