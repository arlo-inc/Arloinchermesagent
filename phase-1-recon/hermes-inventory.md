# Hermes Inventory

Source: `NousResearch/hermes-agent` main branch at `5d3be898a8671eb9fb99cf18f43165502f54e7f4`. Phase 1 inspection was read-only: source, manifests, docs, and tests were sampled; dependencies were not installed and the repo was not run.

## Top-Level Structure

| Path | Files | What Lives There | Subsystem | Patterns / Dependencies / Test Shape |
| --- | ---: | --- | --- | --- |
| `.github/` | 15 | GitHub workflows and metadata. | CI / delivery | Much smaller than OpenClaw; tests are Python-heavy. |
| `.plans/` | 2 | Planning docs. | Project planning | Not runtime. |
| `acp_adapter/` | 9 | Agent Client Protocol adapter, auth, permissions, sessions, events, server entry. | ACP integration | Python package entrypoint `hermes-acp`; session bridge around Hermes agent. |
| `acp_registry/` | 2 | ACP registry metadata and icon. | ACP packaging | Static descriptor. |
| `agent/` | 56 | Core agent modules: prompt assembly, context compression, memory provider ABC, provider adapters, skill processing, transports, Google/Codex/Anthropic adapters, trajectory. | Agent loop substrate | Python modules with ABCs and clean provider seams mixed with some monolithic root files. |
| `assets/` | 1 | Banner image. | Static asset | Not runtime. |
| `cron/` | 3 | Scheduler and job persistence. | Cron | File-backed jobs plus croniter dependency; gateway ticks every 60 seconds. |
| `datagen-config-examples/` | 4 | Data generation examples. | Research/data | Not core Charlie substrate unless Phase 2 includes RL/data generation. |
| `docker/` | 2 | Docker packaging. | Packaging | Not inspected as runnable. |
| `docs/` | 1 | Minimal in-repo docs pointer. | Documentation | Most docs are external website content. |
| `environments/` | 43 | Benchmarks and RL environments. | Research / training substrate | Atropos/Tinker optional dependencies; not necessary for Phase 2 standup. |
| `gateway/` | 57 | Messaging gateway, sessions, delivery, platform adapters, hooks, pairing, channel directory. | Multi-platform messaging | Python platform adapter classes; plugin platform registry overlays legacy built-ins. |
| `hermes_cli/` | 66 | CLI commands: setup, gateway, model, providers, cron, hooks, kanban, plugins, web server, skills hub, memory setup, TUI support. | Operator CLI and config | Fire CLI via `hermes_cli.main:main`; significant configuration surface. |
| `nix/` | 11 | Nix packaging. | Packaging | Optional distribution path. |
| `optional-skills/` | 187 | Optional skill packs. | Skills | Additional Markdown procedural memory. |
| `packaging/` | 2 | Homebrew packaging. | Distribution | Not runtime. |
| `plans/` | 1 | Planning artifact. | Planning | Not runtime. |
| `plugins/` | 97 | Bundled plugins: memory providers, context engine discovery, image generation backends, Google Meet, observability, Spotify, platform plugins, dashboard plugin. | Plugin and provider layer | `plugin.yaml` + `__init__.py register(ctx)` pattern; special discovery paths for memory/context providers. |
| `scripts/` | 19 | Install/update/test helper scripts. | Tooling | Not executed. |
| `skills/` | 550 | Bundled skills across creative, productivity, research, devops, GitHub, software development, MCP, media, etc. | Skills system | Markdown `SKILL.md` with frontmatter and nested references/templates. |
| `tests/` | 925 | Unit, integration, stress, e2e, gateway, tools, plugin, and CLI tests. | Test suite | Broad Python pytest coverage; `pyproject` runs `pytest -m 'not integration' -n auto`. |
| `tools/` | 88 | Tool modules: terminal, memory, session search, MCP, skills, Kanban, send message, image generation, TTS/STT, web, file ops, environment backends. | Tool dispatch substrate | Central self-registration registry. |
| `tui_gateway/` | 8 | WebSocket/TUI gateway bridge. | TUI messaging bridge | Event publisher, slash worker, server, transport. |
| `ui-tui/` | 304 | Ink/React terminal UI frontend. | TUI dashboard / terminal app | Node/TypeScript package for interactive terminal UI. |
| `web/` | 90 | Local dashboard SPA and API client. | Dashboard / operator console | React/Vite frontend with pages for Chat, Cron, Skills, Sessions, Plugins, Models, Logs, Config, Analytics. |
| `website/` | 332 | Documentation website. | Docs site | Node/React docs; package separate from runtime. |

Primary root files: `run_agent.py` (large agent loop), `cli.py` (interactive CLI), `hermes_state.py` (SQLite session store), `trajectory_compressor.py`, `model_tools.py`, `toolsets.py`, `batch_runner.py`, `mcp_serve.py`, `pyproject.toml`, `uv.lock`, `package.json`, release notes. External dependencies of significance include Python 3.11+, OpenAI, Anthropic, httpx, Rich, prompt_toolkit, Pydantic, Jinja2, PyYAML, croniter, edge-tts, optional messaging SDKs, optional voice/STT, optional MCP, Honcho, ACP, FastAPI/Uvicorn dashboard, Modal/Daytona/Vercel terminal backends, Mistral/Bedrock extras, and RL/data extras.

Test coverage shape: tests are concentrated in `gateway` (220), `tools` (182), `hermes_cli` (170), `agent` (80), `run_agent` (79), `cli` (52), `cron` (11), `acp`/`acp_adapter` (13 combined), plugins (19), plus root/session/search and integration/stress/e2e folders. This is less sprawling than OpenClaw but closer to the agent behavior Arlo wants to exercise.

## Named Subsystem Catalog

### Agent Loop

- Lives in `run_agent.py`, `cli.py`, `model_tools.py`, `toolsets.py`, `agent/*`.
- Interface: `hermes`, `hermes-agent`, slash commands, `AIAgent`, tool registry, toolsets, provider adapters, config file.
- Strategy: a Python-centered agent loop assembles prompts, discovers tools, handles slash commands, manages context compression, routes provider calls, and delegates tool execution. Some key surfaces are monolithic (`run_agent.py` and `cli.py` are very large), but many newer seams live in `agent/` modules.
- Carry-forward note: Hermes exposes the most complete single-agent behavior loop, but Phase 2 should lift interfaces and behavior selectively rather than treat `run_agent.py` as a library boundary.

### Tool Registry / Toolsets

- Lives in `tools/registry.py`, `model_tools.py`, `toolsets.py`, individual `tools/*.py`.
- Interface: module-level `registry.register(...)`, `ToolRegistry`, toolset aliases, availability checks, cached tool definitions, plugin tool registration.
- Strategy: built-in tools self-register after AST discovery/import, then `model_tools.py` queries a generation-counter registry for OpenAI-format tool definitions. Toolsets gate which tools are exposed for CLI, gateway, cron, subagents, Kanban workers, and plugins.
- Carry-forward note: clean and pragmatic; useful for Charlie if Arlo needs a compact tool dispatch layer.

### Memory Backend

- Lives in `agent/memory_provider.py`, `agent/memory_manager.py`, `tools/memory_tool.py`, plugins under `plugins/memory/*`.
- Interface: `MemoryProvider` ABC selected by `memory.provider` config; lifecycle methods include `initialize`, `system_prompt_block`, `prefetch`, `queue_prefetch`, `sync_turn`, `get_tool_schemas`, `handle_tool_call`, `on_pre_compress`, `on_delegation`.
- Strategy: built-in memory is always active and external providers are additive, but only one external provider is active at a time to prevent schema bloat and conflicting writes. Bundled providers include Byterover, Hindsight, Holographic local SQLite, Honcho, Mem0, OpenViking, RetainDB, and Supermemory.
- Carry-forward note: best behavioral match for Arlo's learning-loop concerns. Tenant isolation requires scoping provider identity, config, and remote backend credentials per tenant/profile.

### Skills System

- Lives in `skills`, `optional-skills`, `agent/skill_utils.py`, `agent/skill_preprocessing.py`, `agent/skill_commands.py`, `tools/skills_tool.py`, `tools/skill_manager_tool.py`, `tools/skills_hub.py`, `hermes_cli/skills_*`.
- Interface: Markdown `SKILL.md`, YAML frontmatter, disabled/platform-disabled config, external skill dirs, Skills Hub sync, `skills_list`, `skill_view`, `skill_manage`.
- Strategy: Hermes treats skills as procedural memory. Prompt assembly indexes descriptions/frontmatter; the agent is prompted to create or patch skills after complex workflows and can manage skills through tools.
- Carry-forward note: strongest skills system for Charlie because it includes discovery, execution guidance, and self-improvement behavior.

### Kanban / Task Substrate

- Lives in `hermes_cli/kanban_db.py`, `hermes_cli/kanban.py`, `tools/kanban_tools.py`, `toolsets.py`.
- Interface: shared `$HERMES_HOME/kanban.db`, `kanban_*` tools, CLI commands, worker env vars (`HERMES_KANBAN_TASK`, `HERMES_KANBAN_WORKSPACE`), dispatcher.
- Strategy: SQLite board with WAL, CAS claim locks, tasks, task links, task comments, events, task runs, heartbeat/reclaim, durable handoffs, per-task worker contexts, and workspace kinds (`scratch`, `worktree`, `dir`).
- Carry-forward note: better than OpenClaw's task ledger for multi-worker standup. The schema already has a `tenant` field, but Arlo must enforce tenant scoping at every query/tool boundary.

### Subagent Delegation

- Lives in `tools/terminal_tool.py`, `model_tools.py`, `toolsets.py`, `run_agent.py`, `hermes_state.py`.
- Interface: `delegate_task` tool, subagent toolset, parent/child session lineage, memory provider `on_delegation`.
- Strategy: delegate tasks run in isolated contexts and are explicitly included in the agent-loop toolset. Session lineage distinguishes delegate subagents from compression continuations and branches.
- Carry-forward note: more directly useful for Charlie than OpenClaw's lower-level session spawn tools, but tenant and authority metadata must be propagated into child sessions.

### Context Engine / Compression

- Lives in `agent/context_engine.py`, `agent/context_compressor.py`, `trajectory_compressor.py`, `plugins/context_engine`.
- Interface: `ContextEngine` ABC selected by `context.engine`; methods include `update_from_response`, `should_compress`, `compress`, lifecycle hooks, optional tool schemas.
- Strategy: built-in compressor summarizes middle turns, protects head/tail, redacts sensitive text, handles multimodal token budgeting, preserves active task framing, and supports guided manual compression. Plugin discovery allows alternative engines under `plugins/context_engine/<name>`.
- Carry-forward note: Hermes has the better concrete compressor. OpenClaw has the cleaner slot registry.

### Session Storage

- Lives in `hermes_state.py`, `tools/session_search_tool.py`, `gateway/session.py`, `gateway/session_context.py`.
- Interface: SQLite database with `sessions`, `messages`, FTS5, session lineage (`parent_session_id`), title resolution, compression tips, session search.
- Strategy: persistent SQLite session store replaces per-session JSONL files, uses WAL for concurrent gateway/CLI readers, FTS5 for search, lineage projection for compression continuations, and child filtering to hide delegate subagents unless requested.
- Carry-forward note: best session substrate in either repo. Tenant isolation needs `tenant_id` columns or per-tenant DBs plus query enforcement.

### Prompt Assembly

- Lives in `agent/prompt_builder.py`, `run_agent.py`, `agent/prompt_caching.py`, `agent/skill_utils.py`, `agent/memory_manager.py`.
- Interface: stateless prompt builder functions, context files (`.hermes.md`, `HERMES.md`, `SOUL.md`, AGENTS-like files), skill index, memory prompt blocks, platform hints.
- Strategy: prompt assembly combines identity, memory guidance, session search guidance, skills index, Kanban worker guidance, context files, and platform hints. It also scans context files for prompt injection patterns before injection.
- Carry-forward note: good behavioral guidance, though Arlo should author the final layered prompt contract to keep tenant/moat policy authoritative.

### Gateway

- Lives in `gateway`, `gateway/platforms`, `gateway/platform_registry.py`, `tui_gateway`.
- Interface: `hermes gateway`, platform adapters, pairing/auth, channel directory, delivery, builtin hooks, pre-dispatch plugin hook, home target env vars.
- Strategy: one gateway process dispatches Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Mattermost, Home Assistant, DingTalk, Feishu, WeCom/Weixin, SMS, Email, Webhook, BlueBubbles, QQBot, Yuanbao, and plugin platforms. Platform plugins can override or extend built-ins through `PlatformRegistry`.
- Carry-forward note: capable but less broad than OpenClaw. Better if Charlie needs Python-local integration with Hermes session/memory/skills rather than maximum platform coverage.

### Cron / Scheduling

- Lives in `cron/jobs.py`, `cron/scheduler.py`, `hermes_cli/cron.py`, `tools/cronjob` registration.
- Interface: cronjob tool, `hermes cron`, file-backed jobs in `$HERMES_HOME/cron`, delivery targets, platform delivery.
- Strategy: gateway calls scheduler tick every 60 seconds, uses a file lock to prevent overlapping ticks, croniter for due jobs, and launches agent jobs with resolved toolsets and delivery targets.
- Carry-forward note: adequate for Charlie but less hardened than OpenClaw's isolated cron session model and run-state split.

### Event Hooks

- Lives in `hermes_cli/plugins.py`, `gateway/hooks.py`, `agent/shell_hooks.py`, `hermes_cli/hooks.py`.
- Interface: plugin hooks such as `pre_tool_call`, `post_tool_call`, `transform_terminal_output`, `transform_tool_result`, `pre_llm_call`, `post_llm_call`, `pre_api_request`, `post_api_request`, session lifecycle hooks, `pre_gateway_dispatch`, approval lifecycle hooks.
- Strategy: generic plugin hook registry with fail-open observer patterns and a blocking path for pre-tool/pre-gateway behavior. Hooks are useful for observability, policy, and output transformation.
- Carry-forward note: excellent interception points for Arlo governance if wrapped so tenant policy cannot be bypassed by user/project plugins.

### MCP

- Lives in `tools/mcp_tool.py`, `hermes_cli/mcp_config.py`, `mcp_serve.py`.
- Interface: `mcp_servers` config, stdio and HTTP/StreamableHTTP transports, discovered tools registered into the central tool registry, server-initiated sampling support.
- Strategy: a background asyncio loop manages long-lived MCP sessions; tool calls are scheduled thread-safely, stderr is redirected to profile logs, timeouts/reconnects are configurable, and credential stripping is considered in error paths.
- Carry-forward note: Hermes has the better MCP client substrate. OpenClaw has stronger MCP server exposure.

### Voice

- Lives in `hermes_cli/voice.py`, `tools/voice_mode.py`, `tools/transcription_tools.py`, `tools/tts_tool.py`, `tools/neutts_synth.py`, optional extras `voice`, `tts-premium`.
- Interface: voice extra, edge-tts core dependency, ElevenLabs premium extra, faster-whisper/sounddevice optional STT, voice-related tools and CLI commands.
- Strategy: pragmatic CLI/gateway voice support rather than a full multimodal device node architecture.
- Carry-forward note: enough for voice memo transcription and TTS, but OpenClaw is stronger for modality routing and device surfaces.

### Dashboard / Operator Console

- Lives in `web`, `hermes_cli/web_server.py`, `plugins/*/dashboard`, `plugins/example-dashboard`, `plugins/strike-freedom-cockpit`, `plugins/hermes-achievements/dashboard`.
- Interface: `hermes dashboard` optional `web` extra, local FastAPI/Uvicorn API plus Vite/React frontend; plugin dashboard manifests/API modules.
- Strategy: local dashboard has pages for Chat, Cron, Skills, Sessions, Plugins, Models, Logs, Config, Analytics, and plugin pages. The plugin dashboard pattern is lighter and less centralized than OpenClaw's control UI descriptor system.
- Carry-forward note: useful for operator ergonomics, but Arlo should decide dashboard scope in Phase 2 rather than inheriting plugin-dashboard semantics wholesale.

### Plugin Discovery

- Lives in `hermes_cli/plugins.py`, `plugins/*`, `plugins/memory/__init__.py`, `plugins/context_engine/__init__.py`.
- Interface: bundled plugins, user plugins in `~/.hermes/plugins`, project plugins in `./.hermes/plugins` behind `HERMES_ENABLE_PROJECT_PLUGINS`, pip entry points under `hermes_agent.plugins`, `plugin.yaml`, `register(ctx)`.
- Strategy: source precedence is bundled -> user -> project -> pip, with later sources overriding earlier names. Plugins are opt-in by default via `plugins.enabled`; memory/context providers have specialized discovery and active-provider config.
- Carry-forward note: simpler than OpenClaw and easier to audit, but project plugins are dangerous in multi-tenant mode unless disabled or heavily sandboxed.

### Provider Routing

- Lives in `hermes_cli/providers.py`, `hermes_cli/model_catalog.py`, `agent/transports`, `agent/anthropic_adapter.py`, `agent/bedrock_adapter.py`, `agent/codex_responses_adapter.py`, `agent/gemini_*`, `agent/google_code_assist.py`, `agent/model_metadata.py`.
- Interface: `hermes model`, provider definitions from models.dev plus Hermes overlays, user `providers:` config, model catalog overrides, transport adapters.
- Strategy: provider routing normalizes provider identities and aliases, merges models.dev with in-repo overlays/user config, and isolates API-specific message conversion in adapters. README advertises Nous Portal, OpenRouter, NVIDIA NIM, Xiaomi, z.ai, Kimi/Moonshot, MiniMax, Hugging Face, OpenAI, custom endpoints, and more.
- Carry-forward note: good enough for Charlie, but OpenClaw's provider plugin matrix is broader and more packageable.

## Integrations List

Hermes advertises fewer integrations than OpenClaw, but most are real code paths. The broadest integration surfaces are platform adapters, memory providers, image/provider plugins, terminal backends, and skills.

### Messaging / Platform Adapters

| Integration | Directory | Shape |
| --- | --- | --- |
| Telegram | `gateway/platforms/telegram.py`, `telegram_network.py` | Built-in platform adapter. |
| Discord | `gateway/platforms/discord.py` | Built-in platform adapter. |
| Slack | `gateway/platforms/slack.py`, `hermes_cli/slack_cli.py` | Built-in platform adapter and CLI helpers. |
| WhatsApp | `gateway/platforms/whatsapp.py`, `gateway/whatsapp_identity.py`, `scripts/whatsapp-bridge` | Built-in adapter plus bridge package. |
| Signal | `gateway/platforms/signal.py`, `signal_rate_limit.py` | Built-in platform adapter. |
| Matrix | `gateway/platforms/matrix.py` | Built-in platform adapter. |
| Mattermost | `gateway/platforms/mattermost.py` | Built-in platform adapter. |
| Home Assistant | `gateway/platforms/homeassistant.py`, `tools/homeassistant_tool.py` | Built-in adapter/tool. |
| DingTalk | `gateway/platforms/dingtalk.py`, `hermes_cli/dingtalk_auth.py` | Built-in adapter and auth helper. |
| Feishu/Lark | `gateway/platforms/feishu.py`, `feishu_comment.py`, `tools/feishu_*` | Built-in adapter plus doc/drive tools. |
| WeCom / Weixin | `gateway/platforms/wecom*.py`, `weixin.py` | Built-in adapters. |
| SMS | `gateway/platforms/sms.py` | Built-in adapter. |
| Email | `gateway/platforms/email.py` | Built-in adapter. |
| Webhook | `gateway/platforms/webhook.py`, `hermes_cli/webhook.py` | Built-in webhook adapter. |
| BlueBubbles | `gateway/platforms/bluebubbles.py` | Built-in adapter. |
| QQBot | `gateway/platforms/qqbot/*` | Built-in adapter package. |
| Yuanbao | `gateway/platforms/yuanbao*.py`, `tools/yuanbao_tools.py` | Built-in adapter/tooling. |
| IRC | `plugins/platforms/irc` | Plugin platform adapter. |
| Microsoft Teams | `plugins/platforms/teams` | Plugin platform adapter. |

### Memory Providers

| Integration | Directory | Shape |
| --- | --- | --- |
| Built-in MEMORY/USER files | `agent/memory_manager.py`, `tools/memory_tool.py` | Always-on local memory. |
| Byterover | `plugins/memory/byterover` | Memory provider plugin. |
| Hindsight | `plugins/memory/hindsight` | Memory provider plugin. |
| Holographic | `plugins/memory/holographic` | Local SQLite memory provider plugin. |
| Honcho | `plugins/memory/honcho` | User-model memory provider plugin. |
| Mem0 | `plugins/memory/mem0` | Server-side memory provider plugin. |
| OpenViking | `plugins/memory/openviking` | Context DB memory provider plugin. |
| RetainDB | `plugins/memory/retaindb` | Cloud memory provider plugin. |
| Supermemory | `plugins/memory/supermemory` | Semantic memory provider plugin. |

### Plugins / Tools / Backends

| Integration | Directory | Shape |
| --- | --- | --- |
| Google Meet | `plugins/google_meet` | Standalone plugin. |
| Spotify | `plugins/spotify` | Backend/tool plugin with PKCE OAuth. |
| Langfuse | `plugins/observability/langfuse` | Observability plugin. |
| Disk cleanup | `plugins/disk-cleanup` | Hook/tool plugin. |
| Hermes achievements | `plugins/hermes-achievements` | Dashboard/plugin API. |
| Strike Freedom cockpit | `plugins/strike-freedom-cockpit` | Dashboard plugin. |
| OpenAI image generation | `plugins/image_gen/openai` | Image generation backend plugin. |
| OpenAI Codex image generation | `plugins/image_gen/openai-codex` | Image backend via Codex/ChatGPT surface. |
| xAI image generation | `plugins/image_gen/xai` | Image generation backend plugin. |
| MCP servers | `tools/mcp_tool.py` | Dynamic external tool integration. |
| Terminal backends | `tools/terminal_tool.py`, `tools/environments/*` | Local, Docker, SSH, Daytona, Singularity, Modal, Vercel. |
| ACP | `acp_adapter`, `acp_registry` | ACP server/adapter. |

### Model / Provider Integrations

Hermes primarily routes models through provider definitions rather than one plugin directory per provider. Concrete code exists for OpenAI-wire transports, Anthropic Messages, Bedrock, Codex Responses, Gemini/Google Code Assist, OpenRouter catalog overrides, Nous runtime provider, Moonshot/Kimi/DeepSeek/MiniMax compatibility handling, Mistral/Bedrock extras, NVIDIA/Xiaomi/z.ai/Hugging Face/custom endpoints through provider config and catalogs.

### Skills

Bundled skills are real file-based integrations rather than API adapters. Categories include Apple, autonomous AI agents, creative, data science, devops, diagramming, email, gaming, GitHub, inference, MCP, media, MLOps, note-taking, productivity, red teaming, research, smart home, social media, software development, and Yuanbao. The shape is Markdown instructions with optional references, templates, scripts, tests, and platform frontmatter.

## Architectural Notes Worth Carrying Forward

- Hermes is much closer to the "self-improving agent" posture: memory nudges, skill creation/patching, session search, Kanban delegation, and compression are first-class in the agent prompt and tools.
- The best substrate pieces are Python-native and behavior-level rather than distribution-level: session SQLite/FTS, Kanban, skills, memory provider ABC, MCP client, and compression.
- Some runtime boundaries are still monolithic. `run_agent.py` and `cli.py` are too large to lift wholesale into Arlo without creating a maintenance trap.
- Hermes already contains an OpenClaw migration command and OpenClaw contains a Hermes migration extension. The codebases know about each other; that contradicts any assumption that they are cleanly independent substrates.
- Hermes's plugin system is simpler and more auditable than OpenClaw's, but OpenClaw's is more comprehensive as a packaging/extension contract.
