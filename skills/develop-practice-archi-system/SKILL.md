---
name: develop-practice-archi-system
description: Analyze and coordinate changes across the Practice Architecture System repositories. Use when a request spans components, changes shared contracts, or requires deciding which repository owns a behavior. Do not use for implementation confined to one repository.
---

# Develop Practice Architecture System

Use the `practice-archi-system` repository as the source of truth for
cross-component topology, ownership, and change order.

## Establish System Context

Locate the `practice-archi-system` repository in the workspace.

- Read `docs/system-map.md` when the request depends on runtime topology, data
  flow, or source-of-truth decisions.
- Read `docs/component-catalog.md` when selecting the repository that owns a
  behavior, contract, or data.
- Read `docs/engineering-standards.md` when implementing or reviewing code
  across components, or when deciding shared readability, naming, `impl.go`,
  and backward-compatibility rules.
- Read `docs/testing-standards.md` when adding or reviewing tests, mocks,
  database/ORM verification, external-client failure scenarios, controlled
  integration tests, or race-condition coverage.
- Read `docs/cross-service-changes.md` when the request changes a shared
  contract, shared library, producer, and consumer together.

Read only the documents needed for the current request. For implementation
inside a repository, also follow that repository's applicable `AGENTS.md` and
service-specific skill when available.

## Route by Ownership

- Route public HTTP boundaries and orchestration to `rest-orc`.
- Route authentication, identity, profile, role, JWT, and user persistence to
  `grpc-auth`.
- Route protobuf, shared error keys/codes/messages, mapping contracts, locale,
  and generated contract changes to `api-contracts`.
- Route reusable query, transaction, dialect, and database behavior to `orm`.
- Route reusable error loading, typed error construction, localization mechanics,
  fallback, and output formatting to `error-package`.

Keep the error boundary explicit: `api-contracts` defines which shared errors
exist and how boundaries can map them; `error-package` provides protocol-agnostic
runtime mechanics for presenting them. Do not put business messages or
HTTP/gRPC mapping inside `error-package`.

The current `api-contracts/pkg/dictionary` implementation still overlaps with
some runtime registry and loading mechanics. Preserve compatibility and do not
migrate that overlap unless the user explicitly includes the migration in scope.

Do not place feature-specific business rules in contract or shared library
repositories. Do not assume that access to sibling repositories authorizes
changes to all of them.

## Coordinate Cross-Repository Work

1. Identify the owner, producers, and consumers.
2. Inspect the relevant public contracts and current repository state.
3. Preserve compatibility unless a breaking change is explicitly requested.
4. Change only repositories included in the requested scope.
5. Verify each changed repository with its own test workflow.
6. Verify integration through the local workspace when appropriate.
7. Report publishing order, remaining consumer updates, and open decisions.

When the current documentation does not establish ownership or business
behavior, ask the user instead of silently selecting a new system contract.
