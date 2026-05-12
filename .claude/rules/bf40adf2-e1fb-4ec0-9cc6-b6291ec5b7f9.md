<rule_activation id="bf40adf2-e1fb-4ec0-9cc6-b6291ec5b7f9" title="Standardize Public API Export Patterns for External Integration: Modules Expose Additional" applies_to="**/*">
These rules are ALWAYS ACTIVE for all modules in packages/svelte/src/internal/client/*, packages/svelte/src/compiler/*, and core framework APIs that expose functionality to external consumers.
</rule_activation>

### Rules

- **R-API-001** MAY: Modules MAY expose additional experimental APIs with clear deprecation or stability warnings.

### Verify

```bash
# Count explicit export statements in client and compiler modules
grep -r 'export.*from' packages/svelte/src/internal/client packages/svelte/src/compiler | grep -v '.test.js' | wc -l

# Find modules using explicit export syntax
find packages/svelte/src -name '*.js' -path '*/internal/*' -exec grep -l 'export {' {} \; | wc -l

# Lint for export boundary violations
npm run lint -- --rule 'no-restricted-syntax: [error, ExportAllDeclaration]'
```

**Accept when:**
- All public API modules in scope have explicit export statements that match the detected pattern
- No internal implementation details are exported without @public or @experimental documentation tags
- Linting passes with no violations of export boundary rules
- API documentation clearly distinguishes public vs internal APIs for all modules in scope

<enforcement>
Claude Code MUST verify all export patterns against the linting rules and documentation requirements before approving changes to public APIs. Verification is mandatory and MUST NOT be skipped or deferred.
</enforcement>