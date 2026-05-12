# Establish Public API Export Contracts with Explicit Module Boundaries: Public Modules Not

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all public API modules and external-facing integration points. All exported functions, classes, and constants must adhere to the contract specifications defined herein.

## Context

- The codebase contains 29 files exhibiting a consistent pattern of explicit public API contract definitions with 89.48% confidence, indicating a deliberate architectural approach to API surface management
- Files such as index-server.js, constants.js, and various internal client modules demonstrate clear separation between public exports and internal implementation details
- The pattern spans compiler utilities (assert.js, warnings.js, builders.js), runtime client code (context.js, equality.js, ownership.js), and server-side rendering modules, suggesting a framework-wide architectural principle
- Development and production environments require different API surfaces, with dev-specific modules providing enhanced validation, tracing, and debugging capabilities while maintaining consistent contract structures
- The pattern signature (dc1b57bd689dc0475c8072de90be279a) represents a stable architectural decision that has been consistently applied across multiple subsystems including compiler, runtime, and server components

## Problem Statement

Without explicit public API contracts and clear module boundaries, framework consumers face unpredictable breaking changes, accidental coupling to internal implementation details, and difficulty understanding the stable API surface. This leads to fragile integrations, increased maintenance burden, and reduced framework evolution velocity as internal refactoring becomes constrained by implicit external dependencies.

## Decision

1. MUST_NOT: Public API modules MUST NOT export internal utility functions, private state management primitives, or implementation-specific helpers unless explicitly documented as public

## Policy Block

- MUST_NOT Public API modules MUST NOT export internal utility functions, private state management primitives, or implementation-specific helpers unless explicitly documented as public

In scope:
- All modules under /packages/svelte/src/ that are exposed through package.json exports field
- Index files (index.js, index-server.js) serving as public entry points
- Constants, types, and utilities explicitly exported for external consumption
- Development-mode APIs that mirror production APIs with enhanced diagnostics
- Compiler APIs intended for build tool integration and plugin development

Out of scope:
- Internal implementation modules under /internal/* directories not listed in package exports
- Test utilities and fixtures used exclusively within the test suite
- Build scripts, configuration files, and development tooling
- Private helper functions and utilities not exported from public modules
- Experimental or unstable APIs marked with internal-only documentation

Exceptions:
- EXC-001: Framework plugin developers require access to compiler internals for advanced transformations
- EXC-002: Development tools (debuggers, inspectors) need access to internal runtime state for diagnostic purposes

## Rationale

- The detected pattern across 29 files with 89.48% confidence demonstrates a mature, consistently applied architectural principle that has proven effective in managing API surface complexity
- Explicit contract boundaries enable independent evolution of internal implementations without breaking external consumers, accelerating framework development velocity
- Clear separation between public and internal modules reduces accidental coupling and makes it easier for new contributors to understand which code is stability-critical
- Environment-specific entry points (client, server, dev) allow optimization and feature differentiation while maintaining consistent contract structures across runtime contexts

## Consequences

Positive:
- External consumers gain predictable upgrade paths with clear expectations about API stability and breaking change policies
- Internal refactoring becomes safer and faster as implementation details can change without affecting public contracts
- Documentation and TypeScript definitions can focus on the stable public surface, improving developer experience
- Framework bundle size can be optimized by tree-shaking internal utilities that are never exposed to consumers
- Development and production builds can diverge in implementation while maintaining contract compatibility

Negative:
- Additional maintenance overhead to ensure internal modules are never accidentally exposed through public entry points
- Potential duplication of functionality between internal and public APIs when internal utilities would be generally useful
- Increased complexity in build configuration to manage multiple entry points and conditional exports
- Risk of creating overly restrictive boundaries that force consumers to work around limitations or duplicate framework code

## Alternatives

- Export all internal modules as public API with stability annotations (rejected)
  Rejected because: Creates excessive API surface area, makes it difficult to refactor internals, and places unreasonable documentation and stability burden on all code
  When valid: Appropriate for small libraries with minimal internal complexity where all code is effectively public
- Use single monolithic entry point with all exports in one file (rejected)
  Rejected because: Prevents tree-shaking, forces loading of unused code, and makes it difficult to provide environment-specific optimizations (client vs server vs dev)
  When valid: Suitable for very small libraries with minimal code where bundle size is not a concern
- Implement runtime access control with private symbols or WeakMaps (deferred)
  Rejected because: Adds runtime overhead and complexity; static module boundaries are sufficient for most use cases
  When valid: Could be adopted for specific high-security scenarios where compile-time boundaries are insufficient

## Risks

- Consumers may attempt to import internal modules directly, creating hidden dependencies that break on framework updates
  Mitigation: Implement build-time checks to detect imports from internal paths; document public API clearly; use package.json exports field to restrict access
  Owner: Framework Core Team
- Overly restrictive API boundaries may force consumers to duplicate framework functionality or use unsafe workarounds
  Mitigation: Establish feedback mechanism for API enhancement requests; regularly review common workarounds in community code; maintain escape hatches for advanced use cases
  Owner: API Design Working Group
- Multiple entry points (client, server, dev) may diverge in behavior, creating subtle bugs when code is shared across environments
  Mitigation: Implement comprehensive integration tests covering all entry points; use shared test suites to verify contract compatibility; document environment-specific behaviors clearly
  Owner: QA and Testing Team

## Implementation Notes

- Use package.json 'exports' field to explicitly define public entry points and prevent direct access to internal modules
- Establish naming conventions: public entry points at package root (index.js, index-server.js), internal code under /internal/ or /compiler/utils/
- Implement automated tooling to detect and flag any public API modules that import from internal paths, ensuring unidirectional dependency flow
- Create API documentation generation pipeline that only processes public entry points, automatically excluding internal modules
- For development-specific APIs, use conditional exports or build-time substitution to swap implementations while maintaining identical function signatures
- Consider using TypeScript's 'paths' configuration or module resolution plugins to enforce import restrictions during development

## Continuation Context


Verify commands:
- grep -r "from.*internal" packages/svelte/src/index*.js || echo 'No internal imports in public entry points'
- node -e "const pkg = require('./package.json'); console.log(pkg.exports ? 'Exports field defined' : 'ERROR: No exports field')"
- find packages/svelte/src -name 'index*.js' -exec grep -L 'export' {} \; | wc -l | grep -q '^0$' && echo 'All entry points have exports'

Accept when:
- All public entry point files (index.js, index-server.js) contain only imports from internal modules and explicit export statements without internal path leakage
- Package.json exports field explicitly lists all public entry points and restricts access to internal directories
- Automated linting or build checks successfully detect and reject any attempts to import internal modules from external code
- Documentation clearly distinguishes public API surface from internal implementation with examples of correct import patterns

## Enforcement

- Verified by: Automated CI checks using package.json exports validation and import path analysis
- Verified by: Code review checklist requiring verification that new exports are intentionally public
- Verified by: Integration tests that import the package as an external consumer would, ensuring internal paths are inaccessible
- Verified by: TypeScript compilation checks with strict module resolution to catch internal imports
- Violation handling: CI build fails if public entry points contain direct exports of internal implementation details
- Violation handling: Pull requests adding new exports to public APIs require explicit approval from API design reviewers
- Violation handling: Runtime warnings in development mode when deprecated or internal APIs are accessed
- Violation handling: Automated issue creation when community packages are detected importing from internal paths
- Exception process: Submit API enhancement proposal documenting the use case and why existing public APIs are insufficient
- Exception process: API design working group reviews proposal in bi-weekly meeting with 3-day async feedback window
- Exception process: Approved exceptions are documented in PLUGIN_API.md or ADVANCED_USAGE.md with stability warnings
- Exception process: Temporary exceptions for experimental features require sunset date and migration plan