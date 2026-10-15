# AGENTS.md

TypeScript monorepo managed with pnpm.

## Working in the repository

- Read the relevant README, owning package's `package.json`, and any scoped `AGENTS.md` before editing.
- Follow existing patterns and keep changes focused on the task. Preserve unrelated work already in the working tree.
- Use package manifests and configuration as the source of truth for scripts, versions, workspace membership, and runtime settings.

## Code and dependencies

- Keep behavior in the package that owns it. Reuse existing code before introducing shared abstractions.
- Import other packages through their public APIs. Declare dependencies in the packages that use them, use `workspace:*` for internal dependencies, and avoid circular package dependencies.

## TypeScript style

- Preserve strict type checking. Prefer `unknown` and narrowing over `any` or broad assertions; fix errors rather than suppressing diagnostics or weakening configuration.
- Define each object-shaped union member as a separate, meaningfully named `type` or `interface`, then compose the union from those names. Do not inline anonymous object types in a union.

```ts
type Success = { kind: 'success'; value: string };
type Failure = { kind: 'failure'; error: Error };
type Result = Success | Failure;
```

## Validation commands

Oxlint and Oxfmt use shared root configs: [.oxlintrc.json](.oxlintrc.json) and [.oxfmtrc.json](.oxfmtrc.json). Use scoped `overrides` for package exceptions. Package-local Oxlint configs should extend the root config; nested Oxfmt configs replace it, so keep formatting exceptions in root overrides.

Run package scripts from the repository root. Replace `<package>` with the name in its `package.json`; use these commands where the corresponding scripts are defined:

| Check        | One package                                | Workspace               |
| ------------ | ------------------------------------------ | ----------------------- |
| Tests        | `pnpm --filter <package> run test`         | `pnpm -r run test`      |
| Lint         | `pnpm --filter <package> run lint`         | `pnpm run lint`         |
| Lint fixes   | `pnpm --filter <package> run lint:fix`     | `pnpm run lint:fix`     |
| Format check | `pnpm --filter <package> run format:check` | `pnpm run format:check` |
| Format       | `pnpm --filter <package> run format`       | `pnpm run format`       |
| Types        | `pnpm --filter <package> run typecheck`    | `pnpm -r run typecheck` |

- Start with the smallest relevant tests and checks. Inspect the configured runner before passing file paths or filtering flags.
- Check affected consumers when changing shared behavior or exported types. Broaden validation for repository-wide configuration changes or when the impact cannot be bounded.
- Recursive commands run defined scripts; they do not guarantee every package has a check. Report relevant missing, skipped, or failing checks explicitly.
- For documentation-only changes, review the diff and check whitespace. Code tests are unnecessary unless behavior or tooling also changes.
- Before finishing, review the diff for unintended changes and report which checks ran and their results.

## Test judgment

Tests have maintenance costs. Add the smallest set that protects meaningful application behavior.

- Before adding a test, identify the specific, plausible regression in our code it would catch. If you cannot name one, omit the test.
- Inspect existing coverage first. Extend an existing test when it already exercises the relevant behavior.
- Test observable requirements, decisions, transformations, and failure handling. Do not mirror the implementation or calculate expected results using the same logic as the implementation.
- Never mock the behavior being tested. Mock external boundaries only when necessary; keep our relevant production logic real.
- Do not test third-party libraries, framework internals, language semantics, trivial accessors, or straightforward delegation unless our application adds a meaningful contract.
- Each case must protect a distinct behavior or failure mode. Remove duplicates; parameterize similar cases only when each input matters.
- Do not introduce fixtures, local servers, certificates, dependencies, or test infrastructure merely to demonstrate a dependency's feature. Any such infrastructure must protect a named application contract.
- Do not add tests solely for coverage percentages or test counts. Zero new tests is acceptable when existing tests adequately protect the change or there is no meaningful new behavior to protect.
- Before finishing, review tests added in this task. Remove redundant, tautological, and implementation-coupled tests. Run relevant existing checks and briefly explain what any new tests protect.
