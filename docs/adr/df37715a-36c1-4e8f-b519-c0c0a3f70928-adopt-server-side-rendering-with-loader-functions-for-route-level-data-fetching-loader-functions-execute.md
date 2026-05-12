# Adopt Server-Side Rendering with Loader Functions for Route-Level Data Fetching: Loader Functions Execute

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all route components in the Remix application framework. All route modules MUST implement server-side data loading patterns as specified herein.

## Context

- The application uses Remix framework which provides server-side rendering capabilities with loader functions for data fetching at the route level
- Pattern detected across 4 route files including public routes (_index.tsx), internal PDF generation routes (__htmltopdf), and authentication flows (SSO confirmation)
- The facet 'paradigm.concurrency_model' indicates this pattern relates to how data loading and rendering are coordinated between server and client execution contexts
- Server-side data loading enables better performance, SEO optimization, and security by keeping sensitive data fetching logic on the server
- The pattern shows consistent adoption across different route types (public, internal, authenticated) suggesting a standardized architectural approach

## Problem Statement

Applications need a consistent, performant, and secure mechanism for loading data required by route components. Client-side data fetching introduces waterfall loading patterns, exposes API endpoints unnecessarily, and degrades initial page load performance. Without a standardized approach, routes may implement inconsistent data loading strategies leading to maintenance complexity and unpredictable user experiences.

## Decision

1. MUST: Loader functions MUST execute on the server and return serializable data to the route component

## Policy Block

- MUST Loader functions MUST execute on the server and return serializable data to the route component

In scope:
- All Remix route modules (files in app/routes directory)
- Public routes requiring initial data (_index.tsx)
- Authenticated routes with user-specific data
- Internal routes for server-side rendering (PDF generation, reports)
- SSO and authentication flow routes

Out of scope:
- Client-side only components that don't correspond to routes
- Shared utility components without route-level data requirements
- Real-time data subscriptions (WebSocket, SSE) for live updates
- Progressive enhancement features that load after initial render

Exceptions:
- EXC-001: Route requires real-time data updates that cannot be efficiently handled through revalidation
- EXC-002: Third-party widget or embed requires client-side initialization with dynamic data

## Rationale

- Pattern detected with 90.92% confidence across 4 diverse route types (index, internal PDF, SSO confirmation, certificate) indicates strong architectural consistency
- Server-side rendering with loader functions eliminates client-side data fetching waterfalls, improving Time to First Byte (TTFB) and Largest Contentful Paint (LCP) metrics
- Keeping data fetching logic on the server enhances security by preventing exposure of API endpoints, authentication tokens, and business logic to the client
- Remix framework's loader pattern provides built-in support for progressive enhancement, error handling, and type safety, reducing boilerplate and improving developer experience

## Consequences

Positive:
- Improved initial page load performance through server-side rendering and elimination of client-side data fetching waterfalls
- Enhanced security posture by keeping sensitive data access logic, authentication, and authorization on the server
- Better SEO as content is rendered server-side and available to crawlers immediately
- Simplified error handling through Remix error boundaries and consistent server-side error responses
- Type-safe data flow from loader to component using TypeScript inference

Negative:
- Increased server load as all initial data fetching happens server-side rather than distributed to clients
- Potential for longer Time to First Byte (TTFB) if loader functions perform slow operations without optimization
- Learning curve for developers unfamiliar with server-side rendering patterns and Remix conventions
- Complexity in handling client-side state updates that need to sync with server-rendered data

## Alternatives

- Client-side data fetching with React Query or SWR in useEffect hooks (rejected)
  Rejected because: Creates waterfall loading patterns, exposes API endpoints unnecessarily, degrades initial page load performance, and complicates SEO
  When valid: Only valid for progressive enhancement features or real-time updates after initial render
- Static site generation (SSG) with build-time data fetching (rejected)
  Rejected because: Not suitable for dynamic, user-specific, or frequently changing data as seen in authentication flows and audit logs
  When valid: Valid for truly static content that changes infrequently and doesn't require user context
- Hybrid approach with getServerSideProps (Next.js pattern) (rejected)
  Rejected because: Remix loader pattern provides superior developer experience with better type inference, error handling, and progressive enhancement support
  When valid: Valid if migrating from Next.js and maintaining consistency during transition period

## Risks

- Slow loader functions can block page rendering and degrade user experience with long TTFB
  Mitigation: Implement loader performance monitoring, add timeouts, use caching strategies, and optimize database queries. Consider parallel data fetching where possible.
  Owner: Engineering team with performance monitoring by SRE
- Server resource exhaustion under high traffic if all data fetching happens server-side
  Mitigation: Implement rate limiting, caching layers (Redis, CDN), database connection pooling, and horizontal scaling. Monitor server metrics and set up auto-scaling.
  Owner: Infrastructure team with support from backend engineers
- Developers may bypass loader pattern and implement client-side fetching without proper review
  Mitigation: Establish code review guidelines, implement linting rules to detect client-side data fetching in route components, and provide clear documentation with examples.
  Owner: Engineering team leads and architecture review board

## Implementation Notes

- Use Remix's useLoaderData() hook in route components to access data returned from loader functions with full type safety
- Implement error handling in loaders using throw new Response() for HTTP errors or throw redirect() for authentication redirects
- Leverage Remix's built-in caching headers in loader responses to enable CDN and browser caching where appropriate
- For routes requiring authentication, check session/token validity in the loader and redirect to login if unauthorized
- Use defer() for streaming server-side rendering when some data can be loaded asynchronously after initial render
- Structure loader functions to be testable independently from components, enabling unit tests for data fetching logic

## Continuation Context


Verify commands:
- grep -r "export.*loader" apps/remix/app/routes/ --include="*.tsx" --include="*.ts" | wc -l
- grep -r "useEffect.*fetch\|useEffect.*axios" apps/remix/app/routes/ --include="*.tsx" | wc -l
- npm run type-check -- --noEmit

Accept when:
- All route files that require initial data have exported loader functions (grep count > 0 for routes with data needs)
- No route components use useEffect for initial data fetching (grep count = 0 for useEffect with fetch/axios in routes)
- TypeScript compilation passes without errors, confirming type safety between loaders and components

## Enforcement

- Verified by: Automated CI pipeline checks for loader function presence in route files
- Verified by: ESLint custom rule to detect client-side data fetching patterns in route components
- Verified by: Code review checklist requiring verification of loader implementation for new routes
- Verified by: TypeScript strict mode compilation ensuring type safety between loaders and components
- Violation handling: CI pipeline fails if route files fetch data client-side without documented exception
- Violation handling: Pull requests blocked until loader pattern is implemented or exception is approved
- Violation handling: Architecture review required for any exceptions with security and performance impact assessment
- Violation handling: Quarterly audit of existing routes to identify and remediate violations
- Exception process: Developer documents technical justification for exception in ADR exception log
- Exception process: Tech lead reviews exception request for validity against policy scope and alternatives
- Exception process: Security review required if exception involves sensitive data or authentication flows
- Exception process: Approved exceptions must be documented in code comments with reference to exception ID
- Exception process: Exceptions reviewed quarterly to determine if they can be migrated to standard pattern