# Arlo Core Dependency Map for Charlie Composition

## 1. Executive Summary

- `arlo-core` is the Arlo Inc. platform/moat repository: capability registry, contract framework, identity/cognition doctrine, memory and intelligence surfaces, execution vault patterns, and deploy scaffolding.
- Charlie should inherit the agent-template doctrine from `workspace/core/TEMPLATE-SPINE.md`, but Charlie should not copy Arlo's own `SOUL.md` or `COGNITION.md` as final identity. Those are Arlo-specific examples and source-of-truth references.
- The most relevant Charlie inputs are the layer split in `CLAUDE.md`, the constitutional constraints in `docs/doctrine/CONSTRAINTS.md`, the capability state in `registry/CAPABILITY_REGISTRY.md`, and the contract/event/schema patterns under `workspace/lib/`.
- Authority does not belong in Charlie. Charlie can propose, remember, reason, coordinate, and ship artifacts; Arlo routes and persists through platform substrate; Lyhna authorizes execution.
- Stage 2 safety scaffolding is present: root `.gitignore`, `CLAUDE.md`, `README.md`, `docs/doctrine/CONSTRAINTS.md`, and scaffold-only deploy scripts under `deploy/`.
- The repo contains a large staged import of `/root/.openclaw/` material under `workspace/`, including tenant/product files and live-looking runtime references. Charlie should reference only the platform contracts and doctrine, not copy runtime state or tenant content.
- Memory/intelligence is partly live-by-docs and partly planned. CAP-MEM-006 person resolver has migrations, a writer-path proof, and a plugin skeleton, but resolver behavior is still described as skeletal in the checked source.
- L11/Dreaming and IRL surfaces are documented and partially scaffolded, but Charlie Phase 1 should treat them as compatibility targets unless a later doctrine proves live runtime availability.
- The requested `docs/CONSTRAINTS.md` path is not present; the actual constraints document is `docs/doctrine/CONSTRAINTS.md`.
- Readiness verdict: READY WITH WARNINGS. Charlie can run a composition doctrine against these sources, but only as a governed proposal/build artifact, not a deploy or live integration pass.

## 2. Directory Map

| Path | Purpose | Charlie Relationship | Notes |
|---|---|---|---|
| `README.md` | Repo scope, related repos, operating principles, and Track B/Track C split. | reference | Establishes arlo-core as platform/moat, not tenant identity or runtime state. |
| `CLAUDE.md` | Operating doctrine for agents working in arlo-core. | inherit | Hard source for Codex vs Claude Code responsibilities, no-VPS rule, no auto-promotion, JWT/RLS discipline, and layer split. |
| `.gitignore` | Secret, runtime-state, memory-state, media, and deploy-output exclusions. | inherit | Charlie should mirror the safety posture for generated composition repos. |
| `SYSTEM_MAP.md` | Staged map of live `/root/.openclaw/` surfaces, plugin chain, sidecars, Supabase surfaces, and known gaps. | reference | High-value map, but it is a snapshot and says live state may diverge from disk. Do not treat as proof without later verification. |
| `openclaw.json.example` | Redacted OpenClaw runtime config template. | avoid | Contains env keys, provider settings, runtime paths, and live-looking identifiers. Do not copy values into Charlie. |
| `.claude/` | Claude Code local settings. | ignore-for-Phase-1 | Not needed for Charlie composition. |
| `bin/` | Admin CLI scaffolds, notably credential vault admin. | compatible-with | Useful as execution-surface reference only. Charlie must not run or modify credential tooling. |
| `canvas/` | Visual or product artifacts. | ignore-for-Phase-1 | Not needed for substrate composition. |
| `cron/` | OpenClaw cron jobs and run logs. | reference | Shows memory promotion/dream job shape, but also contains runtime run state. Do not copy run logs. |
| `deploy/` | Scaffold-only `deploy-to-vps.sh` and `branch-from-vps.sh`. | avoid | Important boundary: Codex does not activate, replace placeholders, or contact VPS. Claude Code later owns this. |
| `docs/` | Repo docs organized into `closures/`, `doctrine/`, and `recon/`. | inherit | `docs/doctrine/CONSTRAINTS.md` is mandatory. Closure docs prove Stage 2 scaffold state. |
| `extensions/` | OpenClaw plugin sources for orientation, cognition, routing, retrieval, skill loading, observability, person resolver, etc. | compatible-with | Reference adapter patterns and contract usage. Do not blindly lift Arlo-specific paths or env access. |
| `lib/` | Root shared libraries currently focused on person resolver writer. | compatible-with | CAP-MEM-006 writer-path reference; Charlie should integrate via platform writer, not self-write. |
| `migrations/` | Supabase SQL for EXEC credentials/bindings and CAP-MEM-006 person resolver. | reference | Good schema/RLS reference. Phase 1 must not execute migrations. |
| `recon/` | CAP-MEM-006 recon and writer-path proof reports. | reference | Useful to distinguish proven writer path from unimplemented resolver behavior. |
| `registry/` | Capability registry. | inherit | Source of capability state and doctrine lines; must remain platform-owned. |
| `subagents/` | Runtime/subagent run state. | ignore-for-Phase-1 | Not a Charlie build input. |
| `telegram/` | Telegram bridge state/cache artifacts. | ignore-for-Phase-1 | Runtime channel state; avoid. |
| `tenants/` | Per-tenant sanitization and sandbox config. | compatible-with | Shows tenant pathing/allowlist pattern. Charlie must not absorb Arlo tenant config as identity. |
| `workspace/` | Large staged OpenClaw workspace: core doctrine, skills, docs, memory, product artifacts, tenant content, libraries. | reference | Use `workspace/core/`, `workspace/lib/`, and `workspace/docs/`; avoid tenant media/content and runtime memory artifacts. |

## 3. Identity / Cognition Sources

| Candidate | Path | Why It Matters | Charlie Action | Ambiguity |
|---|---|---|---|---|
| Agent template doctrine | `workspace/core/TEMPLATE-SPINE.md` | Best canonical source for how every Agency agent is parameterized: `SOUL.md`, `COGNITION.md`, governor gate, five-layer separation, and forming/executing boundary. | Copy/reference as the governing template for Charlie draft artifacts. | Low. This is the strongest Charlie identity/cognition input. |
| Canonical cognition example | `workspace/core/COGNITION.md` | Arlo's current cognitive doctrine: frames, postures, source-of-truth hierarchy, memory stance, Lyhna boundary, and no self-edit of cognition/identity. | Reference only. Use structure, not content, when drafting Charlie-specific cognition. | Medium. It is canonical for Arlo, not Charlie. |
| Canonical SOUL example | `workspace/core/SOUL.md` | Arlo's current identity file and a worked example of first-person identity, refusal boundaries, voice, and continuity anchor. | Reference only. Do not copy as Charlie identity. | Medium. It is an example and source of invariants, not Charlie's voice. |
| Runtime identity injection source | `extensions/prime-orientation/index.ts` | Loads `workspace/core/SOUL.md` Core Identity and injects it before model calls. Shows how identity enters the prompt path. | Reference the mechanism; adapt path and source to Charlie later. | Medium. Hardcoded Arlo paths must be parameterized. |
| Agent startup convention | `workspace/AGENTS.md` | Legacy startup sequence: read SOUL, USER, cognition, ACTIVE, memory. | Reference with caution. | High. It allows agent memory edits in ways that conflict with the newer governor-promotion invariant. |
| Visual identity/history | `workspace/IDENTITY.md`, `workspace/ARLO.md`, `workspace/SOUL.md` | Older Arlo identity/brand/persona materials. | Avoid for Charlie except as historical context. | High. These contain Arlo/Frostbite-specific identity and older self-update patterns. |
| Agent-card convention | `workspace/core/TEMPLATE-SPINE.md`, prior Track B `agency-v0` docs in this repo if needed | `arlo-core` does not expose a single file named "agent card"; the template spine is the closest canonical convention. | Defer final agent-card mapping until Charlie doctrine names the field contract. | Medium. Multiple candidate conventions exist across repos. |

## 4. Memory / Intelligence Sources

| Surface | Path(s) | Live / Planned / Unknown | Charlie Relevance |
|---|---|---|---|
| Memory container policy | `workspace/core/MEMORY-POLICY.md` | Live-by-docs / partially planned | Use container separation as doctrine: operational memory, curated memory, legacy archive; Charlie must not manually write durable atoms. |
| Ingestion registry | `workspace/core/ingestion-registry.json` | Planned/partial | Shows manual, snapshot, and promotion recipes. Treat promotion as governor-gated. |
| Memory promotion config | `workspace/config/promotion-config.json`, `cron/jobs.json` | Live-by-docs / unverified | Shows thresholded promotion and proposal output path. Charlie should emit proposals, not promote to durable memory. |
| Memory atoms schema | `migrations/cap-mem-006/001_add_subjects_to_memory_atoms.sql` | Planned/applied-by-docs | Adds `subjects` for person-scoped memory. Charlie should remain compatible with subject tagging, not author schema. |
| Person registry | `migrations/cap-mem-006/002_create_person_registry.sql` | Planned/applied-by-docs | Canonical person table with tenant-scoped reader and admin roles. Charlie can depend on resolver output later. |
| Person resolution history | `migrations/cap-mem-006/003_create_person_resolution_history.sql`, `lib/person_resolver/resolution_writer.mjs` | Live-by-docs / behavior partial | Writer-path proof exists; plugin handlers are still skeletal. Charlie should not treat person resolution as fully automatic without Phase 2 verification. |
| Person resolver plugin | `extensions/person-resolver/index.ts` | Planned/partial | Hook skeleton only logs and does not inject or resolve. Charlie should mark CAP-MEM-006 as compatibility target, not runtime dependency. |
| Retrieval gate | `extensions/retrieval-gate/index.ts` | Live-by-docs / unverified | Orientation-constrained hybrid retrieval pattern. Hardcoded tenant/env paths must be parameterized for Charlie. |
| Cognitive event bus / IRL P1 | `workspace/lib/events/`, `workspace/lib/events/schemas/*.json`, `workspace/docs/obs-002-phase2-cognition.md` | Live-by-docs / unverified | Useful event taxonomy for corrections, plan rewrites, compression, retrieval, memory promotion, and decay. |
| Loop Charter / IRL governance | `workspace/docs/loop-charter.md` | Planned/partial | Governs optimization surfaces and self-promotion asymmetry. Charlie should respect it but not implement authority. |
| Dreaming / consolidation | `workspace/docs/CAP-CON-001_Build_Specification_v1_0.md`, `workspace/lib/dreaming_compactor/`, `SYSTEM_MAP.md` | Planned/partial | Future learning loop compatibility only. Do not claim Charlie has live L11 without runtime proof. |
| Session continuity | `extensions/session-continuity/index.ts`, `workspace/lib/contracts/schemas/continuity.ts` | Live-by-docs / unverified | Useful context compression pattern; Charlie can adopt through substrate, not by editing memory. |

## 5. Capability / Contract Surfaces

| Surface | Path(s) | Charlie Use | Notes |
|---|---|---|---|
| Capability registry | `registry/CAPABILITY_REGISTRY.md` | Reference/inherit | Source of layer status, capability states, queued follow-ons, and locked doctrine lines. Platform-owned; Charlie must not mutate it. |
| Structured output contracts | `workspace/lib/contracts/README.md`, `workspace/lib/contracts/registry.ts`, `workspace/lib/contracts/schemas/` | Inherit pattern | "No unstructured text crosses a layer boundary" is a core compatibility rule for Charlie substrate. |
| Orientation contract | `workspace/skills/prime/contract.mjs`, `extensions/prime-orientation/index.ts`, `workspace/lib/contracts/schemas/orientation.ts` | Reference | Normalizes orientation to `orientation.v1`; useful for prompt assembly and retrieval gating. |
| Sidecar schemas | `workspace/lib/contracts/schemas/*.ts` | Inherit pattern | Orientation, cognition, router, ensemble, plan, salience, continuity, retrieval, coordinator, trace, span, and cost-event schemas define substrate interfaces. |
| Cognitive/event schemas | `workspace/lib/events/types.ts`, `workspace/lib/events/schemas/*.json` | Compatible-with | Charlie should emit compatible event intents later, but event persistence is platform substrate. |
| Execution action registry | `workspace/lib/integrations/actions/registry.ts`, `workspace/lib/integrations/execute.mjs` | Reference | Business verbs are provider-independent. Charlie can propose actions; execution binding remains platform/Lyhna governed. |
| Credential vault | `migrations/2026_04_20_exec_003_credentials.sql`, `bin/credential-admin.mjs` | Avoid direct use | Shows RLS and narrow-role pattern. Charlie must not create, read, or manage credentials. |
| Binding table | `migrations/2026_04_22_exec_004_bindings.sql` | Reference | Tenant action bindings are platform-owned. Charlie should call/provider-plan against action names, not provider URLs. |
| MCP baseline | `workspace/docs/mcp-platform-baseline.md` | Compatible-with | MCP tools require predeclared schemas, sanitization, and Lyhna bind. No runtime discovery bypass. |
| Sanitization gate | `extensions/sanitization-gate/`, `tenants/*/sanitization/*.json` | Compatible-with | Charlie should depend on substrate gate, not embed allowlists or policy decisions in identity. |
| Proposal/promotion invariant | `docs/doctrine/CONSTRAINTS.md`, `workspace/core/TEMPLATE-SPINE.md`, `workspace/docs/loop-charter.md` | Inherit | Agents propose; governors promote. Any durable changes by Charlie must land as reviewable proposals. |

## 6. Execution / Credential / Deploy Surfaces

- `deploy/deploy-to-vps.sh` and `deploy/branch-from-vps.sh` are scaffold-only and intentionally exit before doing work. Charlie Phase 1 must not activate them, replace placeholders, set `DRY_RUN=0`, contact a VPS, or infer deploy details.
- `CLAUDE.md` assigns VPS integration to Claude Code, not Codex. Codex composes repo artifacts; Claude Code later pulls/activates on the VPS after governor approval.
- `migrations/2026_04_20_exec_003_credentials.sql` defines the credential vault pattern with narrow reader/admin roles and RLS. Charlie must not create credentials, tokens, URLs, keys, DSNs, or secret material.
- `bin/credential-admin.mjs` is a CLI for credential administration. It is a platform/operator tool, not a Charlie capability.
- `migrations/2026_04_22_exec_004_bindings.sql` and `workspace/lib/integrations/execute.mjs` define binding resolution and provider adapter dispatch. Charlie can produce normalized action requests later; it must not carry provider secrets or binding authority.
- `openclaw.json.example` is a redacted runtime config template but still includes environment key names, provider config, runtime paths, and live-looking identifiers. It should be reference-only and never copied into Charlie as config.
- `workspace/docs/exec-003-external-tool-execution.md` and `workspace/docs/exec-004-integration-execution-mapping.md` are the best conceptual references for external action flow.
- `workspace/docs/mcp-platform-baseline.md` states the standing chain: sanitization gate first, then Lyhna bind for MCP-backed tool calls. Charlie must remain compatible with that chain and not implement its own authority verifier.

## 7. Charlie Build Inputs from Arlo-core

| Asset | Source Path | Use in Charlie | Copy / Reference / Defer |
|---|---|---|---|
| Core work rules | `CLAUDE.md` | Governs Codex/Claude split, no-VPS boundary, no auto-promotion, no source logic mutation. | Reference |
| Repo constraints | `docs/doctrine/CONSTRAINTS.md` | Charlie Phase 1 constraints and durable-write rules. | Reference |
| Template spine | `workspace/core/TEMPLATE-SPINE.md` | Shape Charlie's governor-reviewed SOUL/COGNITION drafts. | Copy/reference |
| Arlo cognition example | `workspace/core/COGNITION.md` | Model structure for Charlie cognition without copying Arlo-specific content. | Reference |
| Arlo SOUL example | `workspace/core/SOUL.md` | Model structure and invariants for Charlie identity. | Reference |
| Capability registry | `registry/CAPABILITY_REGISTRY.md` | Determine which platform surfaces Charlie must remain compatible with. | Reference |
| Contract framework | `workspace/lib/contracts/` | Structured sidecar/schema boundary for Charlie substrate integration. | Reference |
| Event bus types | `workspace/lib/events/` | Event taxonomy compatibility for future observability/learning. | Reference |
| Memory policy | `workspace/core/MEMORY-POLICY.md` | Memory container and L12 boundary doctrine. | Reference |
| Person resolver surfaces | `migrations/cap-mem-006/`, `lib/person_resolver/`, `extensions/person-resolver/` | CAP-MEM-006 compatibility, not a live dependency. | Defer |
| Execution action registry | `workspace/lib/integrations/actions/registry.ts` | Provider-independent business-action vocabulary. | Reference |
| Execution docs | `workspace/docs/exec-003-external-tool-execution.md`, `workspace/docs/exec-004-integration-execution-mapping.md` | External action and binding concepts. | Reference |
| MCP baseline | `workspace/docs/mcp-platform-baseline.md` | Tool-governance and sanitization/Lyhna chain. | Reference |
| Deploy scaffolds | `deploy/*.sh` | Boundary awareness only. | Defer |

## 8. Phase 1 Constraints for Charlie

- Do not contact the VPS.
- Do not deploy, start, or stand up any substrate.
- Do not create, copy, or invent credentials, tokens, keys, DSNs, secrets, provider URLs, or live host values.
- Do not modify arlo-core source logic.
- Do not put authority logic inside Charlie.
- Do not implement Lyhna behavior inside Charlie.
- Do not perform direct durable self-writes to memory, skills, doctrine, capability registry, contracts, or identity files.
- Do not author final `SOUL.md` for Charlie as if governor-approved. Draft only, if later requested.
- Do not treat planned or snapshot-described substrate as live without verification.
- Do not copy Arlo tenant identity, Frostbite content, personal media, runtime logs, or memory state into Charlie.
- Do not auto-load or enable agent-authored plugins or skills without governor review.
- Do not run migrations or execute admin CLIs.

## 9. Open Questions

- Should Charlie Phase 1 produce draft `SOUL.md` and `COGNITION.md`, or only a composition spec that waits for governor identity authorship?
- Which tenant/governor naming should Charlie use in generated metadata, if any, before a dedicated Charlie repo exists?
- Should Charlie's first substrate target be pure Hermes-derived runtime with Arlo-core contracts layered on top, or OpenClaw plugin substrate with Hermes modules lifted selectively?
- Is CAP-MEM-006 person resolver considered available for Charlie Phase 2, or should Charlie use generic memory retrieval until resolver behavior is implemented and verified?
- Should `registry/CAPABILITY_REGISTRY.md` be treated as authoritative despite `SYSTEM_MAP.md` warning that live state can diverge from disk artifacts?
- Should Charlie inherit Arlo's five-layer learning separation exactly, or should the governor narrow Charlie's allowed lower-layer auto-updates further for the initial standup?
- What is the approved proposal directory/queue for Charlie durable-change proposals if the promotion queue is still build-pending?

## 10. Readiness Verdict

READY WITH WARNINGS.

Arlo-core is ready for Codex to run the Charlie build doctrine as a repo-only composition pass. The doctrine, constraints, identity template, contract framework, capability registry, and execution boundaries are present and coherent. The warnings are material: some live-state claims are snapshots rather than verified runtime facts; the requested constraints path differs from the actual path; several memory/intelligence surfaces are planned or skeletal; and the repo includes runtime templates and tenant artifacts that Charlie must not copy. The next safe action is a governor-approved Charlie composition pass that references these assets, drafts only proposal artifacts, and leaves deployment, credentials, authority, and durable promotion outside Charlie.
