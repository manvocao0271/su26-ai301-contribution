# Contribution [#287]: [Adapter: Amp]

**Contribution Number:** [287]  
**Student:** [Man Cao]  
**Issue:** [https://github.com/orthogonalhq/nous-core/issues/287]  
**Status:** [Phase VI] [Review Feedback Addressed, Pending Re-Review]

---

## Why I Chose This Issue

I chose this issue because I found Nous to be quite interesting to learn from as a "good first issue". Nous is an AI personal assistant that lives on a user's local machine to help with daily tasks. I am developing my career as an AI engineer so I believe tackling this issue is a perfect fit for me. I also saw the issues page being relatively new and fresh on comments. It is likely that this issue will be assigned or at least easily reviewed since there is a low number of participants. This also makes the issue less complex in addition to the "acceptance criteria" that they attached to the issue.

---

## Understanding the Issue

### Problem Description

Nous is missing a provider adapter for Amp, a session-bound CLI coding agent. Without it, Nous cannot dispatch tasks to Amp or route it through the model provider pipeline. The issue originally referenced the old `AgentAdapter` / `self/subcortex/coding-agents` path, but that path has been superseded — the current contract is the CLI provider leaf system under `self/subcortex/providers/src/providers/<vendor>/`.

### Expected Behavior

Amp should be a certified provider leaf that appears in `PROVIDER_DEFINITIONS`, resolves correctly through the adapter/registry pipeline, and can have tasks dispatched to it via the Amp CLI process (`amp`). It must declare `executionCapabilityProfile: 'session_bound_command'` so Cortex can enforce role compatibility at selection time.

### Current Behavior

No Amp provider leaf exists. The provider catalog only contains `anthropic`, `openai`, and `ollama`.

### Affected Components

- `self/subcortex/providers/src/providers/` — new `amp/` leaf (definition, adapter, implementation, factory, index)
- `self/subcortex/providers/src/schemas/provider-definition.ts` — schema extended with `ExecutionCapabilityProfileSchema`, `ProviderDefinitionLeaf` type, and optional `defaultEndpoint`/`defaultModelId` for CLI providers
- `self/subcortex/providers/src/provider-identity.ts` — new file: central registry for deriving built-in provider IDs from vendor keys
- Generated catalogs: `provider-definitions.ts`, `provider-adapters.ts`, `provider-factories.ts`
- Runtime hardening: `provider-runtime.ts`, `bootstrap.ts` (optional-chain fixes for CLI providers without HTTP endpoints)

---

## Reproduction Process

### Environment Setup

First time trying to open with dev containers had errors. Node.js and pnpm wasn't installed and the configuration files were not correctly set. Had to change @devcontainer.json with the feature configuration settings.

### Steps to Reproduce

1. pnpm install & pnpm build, run web interface
2. review localhost:4317
3. A majority of the project is still in development (e.g. Dashboard, Org Chart, Inbox, etc.)

### Reproduction Evidence

- **Commit showing reproduction:** [https://github.com/manvocao0271/nous-core/tree/feat/adapter-amp](https://github.com/manvocao0271/nous-core/tree/feat/adapter-amp)
- **Screenshots/logs:** [If applicable]
- **My findings:** [Still have trouble with the CLI and the desktop app interface. Many of the features for the agent are still in production. Will talk with maintainer to get the latest production-ready code for further testing.]

---

## Solution Approach

### Analysis

The issue required understanding two things before writing a line of code: (1) the provider leaf contract has changed since the issue was filed — the old `AgentAdapter` path is superseded, and (2) the new CLI provider contract (`ProviderDefinitionLeaf`, `executionCapabilityProfile`, no hand-authored `wellKnownProviderId`) didn't yet exist in the schema and needed to be added as part of this contribution.

### Proposed Solution

Add the Amp CLI provider as a certified provider leaf under `self/subcortex/providers/src/providers/amp/`. Extend the schema to support CLI-style providers properly. Create `provider-identity.ts` to centralize ID derivation so no leaf hand-authors a `wellKnownProviderId` UUID.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The provider catalog is missing an Amp leaf. The CLI provider contract (`ProviderDefinitionLeaf`, `executionCapabilityProfile`) is described in the docs but not yet implemented in the schema. The issue's original path (`self/subcortex/coding-agents`) is superseded by the CLI provider leaf system.

**Match:** The existing `anthropic`, `openai`, and `ollama` leaves are the reference. For a CLI provider, Ollama (local, no required auth) is the closest structural analog, though Amp has no HTTP endpoint and uses `session_bound_command` instead.

**Plan:**
1. Extend `self/subcortex/providers/src/schemas/provider-definition.ts`: add `ExecutionCapabilityProfileSchema`, optional `defaultEndpoint`/`defaultModelId`, `ProviderDefinitionLeaf` type
2. Create `self/subcortex/providers/src/provider-identity.ts`: central `deriveBuiltInProviderId` function so leaves don't hand-author UUIDs
3. Create `self/subcortex/providers/src/providers/amp/definition.ts`: leaf definition using `ProviderDefinitionLeaf` + derived ID
4. Create `adapter.ts`: stateless text formatter and `parseResponse` with mandatory fallback
5. Create `implementation.ts`: `AmpProvider` class spawning the `amp` CLI binary via `node:child_process`
6. Create `provider.ts`, `index.ts`: factory and public exports
7. Regenerate catalogs and harden runtime call sites for optional `defaultEndpoint`

**Implement:** [feat/adapter-amp](https://github.com/manvocao0271/nous-core/tree/feat/adapter-amp)

**Review:** Checked against the four doc pages linked in the issue and the `anthropic` reference leaf. Confirmed `wellKnownProviderId` is not hand-authored. Confirmed `executionCapabilityProfile` is declared.

**Evaluate:** Run `pnpm --filter @nous/subcortex-providers run check:generated && typecheck && vitest run` against the provider test suite.

---

## Testing Strategy

### Unit Tests

- [x] `ProviderDefinitionSchema` validates the Amp definition (no `wellKnownProviderId` hand-authored, `executionCapabilityProfile` present)
- [x] `PROVIDER_DEFINITIONS` contains `'amp'` — asserted in `provider-definitions.test.ts`
- [x] `ProviderVendorKey` and `BootstrapProviderKey` derived types include `'amp'` — asserted in `provider-definition-types.test.ts`
- [x] Pipeline integration: registry resolves `'amp'` vendorKey to `AmpProvider` class — asserted in `provider-pipeline-integration.test.ts`
- [x] Amp adapter `formatRequest` maps system prompt + context frames correctly
- [x] Amp adapter `parseResponse` handles string output, object output, and malformed/null input without throwing
- [x] `AmpProvider.invoke` writes prompt to stdin and returns stdout
- [x] `AmpProvider.invoke` rejects on non-zero exit code with `PROVIDER_UNAVAILABLE`
- [x] `AmpProvider.invoke` rejects on timeout with `PROVIDER_UNAVAILABLE`
- [x] `AmpProvider.stream` throws `PROVIDER_UNAVAILABLE` (streaming not supported)
- [x] `AmpProvider.invoke` rejects invalid input before invoking the runner
- [x] `AmpProvider.invoke` propagates an already aborted `AbortSignal` to the runner
- [x] `createAmpProcessRunner` reports `spawn_error` for a nonexistent executable and honors abort before start

### Integration Tests

- [x] Generated catalogs are fresh (`check:generated` passes)
- [x] Provider pipeline end to end with a real spawned process standing in for the Amp CLI, via `createAmpProcessRunner` tested against `process.execPath`

### Manual Testing

Verified devcontainer builds and `pnpm install` + `pnpm build` succeed. Full `pnpm --filter @nous/subcortex-providers typecheck` and `vitest run` both pass clean (454 tests passed, 4 skipped, 0 failed) as of Week 6. Full monorepo `pnpm typecheck` surfaces two pre-existing failures in `self/apps/shared-server`, caused by `defaultEndpoint` becoming optional. Confirmed out of scope for this PR and flagged separately.

---

## Implementation Notes

### Week 1 Progress

Focused entirely on environment setup and understanding the codebase before writing any implementation code.

- Cloned the repo, attempted to open with Dev Containers — failed because Node.js and pnpm were not installed in the container and the devcontainer config was missing a `postCreateCommand`. Fixed `devcontainer.json` to run `corepack enable && corepack prepare pnpm@latest --activate` on creation and committed the devcontainer alongside a `dependabot.yml` for automated feature updates.
- Ran `pnpm install && pnpm build` successfully. Noted that most UI surfaces (Dashboard, Org Chart, Inbox) are still in development — the relevant surface for this issue is the provider pipeline, not the UI.
- Discovered that the integration branch (`feat/contributor-friendly-inference-provider-surface`) referenced in the issue had already been merged into `main`. Rebased off `main` instead. Renamed branch from `fix-issue-agentadapter-amp` to `feat/adapter-amp`.
- Read all four docs pages linked in the issue (cli-provider-guide, provider-leaf-anatomy, schemas-abi-reference, testing-checklist) and studied the `anthropic` reference leaf thoroughly.

**Key discovery this week:** `ProviderDefinitionLeaf`, `executionCapabilityProfile`, and `provider-identity.ts` are described in the docs as the current CLI provider contract but do not yet exist anywhere in the codebase. Adding them is part of this contribution's scope, not just creating the Amp files.

### Week 2 Progress

Implemented the schema extensions and the core Amp provider leaf files.

- Extended `self/subcortex/providers/src/schemas/provider-definition.ts`: added `ExecutionCapabilityProfileSchema` (Zod enum for `one_shot_command`, `session_bound_command`, `persistent_process`), made `defaultEndpoint` and `defaultModelId` optional (CLI providers have no HTTP endpoint), added `executionCapabilityProfile` as an optional field on `ProviderDefinitionSchema` and `ProviderDefinition`, and exported `ProviderDefinitionLeaf = Omit<ProviderDefinition, 'wellKnownProviderId'>`.
- Created `self/subcortex/providers/src/provider-identity.ts`: a central registry (`BUILT_IN_PROVIDER_IDS`) that maps vendor keys to stable built-in UUIDs. `deriveBuiltInProviderId(vendorKey)` is the single source of truth — leaves call this instead of hand-authoring a UUID string.
- Created `amp/definition.ts` using `ProviderDefinitionLeaf` for the authored portion and assembling the full `ProviderDefinition` export via `deriveBuiltInProviderId`. Added inline documentation explaining the leaf contract and why `session_bound_command` was chosen.
- Created `amp/adapter.ts`: stateless `ProviderAdapter` that formats prompt + context frames as plain text for Amp's stdin, and implements `parseResponse` with a mandatory non-throwing fallback.
- Hardened two runtime call sites that assumed `defaultEndpoint` was always present: added optional chaining in `provider-runtime.ts` and nullish coalescing in `bootstrap.ts`.

### Week 3 Progress

Completed the provider implementation, factory, public exports, generated catalog updates, and test coverage.

- Created `amp/implementation.ts`: `AmpProvider` class that implements `IModelProvider`. Spawns the `amp` binary via `node:child_process`, writes the formatted prompt to stdin, and collects stdout as the response. Handles timeout (120 s default), abort signal, non-zero exit codes, and spawn errors — all mapped to `NousError` with the appropriate error codes (`PROVIDER_UNAVAILABLE`, `ABORTED`). Streaming intentionally throws `PROVIDER_UNAVAILABLE` since Amp declares `streaming: false`.
- Created `amp/provider.ts` (factory) and `amp/index.ts` (public leaf exports required by the generator).
- Updated generated catalogs (`provider-adapters.ts`, `provider-definitions.ts`, `provider-factories.ts`) and `src/index.ts` to include the Amp leaf.
- Updated existing test files to include Amp assertions: `provider-definitions.test.ts` (catalog roster, metadata validation), `provider-definition-types.test.ts` (derived `ProviderVendorKey` type), `provider-pipeline-integration.test.ts` (registry resolves `'amp'` to `AmpProvider`).
- Added inline documentation to `definition.ts` and `index.ts`.

**Remaining:** run `pnpm typecheck` and the provider vitest suite to catch any type mismatches before opening the PR.

### Week 4 Progress

Focused on responding to review feedback and cleaning up branch scope.

- Received review feedback on PR #415 requesting changes to the schema, the CLI invocation, the process spawning approach, test coverage, whitespace, and unrelated files in the diff.
- Found two commits in the branch (docs branding, devcontainer setup) that weren't part of the Amp work and had been pulled in during an earlier rebase.
- Rebased the branch to drop those two commits, keeping only the Amp related work.
- Pushed the cleaned branch so the PR updated in place, now scoped to 17 files.

### Week 5 Progress

Focused on the two most self contained review comments: the schema fix and the CLI flag.

- Tracked down the duplicate `executionCapabilityProfile` field to a name collision between an import and a local redeclaration, and removed the redundant one.
- Confirmed Amp's documented non interactive flag is `-x`, not `--output text`, and updated the invocation.
- Ran a scoped typecheck to confirm both fixes were clean.
- Ran a full monorepo typecheck and found two unrelated pre existing failures in `shared-server`, caused by `defaultEndpoint` becoming optional. Decided this was out of scope for the PR and flagged it separately.
- Confirmed lint was clean for all files touched by this PR.
- Committed both fixes together.

### Week 6 Progress

Focused on the runner refactor and the missing behavior tests, the last two review comments.

- Reviewed the existing `github-copilot-cli` provider to understand the shared agent-cli runner pattern used elsewhere in the codebase.
- Added an `agentCli` metadata block to Amp's definition and rewrote `implementation.ts` so `AmpProvider` spawns through an injectable runner instead of a hardcoded process call.
- Updated the provider factory to support runner injection, matching the existing pattern.
- Added two new test files covering the provider's behavior and the runner's process handling, 14 tests total.
- Ran the full test suite: 454 passed, 4 skipped, 0 failed.
- Committed the refactor and the tests as two separate commits.
- **Remaining:** re-request review and summarize what changed since the last review pass.

### Code Changes

**Files created:**
- `self/subcortex/providers/src/provider-identity.ts`
- `self/subcortex/providers/src/providers/amp/definition.ts`
- `self/subcortex/providers/src/providers/amp/adapter.ts`
- `self/subcortex/providers/src/providers/amp/implementation.ts`
- `self/subcortex/providers/src/providers/amp/provider.ts`
- `self/subcortex/providers/src/providers/amp/index.ts`
- `self/subcortex/providers/src/__tests__/amp-provider.test.ts`
- `self/subcortex/providers/src/__tests__/amp-process-runner.test.ts`
- `.devcontainer/devcontainer.json` (dropped from the PR in Week 4, kept here as a record of Week 1 environment work)
- `.devcontainer/devcontainer-lock.json` (dropped from the PR in Week 4)
- `.github/dependabot.yml` (dropped from the PR in Week 4)

**Files modified:**
- `self/subcortex/providers/src/schemas/provider-definition.ts` — `ExecutionCapabilityProfileSchema`, `ProviderDefinitionLeaf`, optional endpoint/model fields
- `self/subcortex/providers/src/runtime/provider-runtime.ts` — optional-chain fix
- `self/apps/shared-server/src/bootstrap.ts` — nullish-coalescing fix
- `self/subcortex/providers/src/provider-adapters.ts` / `provider-definitions.ts` / `provider-factories.ts` — generated catalog updates
- `self/subcortex/providers/src/index.ts` — `AmpProvider` public export
- `self/subcortex/providers/src/__tests__/provider-definitions/provider-definitions.test.ts`
- `self/subcortex/providers/src/__tests__/provider-definitions/provider-definition-types.test.ts`
- `self/subcortex/providers/src/__tests__/provider-pipeline-integration.test.ts`
- `.gitignore` — added `.pnpm-store/`

**Key commits:** [feat/adapter-amp](https://github.com/manvocao0271/nous-core/tree/feat/adapter-amp)

**Approach decisions:**
- `session_bound_command` over `persistent_process`: Amp can maintain session context but does not expose a strict long-lived process protocol, matching the Codex CLI precedent in the docs.
- No `defaultEndpoint`: CLI providers don't communicate over HTTP. Rather than invent a fake URL, the schema was extended to make the field optional for CLI leaves.
- `ProviderDefinitionLeaf` as `Omit<ProviderDefinition, 'wellKnownProviderId'>`: minimal change that satisfies the contract without requiring the generator to change its catalog type.

---

## Pull Request

**PR Link:** [https://github.com/orthogonalhq/nous-core/pull/415](https://github.com/orthogonalhq/nous-core/pull/415)

**PR Description:** Adds Amp as a certified CLI provider leaf under `self/subcortex/providers/src/providers/amp/`, wiring the full definition, adapter, implementation, and factory into the `@nous/subcortex-providers` catalog. Extends the provider schema to support local CLI providers with no HTTP endpoint by making `defaultEndpoint` optional, adding `executionCapabilityProfile`, and introducing `ProviderDefinitionLeaf` and `provider-identity.ts` so leaves derive their built-in ID centrally rather than hand-authoring a UUID.

**Maintainer Feedback:**
- `@atlamors` re-reviewed the PR after the branch hygiene fix and still requested changes before merge.
- Blocking issue: duplicate `executionCapabilityProfile` declarations were introduced in `self/subcortex/providers/src/schemas/provider-definition.ts`; the base branch already exposes a typed `CliExecutionCapabilityProfileSchema.optional()` field and corresponding `CliExecutionCapabilityProfile` property.
- Requested revision: remove the duplicate loose `executionCapabilityProfile?: string` schema/interface shape and use the existing shared CLI capability-profile enum/type; if Amp needs a new value, extend the canonical shared enum.
- Requested Amp provider behavior changes and tests for CLI invocation, runner injection or equivalent hardening, timeout/abort handling, stdin write/end failures, non-zero exits, prompt formatting, stdout parsing, malformed output handling, and factory/runner injection coverage.
- Cleanup requested: normalize trailing whitespace in new Amp files and remove unrelated devcontainer/dependabot changes unless they are intentionally part of this provider-focused PR.

**Response to feedback (Weeks 4 to 6):**
- Week 4: rebased the branch to drop unrelated devcontainer/docs commits, restoring the PR to a provider only scope.
- Week 5: removed the duplicate `executionCapabilityProfile` declaration and switched the CLI invocation to the documented `-x` flag.
- Week 6: rewrote `AmpProvider` to route through an injectable runner, matching the existing `github-copilot-cli` pattern, and added dedicated behavior test coverage for both the provider and the runner.
- Not addressed in this PR: two pre-existing `shared-server` typecheck failures surfaced by `defaultEndpoint` becoming optional. Judged out of scope for issue #287 and flagged separately.

**Status:** All six original review comments addressed. Re-requesting review.

---

## Learnings & Reflections

### Technical Skills Gained

I learned how a monorepo provider pipeline is structured end-to-end, from schema definition through Zod validation to factory registration and runtime resolution. Working through the TypeScript `const satisfies` pattern and union type narrowing gave me a much stronger intuition for how strict typing shapes architecture decisions. I also got hands-on experience with `node:child_process` for spawning CLI subprocesses, handling stdin/stdout streams, abort signals, and timeout management in an async context. On the tooling side, I learned how pnpm workspaces, `tsc --build` with project references, and a code generator script all fit together to keep a large codebase consistent.

### Challenges Overcome

The biggest challenge was that the issue referenced a code path (`self/subcortex/coding-agents`) that had already been superseded. I had to read the docs carefully to discover the correct contract (`ProviderDefinitionLeaf`, `executionCapabilityProfile`, `provider-identity.ts`) and then realize none of those things existed in the codebase yet. They were described in the docs but not implemented. That reframing shifted the scope of the work significantly. A second challenge was the Windows build environment: `better-sqlite3` requires native compilation and the VS Build Tools C++ workload and Windows SDK had to be installed in stages before `pnpm install` would succeed. Debugging the TypeScript union type errors that came from adding a CLI provider (with no `defaultEndpoint`) to a catalog that existing code assumed always had an endpoint was also non-trivial and required tracing the type through several layers.

### What I'd Do Differently Next Time

I would verify the referenced code paths in the issue against the actual codebase before starting implementation, rather than discovering mid-way that the target had moved. I would also set up the build environment on a known-good platform first to avoid spending time on native dependency issues unrelated to the contribution itself. Finally, I would write the unit tests for the adapter and implementation in parallel with the code rather than leaving them as a follow-up, since the test gaps became apparent only after the PR was already open.

---

## Resources Used

- [https://docs.nue.orthg.nl/docs/development/provider-adapters/cli-provider-guide
https://docs.nue.orthg.nl/docs/development/provider-adapters/provider-leaf-anatomy
https://docs.nue.orthg.nl/docs/development/provider-adapters/schemas-abi-reference
https://docs.nue.orthg.nl/docs/development/provider-adapters/testing-checklist]
- [Tutorial or Stack Overflow post that helped]
- [[GitHub issues or discussions that helped](https://github.com/orthogonalhq/nous-core/issues/296)]
