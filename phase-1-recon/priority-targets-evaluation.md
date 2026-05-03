# Priority Extraction Targets Evaluation

This supplement evaluates the six Hermes modules named in the Phase 1 addendum. The original keep-list was already committed, so this file is an additive decision artifact.

## Summary Table

| # | Target | Confirm / Refute | Arlo Integration Cost | Tier B Adaptation | Entanglement |
| ---: | --- | --- | --- | --- | --- |
| 1 | `skills/` + skill creation logic | Confirm: real, useful, high leverage | Medium-high | High: all create/patch/delete/write-file flows become governor-approved proposals | Moderately entangled with prompt builder, skills hub, usage telemetry, curator, CLI, and tool registry |
| 2 | `cron/` curator jobs | Partially refute label: curator is real, but not housed in `cron/` | Medium | High for mutating curator passes; low for read-only reports | Entangled with skill usage, skill manager, auxiliary agent, CLI idle loop |
| 3 | Honcho + memory provider interface | Confirm: real and useful | Medium-high | High: provider writes/conclusions/profile updates need approval gateway | Entangled with Hermes config/profile/session identity and remote provider semantics |
| 4 | `delegate_tool.py` / subagent spawning | Confirm with path correction: `tools/delegate_tool.py` | Medium | Medium: child agents need proposal-only durable rights | Entangled with AIAgent internals, terminal tool, toolsets, threading, approvals |
| 5 | `hermes_state.py` lazy session creation | Confirm: real and useful | Medium | Low-medium: session records are okay; derived durable insights need approval | Large single file, but DB boundary is portable |
| 6 | Trajectory compression + Atropos integration | Confirm as future-facing, not Phase 2-critical | High | Medium: training-data promotion requires approval/redaction | Highly entangled with model/tool formats, terminal backends, optional RL deps |

## 1. `skills/` + Skill Creation Logic

Finding: confirmed. Hermes has a real file-based skills catalog (`skills/`, `optional-skills/`) plus agent-facing management tools in `tools/skill_manager_tool.py`, read/discovery logic in `tools/skills_tool.py` and `agent/skill_utils.py`, usage telemetry in `tools/skill_usage.py`, and prompt guidance in `agent/prompt_builder.py`. The tool explicitly allows create, edit, patch, delete, write support files, and remove support files under `~/.hermes/skills`.

Value: high. This is one of the strongest pieces of Hermes for Charlie because it turns repeated procedures into reusable substrate instructions and gives the agent a way to reuse learned workflows.

Integration cost into Arlo moat: medium-high. Arlo can lift the skill file shape and discovery/indexing model, but must replace direct local writes with tenant-scoped catalog APIs and introduce versions/locks.

Tier B adaptation: high. The autonomous learning loop violates "agents propose, governors promote" as written. `skill_manage` should emit `SkillProposal` records with diffs, rationale, source transcript, risk scan, and desired catalog target. Only a governor promotion service should materialize active skills.

Entanglement: moderate. The pieces are spread through prompt assembly, CLI, Skills Hub, usage telemetry, curator, and tool registry. It is portable if lifted as a behavior contract, not as a direct copy of every path.

Recommendation: lift the file format and skill-index behavior, but gate all write paths before Phase 2 Charlie can self-improve.

## 2. `cron/` Curator Jobs

Finding: partially refuted as named. Hermes has real cron in `cron/jobs.py` and `cron/scheduler.py`, and it has a real curator in `agent/curator.py` plus `hermes_cli/curator.py`, but the curator is not primarily a `cron/` module. Its docstring says it is inactivity-triggered, "no cron daemon," and runs after idle/interval gates; CLI can also run it manually.

Value: medium-high for skill hygiene, but dangerous by default. The curator tracks agent-created skills, marks stale/archived states, spawns an auxiliary review agent, writes reports, and can patch/consolidate/archive through `skill_manage`.

Integration cost into Arlo moat: medium. The scheduling/idle detection and report generation are straightforward; mutating actions require the promotion queue.

Tier B adaptation: high. Read-only curator reports and dry-runs are safe. Patch, consolidation, archive, unarchive, and lifecycle state changes should be proposals requiring governor promotion. Even "archive" is durable demotion and should not be autonomous in Arlo.

Entanglement: moderate-high. Curator depends on `tools/skill_usage.py`, `skill_manage`, auxiliary AIAgent behavior, config, CLI, and idle-loop wiring.

Recommendation: lift curator as a governor advisory/report generator, not as an autonomous maintainer.

## 3. Honcho Integration + Memory Provider Plugin Interface

Finding: confirmed. `agent/memory_provider.py` defines a real `MemoryProvider` ABC with lifecycle methods for initialization, prompt blocks, prefetch, queued recall, turn sync, tools, session switches, compression hooks, and delegation observation. `plugins/memory/honcho` implements Honcho-specific profile, search, context, reasoning, and conclusion tools, including persistent peer cards and conclusions.

Value: high. The memory provider interface is one of Hermes's best extraction targets because it isolates memory behavior from the core agent while allowing both passive recall and explicit memory tools.

Integration cost into Arlo moat: medium-high. The ABC is portable, but Arlo needs a provider gateway that owns tenant identity, provider credentials, write policy, data retention, remote namespace selection, and audit.

Tier B adaptation: high. Honcho-style `honcho_conclude`, `honcho_profile` updates, provider `sync_turn`, `on_delegation`, and `on_pre_compress` can create durable user/peer models. These must emit memory proposals or operate under a narrowly pre-approved governor policy.

Entanglement: moderate. The interface itself is clean, but concrete providers are tied to Hermes config, `HERMES_HOME`, session IDs, profile names, and provider-specific client semantics.

Recommendation: lift the provider lifecycle, not direct provider writes. Start Phase 2 with a tenant-local provider or proposal-only wrapper before enabling Honcho persistence.

## 4. `delegate_tool.py` / Subagent Spawning

Finding: confirmed with path correction. The file is `tools/delegate_tool.py`. It is a real subagent architecture: child `AIAgent` instances get fresh conversations, isolated task IDs, restricted toolsets, terminal/file state isolation, concurrency caps, depth caps, approval callbacks, active-subagent registry, interrupt/status controls, and parent-only summary results.

Value: high. This is a strong parallel work primitive for Charlie, especially paired with Kanban and session lineage.

Integration cost into Arlo moat: medium. The core behavior is portable, but the implementation reaches into AIAgent internals, terminal tools, toolsets, threading, approval callbacks, and config.

Tier B adaptation: medium. Delegation itself does not violate Tier B, but child agents must not inherit direct durable-write rights. Blocked tools already include `memory`, recursive `delegate_task`, `send_message`, and some risky tools; Arlo should extend this with proposal-only rights for memory, skills, plugins, cron, doctrine, and capability registry writes.

Entanglement: high in code, moderate in concept. The design is worth lifting; direct code extraction would need careful wrapping.

Recommendation: lift the primitive and policy model, then reimplement against Arlo's agent runner and capability registry.

## 5. `hermes_state.py` Lazy Session Creation Pattern

Finding: confirmed. `hermes_state.py` is a real SQLite session store with `ensure_session()` / `INSERT OR IGNORE`, WAL mode, retry with jitter, FTS5 plus trigram FTS, schema reconciliation, message storage, model/cost metadata, parent session lineage, title resolution, compression-chain projection, and session search support.

Value: very high. This is the cleanest durable session substrate found in either codebase.

Integration cost into Arlo moat: medium. It is one large file, but the database boundary is clear. Arlo can either adapt the schema or re-author it from the discovered pattern.

Tier B adaptation: low-medium. Raw session/message persistence is not a promotion of doctrine or memory by itself; it is audit/history. However, any derived insight generated from session search or compression that becomes a memory atom, skill, doctrine edit, or capability entry must go through the Tier B proposal gate.

Entanglement: moderate. It imports Hermes constants and memory sanitization and assumes a single `HERMES_HOME`, but the store can be tenantized with schema additions or per-tenant DB partitioning.

Recommendation: prioritize for Phase 2. Add tenant columns or per-tenant DBs before writing real tenant data.

## 6. Trajectory Compression + Atropos Integration

Finding: confirmed, but future-facing. `trajectory_compressor.py` is a real post-processing pipeline for ShareGPT-style trajectories. `agent/trajectory.py`, `run_agent.py`, and `batch_runner.py` save trajectories. `environments/README.md` describes a Hermes-Agent-to-Atropos integration with `HermesAgentBaseEnv`, tool context, tool-call parsers, reward functions, and two-phase OpenAI-server/VLLM-managed training modes. `tools/rl_training_tool.py` exposes Tinker-Atropos training lifecycle tools.

Value: medium for Charlie Phase 2, high for future training/RL-readiness. It is not required to stand up Charlie, but it is valuable if Arlo wants tenant-safe trajectory capture, evaluation, or future model training.

Integration cost into Arlo moat: high. This touches model/tool-call serialization, terminal/backend state, reward functions, optional Tinker/Atropos dependencies, WandB, server processes, and environment discovery.

Tier B adaptation: medium. Saving raw trajectories for audit/debug is acceptable with consent and redaction. Promoting trajectories into training datasets, RL runs, or durable agent improvement doctrine requires governor approval, tenant consent, PII/security redaction, and dataset provenance.

Entanglement: high. The RL tools assume Hermes tool registry, terminal backends, environment paths, and optional submodule/dependency layout. It is not a first extraction target for Charlie standup.

Recommendation: document and preserve for Phase 3+ training readiness; do not include in the minimal Charlie Phase 2 boot path.

## Updated Priority Order

1. `hermes_state.py` session store, tenantized.
2. Memory provider interface, proposal-gated.
3. Skills discovery/indexing, proposal-gated.
4. Delegation primitive, with durable-write rights removed or proposal-only.
5. Curator as governor advisory reports, not autonomous mutation.
6. Trajectory/Atropos path as future RL/data infrastructure after Charlie works.

## Phase 2 Implication

The keep-list recommendation should be sharpened: Phase 2 should not "turn on Hermes learning." It should stand up Charlie with Hermes-derived learning surfaces in **proposal mode**. Success means Charlie can identify a useful skill or memory atom, produce a structured proposal, and let a governor promote it. That is the correct test of Arlo's substrate interchangeability claim under the Tier B invariant.
