# Keep List Tier B Invariant Addendum

This addendum supplements `keep-list.md` after the original Artifact 3 commit. It applies the constitutional invariant: **agents propose, governors promote**. Any substrate pattern that lets an agent self-promote a durable skill, memory atom, capability entry, doctrine edit, `SOUL.md`, `COGNITION.md`, or plugin without governor approval must be gated before Arlo can lift it.

## Tier B Impact Summary

The original keep-list still holds directionally, but the cost profile changes. Hermes remains the best donor for memory, skills, Kanban, delegation, context compression, and session search, yet its learning loop is explicitly built around agent-curated persistence. OpenClaw remains the best donor for gateway, cron, voice, dashboard, and provider routing, but its memory flush and skill/plugin install paths also need governance before durable promotion.

Tier B does not make these systems unusable. It changes the integration shape from "write durable state" to "write a pending proposal artifact." Arlo should introduce a promotion queue with immutable proposal records, diffs, rationale, source session, tenant, agent, substrate concern, risk classification, and governor decision metadata.

## Self-Promotion Findings

| Instance | Repo / Location | Pattern | Concern(s) Affected | Tier B Risk | Gate Cost | Required Adaptation |
| --- | --- | --- | --- | --- | --- | --- |
| Hermes skill creation and patching | `tools/skill_manager_tool.py`, `agent/prompt_builder.py`, `tools/skill_usage.py`, `agent/curator.py` | Agent can create, edit, patch, write support files, remove files, delete/archive, and curator-maintain user skills in `~/.hermes/skills`. | Skills system, cron curator, plugin/skill discovery | High: direct durable procedural memory mutation. | Medium-high | Replace direct skill writes with `SkillProposal` records. Governor approves promotion into active skill catalog; curator can only propose consolidation, archive, patch, or pin changes. |
| Hermes built-in memory writes | `tools/memory_tool.py`, `run_agent.py`, `agent/memory_manager.py` | Agent calls `memory` tool to mutate `MEMORY.md` and `USER.md` immediately. Writes are durable and later injected into system prompt snapshots. | Memory backend, prompt assembly | High: direct durable declarative memory and user-model mutation. | Medium | Convert `add/replace/remove` to proposal mode for durable/cross-session atoms. Allow only ephemeral session notes without governor approval; promoted atoms require governor acceptance and tenant scoping. |
| Hermes external memory conclusions | `agent/memory_provider.py`, `plugins/memory/honcho`, other `plugins/memory/*` | Providers can prefetch, sync turns, observe delegation, and expose persistent conclusion/profile tools such as `honcho_conclude` and `honcho_profile` update. | Memory backend, subagent delegation | High: remote memory providers may persist user/peer models outside Arlo control. | High | Wrap provider writes behind Arlo memory gateway. Provider `sync_turn`, `on_delegation`, `on_pre_compress`, and explicit conclusion/profile writes become proposals unless governor has pre-authorized a narrow write policy. |
| Hermes curator | `agent/curator.py`, `hermes_cli/curator.py`, `tools/skill_usage.py` | Inactivity-triggered auxiliary agent can patch, consolidate, mark stale, archive, and maintain agent-created skills. | Skills system, cron/curator jobs | High: automated maintenance promotes or demotes durable procedural memory without governor review. | Medium-high | Curator becomes a report/proposal generator. Pure status and dry-run analysis are allowed; archive/patch/consolidation require governor promotion. |
| Hermes project/user/pip plugin discovery | `hermes_cli/plugins.py` | Bundled, user, project, and pip entry-point plugins can be discovered; project plugins are opt-in but can override earlier sources when enabled. | Plugin discovery, event hooks, tools | Medium-high: code/tool/hook capability may enter runtime from non-governed source. | Medium | Disable project plugins by default for tenants; require signed plugin packages, governor approval, provenance records, and grant-scoped activation. Agent-authored plugins are proposals only. |
| Hermes cron jobs with attached skills | `cron/*`, `tools/cronjob_tools.py`, `hermes_cli/cron.py` | Agent/user can create recurring prompts and attach skills. Cron may run later with durable delivery and toolsets. | Cron, skills, prompt assembly | Medium: scheduled work can repeatedly reinforce unreviewed prompts/skills. | Medium | Cron creation by an agent becomes a scheduled-job proposal unless within an approved capability. Attached skills must reference governor-promoted versions. |
| OpenClaw pre-compaction memory flush | `extensions/memory-core/src/flush-plan.ts`, `extensions/memory-core/index.ts` | Before compaction, an agent turn is instructed to capture durable memories to `memory/YYYY-MM-DD.md`, appending content to disk. | Memory backend, context engine, prompt assembly | High: direct durable memory extraction from agent judgment. | Medium | Route memory flush output to pending memory proposals. Governor promotes selected entries into tenant memory corpus; compaction summaries may cite unpromoted proposals only as session-local context. |
| OpenClaw short-term promotion / dreaming | `extensions/memory-core/index.ts`, `extensions/memory-core/src/dreaming*.ts` | Memory core registers short-term promotion/dreaming flows for memory consolidation. | Memory backend, learning loop | High if enabled to persist consolidated memories. | Medium-high | Dreaming outputs become candidate memory proposals with source citations and confidence. No autonomous promotion into durable tenant memory. |
| OpenClaw skills and ClawHub | `skills/clawhub/SKILL.md`, `src/commands/onboard-skills.ts`, `src/agents/skills*`, plugin bundle skills | Skills can be installed/configured and plugin bundles can declare skills; onboarding can install missing skill dependencies with user confirmation. | Skills system, plugin discovery | Medium: less agent-self-promoting than Hermes, but can still introduce durable procedural memory and dependencies. | Medium | Treat skill install/update/publish as governor-managed catalog operations. Agent may suggest a skill or dependency install, but active tenant catalogs change only after governor approval. |
| OpenClaw plugin installation/update | `src/plugins/update.ts`, `src/commands/onboarding-plugin-install.ts`, `packages/plugin-package-contract` | External plugins can be installed/updated from ClawHub/package metadata and then register tools, hooks, channels, providers, and skills. | Plugin discovery, gateway, hooks, providers | High: plugin activation expands capability surface. | Medium-high | Require signed package metadata, compatibility checks, tenant capability grants, and governor approval before activation. Auto-update should stage proposals, not mutate active plugin set. |

## Concern-Level Revisions to Keep List Reasoning

| Concern | Original Winner | Tier B Revision |
| --- | --- | --- |
| Memory backend | Hermes | Keep Hermes as abstraction winner, but only if Arlo inserts a governor-approved memory promotion queue before `MEMORY.md`, `USER.md`, Honcho conclusions, provider sync writes, and OpenClaw memory-flush imports. Without this gate, Hermes memory is constitutionally unsafe. |
| Skills system | Hermes | Keep Hermes for discovery and skill ergonomics, but direct `skill_manage` writes and curator maintenance must become proposals. The learning loop is valuable precisely because it proposes improvements; promotion must be human/governor-owned. |
| Kanban / task substrate | Hermes | Mostly safe if tasks are treated as work coordination, not doctrine. However, Kanban worker-created child tasks that install/promote skills, plugins, memory, or cron jobs must route those outputs to the Tier B proposal queue. |
| Subagent delegation | Hermes | Keep Hermes. Delegated agents must inherit proposal-only rights for durable memory/skill/plugin/doctrine writes unless explicitly granted by governor policy. |
| Context engine | Hermes behavior behind Arlo/OpenClaw slot | Compression summaries may be persisted as session lineage, but extracted "durable memories" from compression must be proposals. OpenClaw's memory flush is the key extra gate. |
| Session storage | Hermes | Session transcripts are records, not promoted doctrine. Search indexes are acceptable if tenant-scoped; any derived session insight/memory generated from them requires approval. |
| Prompt assembly | Arlo authors | Tier B reinforces the original decision. Arlo must own the system prompt layer that tells agents to propose durable changes rather than apply them. |
| Gateway | OpenClaw | No direct Tier B change, except inbound messages that trigger install/update/memory/skill actions must pass through governor approval. |
| Cron / scheduling | OpenClaw | Cron job creation and recurring self-improvement passes need approval when they can mutate durable memory, skills, plugins, doctrine, or capability registry. Routine read-only reporting can remain normal scheduled work. |
| Event hooks | Hermes | Hooks can implement the Tier B gate if ordered before user/project plugins. Any hook that auto-promotes durable state is unsafe unless converted to proposal emission. |
| MCP integration | Hermes | MCP tools that mutate durable memory, skills, plugins, docs, or registry entries must be capability-scoped and proposal-gated. Read/search MCP tools are lower risk. |
| Voice | OpenClaw | Voice-derived transcripts can be session records, but extracted user facts or voice/persona profile updates require governor promotion. |
| Dashboard / operator console | OpenClaw | Dashboard should become the governor promotion console: review proposals, compare diffs, approve/reject, audit. |
| Plugin discovery | Hermes | Keep Hermes's tier model only with default-deny project plugins and governor-approved package/user plugins. Agent-authored plugins are non-starters unless staged as proposals. |
| Provider routing | OpenClaw | Provider routing itself is not self-promotion, but provider/model policy changes or capability-registry entries created by agents must be proposals. |

## Non-Starters Without Gating

- Direct Hermes `skill_manage` promotion into active tenant skills.
- Direct Hermes `memory` writes into cross-session tenant memory.
- Direct Honcho or other provider conclusion/profile writes outside an Arlo approval wrapper.
- OpenClaw pre-compaction memory flush writing directly into tenant memory corpus.
- Agent-authored or project-local plugins auto-loading into an active tenant runtime.

## Recommended Tier B Adaptation

Introduce a single Arlo durable-promotion boundary shared by memory, skills, plugins, cron jobs, doctrine files, and capability registry changes:

1. Agent emits a proposal with source transcript, diff/content, target durable surface, confidence, risk flags, and tenant/agent/session metadata.
2. Proposal is stored as immutable pending state and may be shown in dashboard, CLI, or governor inbox.
3. Governor approves, rejects, edits, or asks for revision.
4. Only the promotion service writes to durable memory files, skill catalogs, plugin activation records, doctrine files, or capability registry.
5. Every promoted artifact records provenance and rollback metadata.

This preserves the value of Hermes's learning loop and OpenClaw's memory automation while honoring the constitutional asymmetry: agents propose; governors promote.
