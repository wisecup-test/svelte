# Adopt Modular Public API Export Pattern with Explicit Re-exports: Exported Functions Classes

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 18 files exhibiting a consistent pattern for exposing public APIs through explicit re-export modules
- Files are organized into internal implementation directories (compiler/utils, internal/client/dev) with separate public-facing entry points
- The pattern appears in both client-side runtime code and compiler utilities, indicating a framework-wide architectural decision
- Development and production builds require different API surfaces, with dev-specific utilities separated from core functionality
- The pattern signature (eff7ce064ab2e3f219859e6512c7e811) shows 90% confidence across multiple subsystems including compiler, client runtime, and server entry points

## Problem Statement

Framework codebases need to maintain clear boundaries between internal implementation details and public APIs while supporting multiple build targets (development, production, server, client). Without a consistent export pattern, consumers may inadvertently depend on internal APIs, breaking encapsulation and making refactoring difficult. The challenge is to establish a standardized approach for exposing public APIs that maintains backward compatibility, enables tree-shaking, and clearly communicates API stability guarantees.

## Decision

1. MUST: All exported functions and classes MUST be documented with JSDoc comments indicating their public API status

## Policy Block

- MUST All exported functions and classes MUST be documented with JSDoc comments indicating their public API status

In scope:
- All framework entry points (index.js, index-server.js, etc.)
- Public API modules intended for external consumption
- Compiler utilities exposed to plugin authors
- Runtime functions documented in public API documentation
- Development tools and helpers exported for debugging

Out of scope:
- Internal implementation files not re-exported through entry points
- Test utilities and fixtures
- Build scripts and tooling configuration
- Private helper functions used only within single modules
- Experimental APIs not yet stabilized

Exceptions:
- EXC-001: Framework plugins require access to internal compiler APIs for advanced transformations
- EXC-002: Performance-critical paths require direct access to internal runtime functions

## Rationale

- The pattern is detected across 18 files with 90% confidence, indicating a deliberate architectural decision rather than coincidental similarity
- Separation of internal implementation from public APIs enables safe refactoring without breaking external consumers
- Explicit re-export pattern provides a single source of truth for what constitutes the public API surface
- Build-target-specific entry points (server vs client) allow optimal bundle sizes through tree-shaking while maintaining a unified codebase
- The pattern aligns with modern JavaScript module best practices and supports both ESM and CommonJS consumers

## Consequences

Positive:
- Clear API boundaries reduce accidental coupling to internal implementation details
- Refactoring internal code becomes safer as public API surface is explicitly defined
- Tree-shaking and dead code elimination work more effectively with explicit exports
- Documentation generation can focus on public entry points, improving API documentation quality
- Multiple build targets (dev/prod, client/server) can be supported without code duplication

Negative:
- Additional maintenance overhead to keep entry point re-exports synchronized with internal modules
- Developers must remember to export new public APIs through entry points, creating potential for oversight
- Slightly increased build complexity due to multiple entry points for different targets
- Risk of confusion between internal and public import paths during development

## Alternatives

- Single monolithic export file with all APIs exposed (rejected)
  Rejected because: Would eliminate tree-shaking benefits and expose all internal APIs, making refactoring dangerous and increasing bundle sizes
  When valid: Only appropriate for very small libraries with minimal internal complexity
- Direct imports from internal modules without entry points (rejected)
  Rejected because: Creates tight coupling to internal file structure, making refactoring break external consumers and eliminating API boundaries
  When valid: Acceptable for internal-only modules within the same package that are never exposed externally
- Use package.json exports field to define public API surface (deferred)
  Rejected because: Complementary approach that could be adopted alongside explicit re-exports for additional enforcement
  When valid: Should be considered as an enhancement to provide tooling-level enforcement of API boundaries

## Risks

- Developers may forget to export new public APIs through entry points, causing confusion when features are implemented but not accessible
  Mitigation: Implement automated tests that verify all intended public APIs are exported through entry points; add linting rules to detect unexported public functions
  Owner: Engineering team
- Entry point files may become large and difficult to maintain as the API surface grows
  Mitigation: Organize entry points by feature area or subsystem; use intermediate aggregation modules if a single entry point exceeds 200 lines
  Owner: Architecture team
- External consumers may bypass entry points and import directly from internal modules, defeating the purpose of API boundaries
  Mitigation: Use package.json exports field to restrict access to internal paths; document import patterns clearly; consider build-time warnings for internal imports
  Owner: DevEx team

## Implementation Notes

- Create entry point files (index.js) at package roots that re-export all public APIs using named exports
- Organize internal implementation in subdirectories (internal/, compiler/, etc.) that are not directly accessible to consumers
- Use build-target-specific entry points (index-server.js, index-client.js) for environment-specific APIs
- Add JSDoc @public tags to all functions/classes intended for external use and @internal tags for private APIs
- Configure package.json exports field to explicitly allow only entry point imports and block direct internal module access
- Implement CI checks that verify entry point exports match documented public APIs

## Continuation Context


Verify commands:
- grep -r "export.*from.*internal" packages/*/src/index*.js | wc -l
- find packages/*/src -name 'index*.js' -exec grep -L 'export.*from' {} \;
- npm run build && node -e "const api = require('./dist'); console.log(Object.keys(api).length > 0)"

Accept when:
- All public entry point files contain only re-export statements without implementation logic
- Internal implementation files are organized in separate directories from entry points
- Build succeeds and produces separate bundles for different targets (client, server, dev) when applicable
- Public API documentation can be generated solely from entry point files

## Enforcement

- Verified by: Automated CI checks verify entry point structure and export patterns
- Verified by: Code review checklist includes verification of public API exports
- Verified by: Linting rules detect direct imports from internal modules in external-facing code
- Verified by: API documentation generation process validates entry point completeness
- Violation handling: CI build fails if entry points contain implementation logic beyond re-exports
- Violation handling: Pull requests are blocked if new public APIs are not exported through entry points
- Violation handling: Linter warnings are elevated to errors for direct internal module imports in public examples
- Violation handling: Quarterly API audits identify and remediate violations
- Exception process: Submit exception request to architecture review board with justification
- Exception process: Document exception in ADR exceptions section with approval date and reviewer
- Exception process: Add inline code comments explaining why exception is necessary
- Exception process: Schedule review of exception after 6 months to determine if it can be removed