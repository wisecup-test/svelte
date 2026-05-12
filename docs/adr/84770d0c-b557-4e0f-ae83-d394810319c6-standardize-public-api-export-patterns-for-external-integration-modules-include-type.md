# Standardize Public API Export Patterns for External Integration: Modules Include Type

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple internal modules (async blocks, CSS dev tools, compiler warnings, context management) that expose functionality to external consumers
- A consistent pattern has emerged across 4 files with 90% confidence for how public APIs are structured and exported
- External integrators need stable, well-defined interfaces to interact with framework internals without coupling to implementation details
- The pattern signature f8cf7f20e7f18319e094d3c7c19a989d represents a standardized approach to service boundary definitions in the boundaries.service_definitions facet

## Problem Statement

Without standardized public API export patterns, external consumers face inconsistent interfaces, breaking changes, and tight coupling to internal implementation details. This creates maintenance burden, reduces framework adoption, and makes versioning and backward compatibility difficult to manage.

## Decision

1. SHOULD: API modules SHOULD include type definitions or JSDoc annotations for all exported functions and objects

## Policy Block

- SHOULD API modules SHOULD include type definitions or JSDoc annotations for all exported functions and objects

In scope:
- All modules in packages/svelte/src/internal/client/* that expose functionality to external consumers
- Compiler modules in packages/svelte/src/compiler/* that provide public interfaces
- Core framework APIs including context management, DOM manipulation, and async handling
- Developer tooling APIs such as CSS debugging and warning systems

Out of scope:
- Pure internal utility functions with no external consumers
- Test-only modules and fixtures
- Build system and tooling configuration files
- Private implementation details not intended for external use

Exceptions:
- EXC-001: Framework maintainers need to expose experimental APIs for early adopter testing

## Rationale

- Pattern detected across 4 critical files (async.js, css.js, warnings.js, context.js) with 90% confidence indicates a proven, stable approach
- Consistent API boundaries reduce cognitive load for external integrators and improve framework maintainability
- The boundaries.service_definitions facet classification indicates this pattern specifically addresses service boundary concerns
- Standardization enables better tooling support, type checking, and automated compatibility verification

## Consequences

Positive:
- External consumers gain stable, predictable interfaces that reduce integration friction
- Framework maintainers can refactor internal implementations without breaking external contracts
- Improved documentation and discoverability of public APIs through consistent patterns
- Better versioning and backward compatibility management through clear boundary definitions

Negative:
- Additional overhead in maintaining explicit export boundaries and documentation
- Potential rigidity in API evolution if boundaries are too restrictive
- May require refactoring existing modules that don't follow the pattern
- Increased complexity in distinguishing between internal and public APIs during development

## Alternatives

- Export all functions and let consumers decide what to use (rejected)
  Rejected because: Creates tight coupling to implementation details, makes breaking changes unavoidable, and provides no clear contract for external consumers
  When valid: Never recommended for production frameworks with external consumers
- Use separate public API wrapper packages that re-export selected internals (rejected)
  Rejected because: Adds maintenance overhead of keeping wrapper packages in sync, creates indirection that complicates debugging, and duplicates documentation
  When valid: Could be valid for major version transitions or when supporting multiple API versions simultaneously
- Implement API versioning with explicit v1, v2 namespaces (deferred)
  Rejected because: Adds complexity that may not be needed initially, but could be valuable for future major version transitions
  When valid: Should be reconsidered when breaking changes accumulate or multiple API versions need simultaneous support

## Risks

- Existing external consumers may be using undocumented internal APIs that will break when boundaries are enforced
  Mitigation: Conduct API usage analysis across known integrations, provide migration guides, and use deprecation warnings before removing access
  Owner: Framework API team
- Overly restrictive boundaries may force legitimate use cases to work around the API or fork the framework
  Mitigation: Establish clear exception process, gather feedback from major integrators, and iterate on boundary definitions based on real-world usage
  Owner: Engineering team
- Pattern may not scale to all module types or future architectural changes
  Mitigation: Review pattern applicability quarterly, allow for pattern evolution through ADR updates, and maintain flexibility for special cases
  Owner: Architecture review board

## Implementation Notes

- Start by auditing the 4 detected files (async.js, css.js, warnings.js, context.js) to document the exact pattern structure
- Create a template or linting rule that enforces the pattern for new public API modules
- Gradually refactor existing modules to align with the pattern, prioritizing high-traffic APIs first
- Document the pattern in contributor guidelines with examples from the detected files

## Continuation Context


Verify commands:
- grep -r 'export.*from' packages/svelte/src/internal/client packages/svelte/src/compiler | grep -v '.test.js' | wc -l
- find packages/svelte/src -name '*.js' -path '*/internal/*' -exec grep -l 'export {' {} \; | wc -l
- npm run lint -- --rule 'no-restricted-syntax: [error, ExportAllDeclaration]'

Accept when:
- All public API modules in scope have explicit export statements that match the detected pattern
- No internal implementation details are exported without @public or @experimental documentation tags
- Linting passes with no violations of export boundary rules
- API documentation clearly distinguishes public vs internal APIs for all modules in scope

## Enforcement

- Verified by: Automated linting rules in CI pipeline checking export patterns
- Verified by: Code review checklist requiring explicit approval for new public API exports
- Verified by: Quarterly API boundary audits by architecture team
- Violation handling: CI build fails if linting detects non-compliant export patterns
- Violation handling: Pull requests with new public exports require architecture team review
- Violation handling: Existing violations are tracked in technical debt backlog with prioritized remediation plan
- Exception process: Submit exception request to architecture review board with justification and impact analysis
- Exception process: Core team lead must approve all exceptions with documented rationale
- Exception process: Exceptions are time-limited (max 2 release cycles) and require follow-up remediation plan