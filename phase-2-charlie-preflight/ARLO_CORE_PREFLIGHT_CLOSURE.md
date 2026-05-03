# Arlo Core Preflight Closure

## Files Inspected

- `README.md`
- `CLAUDE.md`
- `.gitignore`
- `SYSTEM_MAP.md`
- `openclaw.json.example`
- `docs/README.md`
- `docs/doctrine/CONSTRAINTS.md`
- `docs/closures/TRACK_C_STAGE_2_CLOSURE.md`
- `registry/CAPABILITY_REGISTRY.md`
- `deploy/deploy-to-vps.sh`
- `deploy/branch-from-vps.sh`
- `workspace/core/COGNITION.md`
- `workspace/core/SOUL.md`
- `workspace/core/TEMPLATE-SPINE.md`
- `workspace/core/MEMORY-POLICY.md`
- `workspace/core/OPERATING-RULES.md`
- `workspace/core/tool-registry.json`
- `workspace/core/specialists.json`
- `workspace/core/ingestion-registry.json`
- `workspace/AGENTS.md`
- `workspace/IDENTITY.md`
- `workspace/ARLO.md`
- `workspace/SOUL.md`
- `workspace/ACTIVE.md`
- `workspace/config/promotion-config.json`
- `workspace/docs/exec-003-external-tool-execution.md`
- `workspace/docs/exec-004-integration-execution-mapping.md`
- `workspace/docs/mcp-platform-baseline.md`
- `workspace/docs/loop-charter.md`
- `workspace/docs/CAP-CON-001_Build_Specification_v1_0.md`
- `workspace/lib/contracts/README.md`
- `workspace/lib/contracts/registry.ts`
- `workspace/lib/events/types.ts`
- `workspace/lib/events/index.ts`
- `workspace/lib/events/emit.ts`
- `workspace/lib/events/schemas/memory_promote_event.json`
- `workspace/lib/integrations/execute.mjs`
- `workspace/lib/integrations/actions/registry.ts`
- `workspace/skills/prime/contract.mjs`
- `extensions/prime-orientation/index.ts`
- `extensions/person-resolver/index.ts`
- `extensions/retrieval-gate/index.ts`
- `extensions/session-continuity/index.ts`
- `extensions/skill-loader/index.ts`
- `migrations/2026_04_20_exec_003_credentials.sql`
- `migrations/2026_04_22_exec_004_bindings.sql`
- `migrations/cap-mem-006/001_add_subjects_to_memory_atoms.sql`
- `migrations/cap-mem-006/002_create_person_registry.sql`
- `migrations/cap-mem-006/003_create_person_resolution_history.sql`
- `lib/person_resolver/resolution_writer.mjs`
- `recon/cap-mem-006-phase3-writer-path-proof.md`
- `cron/jobs.json`

## Documents Created

- `phase-2-charlie-preflight/ARLO_CORE_DEPENDENCY_MAP.md`
- `phase-2-charlie-preflight/ARLO_CORE_PREFLIGHT_CLOSURE.md`

## Stage 2 Scaffolding

Arlo-core Stage 2 scaffolding is present. The repo has the root safety files (`README.md`, `CLAUDE.md`, `.gitignore`), doctrine constraints at `docs/doctrine/CONSTRAINTS.md`, and scaffold-only deploy scripts in `deploy/` that refuse execution until governor/Claude Code activation. The requested `docs/CONSTRAINTS.md` path is absent; the actual constraints path is one level deeper under `docs/doctrine/`.

## Surprises

The biggest surprise is that arlo-core combines clean platform scaffolding with a large staged workspace capture that includes runtime state, tenant/product artifacts, and live-looking config references. The second surprise is that CAP-MEM-006 has a proven writer-path report and migrations, but the checked person-resolver plugin remains a no-op skeleton. The third is that the docs strongly reinforce the right layer split for Charlie: Charlie should not carry authority, and Lyhna should not be reimplemented in the agent.

## Missing or Undetermined

I could not determine live runtime truth without standing up substrate or contacting the VPS, both of which were explicitly out of scope. I also could not verify whether L11/Dreaming and IRL surfaces are operational beyond the checked docs, staged code, and system map. A single canonical "agent card" file was not present in arlo-core; the best available convention is `workspace/core/TEMPLATE-SPINE.md`.

## Readiness Verdict

READY WITH WARNINGS. The repo is ready for a governed Charlie composition pass that references arlo-core doctrine and contracts, but not for deploy, credential wiring, source mutation, or authority implementation inside Charlie.

## Exact Next Recommended Action

Run the Charlie build doctrine only after governor approval, targeting repo-only composition artifacts. Use `workspace/core/TEMPLATE-SPINE.md`, `CLAUDE.md`, `docs/doctrine/CONSTRAINTS.md`, `registry/CAPABILITY_REGISTRY.md`, and `workspace/lib/contracts/` as primary inputs; keep Charlie's identity and durable-write outputs in proposal/draft mode until governor promotion.
