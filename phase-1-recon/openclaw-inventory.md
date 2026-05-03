# OpenClaw Inventory

Source: `openclaw/openclaw` at tag `v2026.4.29` (`a448042c2edd94a4e8ee86d5ed9fe8e4cd`). Phase 1 inspection was read-only: source, manifests, docs, and tests were sampled; dependencies were not installed and the repo was not run.

## Top-Level Structure

| Path | Files | What Lives There | Subsystem | Patterns / Dependencies / Test Shape |
| --- | ---: | --- | --- | --- |
| `.agents/` | 30 | Agent-authored local instructions and harness notes. | Repo operating policy | Markdown policy surface; no runtime dependency observed. |
| `.github/` | 80 | CI workflows, release validation, labels, issue automation. | Delivery and quality gates | Heavy GitHub Actions matrix; docs show split Node, plugin, channel, Docker, live-provider, and security lanes. |
| `.pi/` | 8 | Small local automation/config artifacts. | Project support | Not core substrate. |
| `.vscode/` | 2 | Editor settings. | Developer support | Not runtime. |
| `apps/` | 900 | Android, iOS, macOS companion apps. | Platform nodes and native surfaces | Swift/Kotlin/mobile app code, fastlane metadata; docs identify macOS menu bar, iOS/Android Talk, Canvas, Camera, Screen Capture, node command families. |
| `assets/` | 7 | Package assets. | Distribution | Static assets only. |
| `docs/` | 569 | Product and architecture docs: gateway, channels, automation, tools, plugins, models, security, platforms. | Operator documentation | Mature docs tree with generated sections and CI docs checks. |
| `extensions/` | 6,049 | Bundled plugins and adapters. | Integrations layer | Each real integration normally has `openclaw.plugin.json`, `package.json`, `index.ts`, and tests; core runtime packages omit manifests intentionally. |
| `git-hooks/` | 1 | Hook helper. | Developer workflow | Not runtime. |
| `packages/` | 130 | Published SDK packages: `sdk`, `plugin-sdk`, `plugin-package-contract`, `memory-host-sdk`. | Public API and package contracts | TypeScript packages with contract tests, export maps, and package-boundary checks. |
| `patches/` | 3 | Package-manager patches. | Build support | pnpm patching. |
| `qa/` | 82 | QA labs, credential broker, matrix fixtures. | Validation substrate | Synthetic channels/labs; package profiles and release checks reference QA plugins. |
| `scripts/` | 602 | Build, release, docs, install, validation, package helpers. | Tooling | Node scripts plus shell helpers; not user-facing substrate except packaging. |
| `security/` | 5 | Security docs/config. | Security posture | opengrep/semgrep references, secret detection baseline. |
| `skills/` | 73 | Bundled `SKILL.md` directories. | Agent skills | Managed/workspace/bundled skill model described in docs and README. |
| `src/` | 7,636 | Main TypeScript runtime. | Core substrate | Dense modular layout: gateway, agents, sessions, tasks, cron, hooks, plugins, MCP, context engine, config, channels, model catalog, memory host, TTS/voice. Very broad Vitest coverage. |
| `Swabble/` | 36 | Separate small app/package. | Side project / auxiliary UI | Not central to substrate assessment. |
| `test/` | 333 | Cross-cutting tests and fixtures. | Test suite | Supports package, channel, gateway, plugin, and CLI validation. |
| `test-fixtures/` | 1 | Fixture data. | Test support | Minimal. |
| `ui/` | 414 | Control UI frontend. | Dashboard / operator console | TypeScript frontend backed by gateway control APIs; CI has UI i18n/control-plane checks. |
| `vendor/` | 173 | Vendored third-party code/assets. | Bundled runtime support | Used to make package/runtime more self-contained. |

Primary root files: `openclaw.mjs` CLI entrypoint, `package.json`, `pnpm-workspace.yaml`, many `tsconfig.*.json` shards, `Dockerfile*`, `docker-compose.yml`, `README.md`, `CHANGELOG.md`, `AGENTS.md`, `SECURITY.md`. External dependencies of significance include Node 22.14+/24, pnpm, TypeScript, Vitest, croner, platform SDKs per plugin, Docker/SSH/OpenShell sandbox backends, and provider SDKs hidden behind bundled plugin runtime deps.

Test coverage shape: `src/` alone contains hundreds of `*.test.ts` files. Counts sampled by directory include `agents` 698 tests, `gateway` 284, `commands` 249, `plugins` 222, `auto-reply` 182, `cli` 154, `config` 130, `channels` 127, `plugin-sdk` 89, `cron` 88, `security` 44, `tasks` 16, `hooks` 22, `sessions` 7, and `mcp` 4. Extensions also carry local tests, including memory, speech, provider, and channel-specific suites.

## Named Subsystem Catalog

### Gateway

- Lives in `src/gateway`, `src/channels`, `src/pairing`, `src/security`, `src/routing`, plus channel plugins in `extensions/*`.
- Interface: `openclaw gateway`, JSON-RPC/control APIs, channel plugin registrations, gateway config, allowlists/pairing policies, operator scopes.
- Strategy: local-first gateway is the control plane for sessions, channels, tools, events, and web/control UI. Inbound channels route to per-agent sessions and workspaces; default DM policy is pairing/allowlist-oriented, and non-main sessions can be sandboxed.
- Carry-forward note: strongest lift candidate for multi-platform messaging and per-agent routing, but Arlo must wrap every route in tenant identity and authority binding before use.

### Plugin System

- Lives in `src/plugins`, `src/plugin-sdk`, `packages/plugin-sdk`, `packages/plugin-package-contract`, `extensions/*/openclaw.plugin.json`.
- Interface: `definePluginEntry`, plugin manifests, plugin slots, SDK registration calls for commands, tools, hooks, HTTP routes, channels, providers, speech, memory, session extensions, control UI descriptors, trusted policies.
- Strategy: large registry-based architecture with owner-scoped registration, plugin slots, validation, command registration, lifecycle hooks, and package contract validation. Plugins can supply both user-facing integrations and core substrate replacements such as context engines.
- Carry-forward note: mature but broad. It is powerful enough for Arlo's replaceable substrate thesis if Arlo treats it as untrusted supply and adds tenant-scoped load policy.

### Context Engine

- Lives in `src/context-engine`.
- Interface: config slot `plugins.slots.contextEngine`; `ContextEngine` methods include `bootstrap`, `maintain`, `ingest`, `ingestBatch`, `afterTurn`, `assemble`, and `compact`.
- Strategy: a process-global registry resolves a configured engine, validates the contract, and falls back to a default legacy engine if non-default resolution fails. It includes compatibility shims for older session-key/prompt parameter shapes.
- Carry-forward note: the interface is a clean substrate seam. It is thinner than Hermes's compression implementation, but better as an Arlo-controlled slot boundary.

### Sessions

- Lives in `src/sessions`, `src/gateway`, `src/agents`, and SDK namespace `packages/sdk/src/client.ts`.
- Interface: `sessions.*` RPC methods and tools such as `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`; session keys, session IDs, chat type, model/level overrides.
- Strategy: session identity is explicit and routable, with transcript events, lifecycle events, delivery policy, model overrides, and agent session-key parsing. Cron, exec, and channel events preserve or isolate session keys depending on execution context.
- Carry-forward note: useful routing and lineage primitives, but Arlo should not inherit OpenClaw's single-user assumptions without tenant-keyed storage and query filters.

### Task / Kanban Substrate

- Lives in `src/tasks`.
- Interface: background task records, task-flow registry, detached task runtime, audit, domain views, control runtime, SQLite stores.
- Strategy: OpenClaw tracks background tasks and task flows as runtime-owned records, especially for cron and detached execution. It is a ledger rather than a full collaborative Kanban surface.
- Carry-forward note: suitable for cron/task observability, weaker than Hermes for durable multi-worker work-item coordination.

### Cron / Scheduling

- Lives in `src/cron`, `docs/automation/cron-jobs.md`, CLI commands under `src/commands`.
- Interface: `openclaw cron add/list/show/edit/run/runs/remove/status`; config `cron.store`, `cron.enabled`, `maxConcurrentRuns`, session target choices; MCP/tool access through agent runs.
- Strategy: persistent job definitions at `~/.openclaw/cron/jobs.json`, split runtime state in `jobs-state.json`, croner expressions, isolated/current/main/custom session targets, run logs, retries, failure delivery, provider preflight, and task-ledger integration.
- Carry-forward note: very mature scheduler with careful session isolation. Arlo needs tenant-scoped job stores, per-tenant concurrency caps, and delivery authorization.

### Hooks and Webhooks

- Lives in `src/hooks`, `docs/automation/hooks`, `docs/automation/cron-jobs.md`, `extensions/webhooks`.
- Interface: plugin hooks, bundled hooks, `hooks.mappings`, `/hooks/wake`, `/hooks/agent`, Gmail Pub/Sub setup, hook tokens, allowed agent/session constraints.
- Strategy: both internal plugin hooks and external HTTP hooks exist. External hooks can transform payloads into wake or agent actions with template/code mappings, while internal hooks support message/tool/update/Gmail flows.
- Carry-forward note: useful event ingress layer, but direct hook-to-agent dispatch must be mediated by Arlo authority binding.

### MCP

- Lives in `src/mcp`.
- Interface: MCP servers for OpenClaw tools, plugin tools, channel tools, and stdio serving.
- Strategy: exposes OpenClaw capabilities outward as MCP surfaces and bridges channel/tool handlers. Tests cover channel server shutdown and OpenClaw/plugin tool serving.
- Carry-forward note: good outward MCP server substrate; less evidence of a full dynamic MCP client registry than Hermes.

### Memory

- Lives in `src/memory-host-sdk`, `packages/memory-host-sdk`, `extensions/memory-core`, `extensions/memory-lancedb`, `extensions/memory-wiki`, `extensions/voyage`, plus prompt hooks in plugin registry.
- Interface: memory CLI/plugin tools (`memory_search`, `memory_get`, reindex), memory runtime registration, memory embedding providers, prompt sections, corpus supplements, flush plan resolvers.
- Strategy: memory is a plugin-hosted subsystem with file-backed core memory, LanceDB-backed vector memory, remote/http/embedding support, batch uploads, query parsing, and prompt supplement hooks.
- Carry-forward note: technically deep, especially for indexed transcripts and embeddings, but tenant isolation must be designed at corpus, vector store, and citation levels.

### Skills

- Lives in `skills`, `src/wizard`, `src/plugins`, docs under `docs/tools/skills`.
- Interface: bundled, managed, and workspace skills discovered by onboarding/runtime; skill files are Markdown `SKILL.md`.
- Strategy: skills are file-based procedural instructions surfaced by onboarding and agent prompt/tooling. The plugin system can add skill packs, such as `open-prose` and `skill-workshop`.
- Carry-forward note: serviceable but less explicitly self-improving than Hermes's skills loop.

### Provider Routing / Models

- Lives in `src/model-catalog`, `src/agents`, `src/plugins/provider-*`, `extensions/*-provider`, `extensions/litellm`.
- Interface: provider plugins, model config, auth profiles, fallback chains, `plugins.entries.<provider>`, `openclaw models`, provider catalogs, LiteLLM extension.
- Strategy: many providers are implemented as bundled plugins using OpenClaw's provider runtime hooks. Auth profile rotation, failover, dynamic model prep, replay policy, usage snapshots, and transport normalization appear in SDK types.
- Carry-forward note: very broad provider matrix. LiteLLM-equivalent needs can likely be met by OpenClaw provider plugins plus the `litellm` adapter.

### Voice / Speech

- Lives in `src/realtime-transcription`, `src/realtime-voice`, `src/tts`, `extensions/speech-core`, `extensions/azure-speech`, `extensions/elevenlabs`, `extensions/microsoft`, `extensions/deepgram`, `extensions/gradium`, `extensions/inworld`, `extensions/tts-local-cli`, `extensions/talk-voice`, `extensions/voice-call`, companion apps in `apps`.
- Interface: speech provider plugin types, Talk config (`talk.provider`, `talk.providers.*`), voice commands, voice-call plugin, mobile/mac nodes.
- Strategy: speech is decomposed into runtime core packages, provider plugins, and device/node surfaces. Talk voice management is a plugin command with operator-admin scope checks for gateway callers.
- Carry-forward note: OpenClaw is the better voice substrate if Charlie needs modality routing beyond simple TTS.

### Dashboard / Operator Console

- Lives in `ui`, `src/web`, gateway control APIs, plugin control UI descriptor registry.
- Interface: control UI frontend, gateway discovery/status/control plane, plugin control UI descriptors.
- Strategy: web UI is packaged separately from runtime but tied to gateway status, control flows, task control-plane runtime contracts, and plugin descriptors.
- Carry-forward note: useful operator console foundation, but Arlo should supply tenant-aware operator roles and audit views.

### Native / Node Host / Canvas

- Lives in `apps`, `src/node-host`, `src/canvas-host`, `src/media*`, `src/image-generation`, `src/video-generation`, docs under `docs/platforms` and `docs/nodes`.
- Interface: macOS/iOS/Android nodes, node-host commands, Canvas/A2UI, camera/screen capture, media understanding/generation providers.
- Strategy: local companion apps expose device capabilities back to the gateway. Media providers are pluginized similarly to model providers.
- Carry-forward note: valuable for future rich-client tenants; not required for Charlie Phase 2 unless voice or operator UI becomes central.

## Integrations List

OpenClaw's advertised 50+ integrations are real in the sense that the repo contains concrete bundled extension directories for far more than 50. Most use a manifest plus TypeScript entrypoint registered through the plugin SDK; a few `*-core` directories are shared runtime packages and intentionally do not expose a standalone plugin manifest.

### Messaging / Channel Adapters

| Integration | Directory | Shape |
| --- | --- | --- |
| WhatsApp | `extensions/whatsapp` | Bundled channel plugin. |
| Telegram | `extensions/telegram` | Bundled channel plugin. |
| Slack | `extensions/slack` | Bundled channel plugin. |
| Discord | `extensions/discord` | Bundled channel plugin. |
| Google Chat | `extensions/googlechat` | Bundled channel plugin. |
| Signal | `extensions/signal` | Bundled channel plugin. |
| iMessage | `extensions/imessage` | Bundled channel plugin. |
| BlueBubbles | `extensions/bluebubbles` | Bundled channel plugin and webhook docs. |
| IRC | `extensions/irc` | Bundled channel plugin. |
| Microsoft Teams | `extensions/msteams` | Bundled channel plugin. |
| Matrix | `extensions/matrix` | Bundled channel plugin. |
| Feishu/Lark | `extensions/feishu` | Bundled channel plugin. |
| LINE | `extensions/line` | Bundled channel plugin. |
| Mattermost | `extensions/mattermost` | Bundled channel plugin. |
| Nextcloud Talk | `extensions/nextcloud-talk` | Bundled channel plugin. |
| Nostr | `extensions/nostr` | Bundled channel plugin. |
| Synology Chat | `extensions/synology-chat` | Bundled channel plugin. |
| Tlon/Urbit | `extensions/tlon` | Bundled channel plugin. |
| Twitch | `extensions/twitch` | Bundled channel plugin. |
| Zalo | `extensions/zalo` | Bundled channel plugin. |
| Zalo Personal | `extensions/zalouser` | Bundled personal-account plugin. |
| QQ Bot | `extensions/qqbot` | Bundled channel plugin. |
| Webhooks | `extensions/webhooks` | HTTP event bridge plugin. |
| QA synthetic channel | `extensions/qa-channel`, `extensions/qa-lab`, `extensions/qa-matrix` | Test/synthetic channel plugins. |

### Model / Provider Routing

| Integration | Directory | Shape |
| --- | --- | --- |
| OpenAI | `extensions/openai` | Provider plugin. |
| Anthropic | `extensions/anthropic` | Provider plugin. |
| Anthropic Vertex | `extensions/anthropic-vertex` | Provider plugin. |
| OpenRouter | `extensions/openrouter` | Provider plugin. |
| LiteLLM | `extensions/litellm` | Provider plugin with image-generation tests/catalog. |
| Ollama | `extensions/ollama` | Local provider plugin. |
| LM Studio | `extensions/lmstudio` | Local provider plugin. |
| vLLM | `extensions/vllm` | Self-hosted provider plugin. |
| SGLang | `extensions/sglang` | Self-hosted provider plugin. |
| Groq | `extensions/groq` | Provider/media-understanding plugin. |
| DeepSeek | `extensions/deepseek` | Provider plugin. |
| Qwen | `extensions/qwen` | Provider plugin. |
| Qianfan | `extensions/qianfan` | Provider plugin. |
| Moonshot/Kimi | `extensions/moonshot`, `extensions/kimi-coding` | Provider plugins. |
| MiniMax | `extensions/minimax` | Provider/OAuth plugin. |
| Mistral | `extensions/mistral` | Provider plugin. |
| Together | `extensions/together` | Provider plugin. |
| Fireworks | `extensions/fireworks` | Provider plugin. |
| DeepInfra | `extensions/deepinfra` | Provider plugin. |
| Cerebras | `extensions/cerebras` | Provider plugin. |
| Chutes | `extensions/chutes` | Provider plugin. |
| NVIDIA | `extensions/nvidia` | Provider plugin. |
| Hugging Face | `extensions/huggingface` | Provider plugin. |
| Amazon Bedrock | `extensions/amazon-bedrock`, `extensions/amazon-bedrock-mantle` | Provider plugins. |
| Microsoft Foundry | `extensions/microsoft-foundry` | Provider plugin. |
| Cloudflare AI Gateway | `extensions/cloudflare-ai-gateway` | Provider plugin. |
| Vercel AI Gateway | `extensions/vercel-ai-gateway` | Provider plugin. |
| Google | `extensions/google` | Google provider/tool plugin. |
| xAI | `extensions/xai` | Provider/tool plugin. |
| Venice | `extensions/venice` | Provider plugin. |
| Z.AI | `extensions/zai` | Provider plugin. |
| Volcengine, BytePlus, Tencent, Alibaba, Xiaomi, StepFun, Arcee, OpenCode, KiloCode, Copilot Proxy, GitHub Copilot, Synthetic | respective `extensions/*` directories | Provider plugins. |

### Tools, Search, Media, Memory, Voice, Diagnostics

| Integration | Directory | Shape |
| --- | --- | --- |
| Browser tools | `extensions/browser` | Tool plugin. |
| Brave, DuckDuckGo, Exa, Tavily, Perplexity, SearXNG | respective `extensions/*` directories | Web search/fetch provider plugins. |
| Firecrawl, Web Readability, Document Extract | `extensions/firecrawl`, `extensions/web-readability`, `extensions/document-extract` | Extraction/fetch plugins. |
| Memory core, LanceDB, Wiki | `extensions/memory-core`, `extensions/memory-lancedb`, `extensions/memory-wiki` | Memory search/vector/wiki plugins. |
| Voyage | `extensions/voyage` | Embedding provider plugin. |
| Speech core | `extensions/speech-core` | Shared speech runtime package, no manifest. |
| Azure Speech, ElevenLabs, Microsoft Speech, Deepgram, Gradium, Inworld, SenseAudio, local CLI TTS | respective `extensions/*` directories | Speech/TTS/STT/media understanding provider plugins. |
| Talk Voice, Voice Call | `extensions/talk-voice`, `extensions/voice-call` | Voice management and call plugins. |
| Image/video/media generation | `extensions/image-generation-core`, `extensions/video-generation-core`, `extensions/media-understanding-core`, `extensions/fal`, `extensions/comfy`, `extensions/runway`, `extensions/vydra` | Runtime packages and provider plugins. |
| File transfer, diffs, llm-task, lobster, open-prose, skill-workshop | respective `extensions/*` directories | Tool/skill/workflow plugins. |
| Bonjour, diagnostics-otel, diagnostics-prometheus | respective `extensions/*` directories | Discovery and observability plugins. |
| OpenShell sandbox | `extensions/openshell` | Sandbox backend plugin. |
| Migrations | `extensions/migrate-claude`, `extensions/migrate-hermes` | Migration provider plugins. |

## Architectural Notes Worth Carrying Forward

- OpenClaw is substrate-rich but product-shaped around a personal single-user assistant. The primitives are strong; the default trust model is not tenant-safe by itself.
- The plugin registry is the standout abstraction: it unifies tools, providers, channels, hooks, UI descriptors, memory, speech, and context slots under one load/registration model.
- The gateway and cron subsystems show the most operational hardening: pairing policies, scoped sessions, isolated cron runs, run logs, failure delivery, and cleanup of child runtimes.
- The task ledger exists, but it is not the same as Hermes's Kanban board. Use OpenClaw tasks for runtime observability; use Hermes or Arlo-authored logic for durable work-item collaboration.
- Integrations are broad and mostly concrete. The advertised messaging list is not marketing-only; it maps to real adapter directories.
