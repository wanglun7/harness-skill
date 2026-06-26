# Mechanism 10: Tool registry and model-visible schema projection

Mechanism: Tool registry and model-visible schema projection

Status: complete

Claude source:
- `src/Tool.ts:158-179` makes `ToolUseContext.options.tools` the per-turn tool set and includes an optional `refreshTools()` callback for mid-query tool refresh.
- `src/Tool.ts:358-360` looks tools up by model/tool name or alias through `findToolByName()` and `toolMatchesName()`.
- `src/Tool.ts:362-405` defines the `Tool` contract with runtime `call()`, async `description()`, Zod `inputSchema`, optional JSON `inputJSONSchema`, `isEnabled()`, `isReadOnly()`, and concurrency/read-only controls.
- `src/Tool.ts:430-472` adds model/exposure-related metadata: MCP flags, `shouldDefer`, `alwaysLoad`, original MCP names, primary `name`, result-size limits, and optional strict mode.
- `src/Tool.ts:518-524` defines the model-facing prompt text generator `prompt()` and user-facing name separately.
- `src/Tool.ts:697-701` defines `Tools` as a readonly collection to track where tool sets are assembled, passed, and filtered.
- `src/Tool.ts:703-792` defines `buildTool()`, which fills default methods for tool definitions and keeps complete `Tool` objects at runtime.
- `src/tools.ts:173-183` exposes the default preset names by filtering `getAllBaseTools()` through `isEnabled()`.
- `src/tools.ts:185-250` defines `getAllBaseTools()` as the exhaustive built-in tool source of truth for the current environment, including feature/env-gated tools and the optimistic `ToolSearchTool`.
- `src/tools.ts:253-269` filters tools before model exposure when permission context contains blanket deny rules, including MCP server-prefix deny rules.
- `src/tools.ts:271-327` defines `getTools()`: simple-mode selection, REPL primitive hiding, special-tool filtering, permission deny filtering, and final `isEnabled()` filtering.
- `src/tools.ts:329-367` defines `assembleToolPool()` as the single built-in-plus-MCP assembly path, filters MCP tools by deny rules, sorts built-ins and MCP tools separately for prompt-cache stability, and deduplicates by name with built-ins winning.
- `src/tools.ts:369-388` defines `getMergedTools()` for contexts that need built-in and MCP tools without the cache-stable/dedup assembly behavior.
- `src/query.ts:546-568` stores current messages in `toolUseContext` and initializes `StreamingToolExecutor` with `toolUseContext.options.tools`.
- `src/query.ts:659-708` passes `toolUseContext.options.tools` and request options, including model, agent definitions, MCP tools, pending MCP state, and `getToolPermissionContext()`, into `deps.callModel()`.
- `src/utils/api.ts:68-78` extends Anthropic beta tool schema with `strict`, `defer_loading`, `cache_control`, and `eager_input_streaming`.
- `src/utils/api.ts:119-178` converts a `Tool` to an API schema with `name`, `description` from `tool.prompt()`, and `input_schema` from `inputJSONSchema` or Zod-to-JSON-Schema conversion, caching the session-stable base schema.
- `src/utils/api.ts:180-208` conditionally adds strict mode and fine-grained tool streaming to the cached base schema.
- `src/utils/api.ts:211-260` overlays per-request `defer_loading` and `cache_control` without mutating the cached base schema and strips beta-only fields when experimental betas are disabled.
- `src/utils/toolSearch.ts:155-198` defines tool-search modes: `tst`, `tst-auto`, and `standard`, with the experimental-beta kill switch forcing standard mode.
- `src/utils/toolSearch.ts:323-365` checks ToolSearchTool availability and computes deferred tool description size from name, prompt text, and input schema.
- `src/utils/toolSearch.ts:367-456` defines the definitive request-time `isToolSearchEnabled()` check using model support, ToolSearchTool availability, configured mode, and optional auto threshold.
- `src/services/api/claude.ts:1118-1177` decides whether tool search is active for a request, computes deferred tool names, filters visible tools to non-deferred plus discovered deferred tools plus ToolSearchTool, removes ToolSearchTool when disabled, and adds the required beta header when enabled.
- `src/services/api/claude.ts:1207-1246` computes per-tool `willDefer()` and builds `toolSchemas` with `toolToAPISchema()`, passing the full tool list to prompt generation while filtering which schemas are sent.
- `src/services/api/claude.ts:1259-1288` normalizes messages against the filtered tool set and strips tool-search-only fields when the selected model does not support tool search.
- `src/services/api/claude.ts:1458-1470` excludes `defer_loading` tools from prompt-cache-break detection because the API strips them from the prompt.

Codex source:
- `codex-rs/tools/src/tool_definition.rs:4-13` defines `ToolDefinition` metadata: name, description, input schema, optional output schema, and `defer_loading`.
- `codex-rs/tools/src/tool_definition.rs:21-25` marks a tool definition deferred and drops its output schema.
- `codex-rs/tools/src/responses_api.rs:25-38` defines `ResponsesApiTool` with `name`, `description`, `strict`, optional `defer_loading`, `parameters`, and skipped `output_schema`.
- `codex-rs/tools/src/responses_api.rs:40-67` defines `LoadableToolSpec`, `ResponsesApiNamespace`, and namespace function tools.
- `codex-rs/tools/src/responses_api.rs:77-136` coalesces loadable namespace specs, converts MCP/dynamic tool definitions to Responses API tools, and maps `defer_loading` to `Some(true)`.
- `codex-rs/tools/src/tool_spec.rs:13-51` defines every model-visible Responses API tool shape: function, namespace, tool search, image generation, web search, and freeform custom tools.
- `codex-rs/tools/src/tool_spec.rs:53-63` gives every `ToolSpec` a request-facing name.
- `codex-rs/tools/src/tool_spec.rs:75-89` serializes `ToolSpec` values into Responses API-compatible JSON values.
- `codex-rs/tools/src/tool_executor.rs:13-42` defines `ToolExposure`: direct, deferred, direct-model-only, and hidden.
- `codex-rs/tools/src/tool_executor.rs:44-68` defines the shared `ToolExecutor` contract tying `tool_name()`, `spec()`, exposure, optional search metadata, parallel support, and handler execution together.
- `codex-rs/tools/src/tool_search.rs:21-65` derives `ToolSearchInfo` from function/namespace `ToolSpec`s by setting `defer_loading: true`, dropping output schemas, and rejecting non-loadable hosted/freeform/search specs.
- `codex-rs/core/src/mcp_tool_exposure.rs:14-54` splits MCP tools into direct or deferred sets based on model-visible MCP filtering, connector/app policy, search-tool support, explicit always-defer feature, and the direct-exposure threshold.
- `codex-rs/core/src/tools/registry.rs:43-144` defines `CoreToolRuntime` as the core runtime contract layered on `ToolExecutor`, including payload kind matching, hook payloads, telemetry tags, updated hook input, and optional streamed argument diff consumers.
- `codex-rs/core/src/tools/registry.rs:237-279` wraps an existing tool runtime with exposure overrides.
- `codex-rs/core/src/tools/registry.rs:321-340` stores runtime handlers by `ToolName` in `ToolRegistry` and rejects duplicate registrations.
- `codex-rs/core/src/tools/router.rs:35-78` keeps runtime registry and `model_visible_specs` together in `ToolRouter`, while exposing only cloned model-visible specs to prompt construction.
- `codex-rs/core/src/tools/router.rs:40-46` defines router inputs: direct MCP tools, deferred MCP tools, tool suggestions, extension executors, and dynamic tools.
- `codex-rs/core/src/tools/spec_plan.rs:100-140` defines planned runtime and hosted-spec buckets plus helper methods for direct, overridden, hidden, and hosted tools.
- `codex-rs/core/src/tools/spec_plan.rs:155-197` builds the `ToolRouter` by adding tool sources, appending tool-search executor, prepending code-mode executors, and then building visible specs plus registry.
- `codex-rs/core/src/tools/spec_plan.rs:199-238` deduplicates runtimes by tool name, emits specs only for direct exposures not hidden by code-mode-only, appends hosted specs, merges namespaces, filters namespaces when provider capability is absent, and builds the runtime registry from all runtimes.
- `codex-rs/core/src/tools/spec_plan.rs:240-257` augments eligible nested tool specs for code mode.
- `codex-rs/core/src/tools/spec_plan.rs:259-293` adds hosted web-search and image-generation specs when provider/model/feature conditions allow.
- `codex-rs/core/src/tools/spec_plan.rs:563-574` adds source families in order: shell, MCP resources, core utilities, collaboration, MCP runtime tools, extension tools, dynamic tools, and hosted model tools.
- `codex-rs/core/src/tools/spec_plan.rs:585-623` selects shell/unified-exec tooling from environment mode, feature gates, and model shell type, keeping legacy shell dispatch hidden when unified exec is model-visible.
- `codex-rs/core/src/tools/spec_plan.rs:636-713` adds MCP resource tools and core utility tools, including direct-model-only tools such as request-user-input and new-context-window.
- `codex-rs/core/src/tools/spec_plan.rs:715-801` adds collaboration and agent-job tools, with v1 multi-agent tools deferred when search and namespace support are available.
- `codex-rs/core/src/tools/spec_plan.rs:803-827` adds direct and deferred MCP runtime handlers.
- `codex-rs/core/src/tools/spec_plan.rs:829-859` adapts dynamic function and namespace tools into runtime handlers.
- `codex-rs/core/src/tools/spec_plan.rs:861-955` adapts extension executors, reserves conflicting names, skips unavailable standalone web/image tools, and warns on duplicate registrations.
- `codex-rs/core/src/tools/spec_plan.rs:871-893` creates a `tool_search` runtime only when search and namespace support are enabled and at least one deferred runtime has search metadata.
- `codex-rs/core/src/tools/spec_plan.rs:432-491` builds code-mode executors by collecting nested tool specs, separating deferred-tool guidance, and creating code-mode execute/wait tools.
- `codex-rs/core/src/tools/spec_plan.rs:494-537` merges namespace specs by name, appends namespace functions, sorts namespace functions, and fills empty namespace descriptions.
- `codex-rs/core/src/tools/handlers/tool_search_spec.rs:7-61` creates the client-executed `tool_search` model-visible spec with source descriptions and a JSON schema for `query` and `limit`.
- `codex-rs/core/src/tools/handlers/tool_search.rs:24-87` caches/builds `ToolSearchHandler` from `ToolSearchInfo`, creates its spec, and builds a BM25 search index over deferred tool search text.
- `codex-rs/core/src/session/turn_context.rs:147-153` carries per-turn `dynamic_tools` in `TurnContext`.
- `codex-rs/core/src/session/turn.rs:1258-1278` builds MCP exposure, then creates a `ToolRouter` from turn context, MCP direct/deferred tools, tool suggestions, extension executors, and dynamic tools.
- `codex-rs/core/src/session/turn.rs:1016-1034` builds the model prompt from input and `router.model_visible_specs()`, plus `parallel_tool_calls`, base instructions, personality, and optional final output schema.
- `codex-rs/core/src/client.rs:780-830` serializes `prompt.tools` with `create_tools_json_for_responses_api()` and sends them as the `tools` field with `tool_choice: "auto"` and provider/model-gated `parallel_tool_calls`.

Runtime trace evidence:
- Claude trace: app/session state supplies `Tools` to `ToolUseContext`; `query()` passes that set to `callModel()`; the Claude API layer filters for tool search, projects each remaining `Tool` through `toolToAPISchema()`, normalizes messages against the filtered tool set, and sends the resulting `toolSchemas` to the Anthropic request.
- Claude deferred trace: `ToolSearchTool` can be present optimistically in base tools; request-time `isToolSearchEnabled()` decides support; deferred tools are omitted until discovered from message history, while `ToolSearchTool` remains visible so the model can request references.
- Codex trace: `build_tool_router()` builds planned runtime handlers and hosted specs from `TurnContext` and external sources; `ToolRouter` stores both the runtime registry and visible specs; `build_prompt()` copies visible specs into `Prompt.tools`; the client serializes those specs into the Responses API `tools` array.
- Codex deferred trace: MCP exposure and tool runtimes can be marked `Deferred`; `append_tool_search_executor()` creates a `tool_search` tool only when provider/model support and deferred search metadata exist; tool-search output later returns loadable specs, but that execution/result path belongs to later mechanisms.

What the model sees:
- Claude: the model sees an Anthropic tool array with tool `name`, `description`, `input_schema`, and optional beta fields such as `strict`, `eager_input_streaming`, `defer_loading`, and `cache_control`, depending on provider/model/feature/request state. ToolSearch-enabled requests may show only non-deferred tools, discovered deferred tools, and ToolSearchTool.
- Codex: the model sees a Responses API `tools` array serialized from `ToolSpec` values: functions, namespaces, tool-search, hosted web/image generation, and freeform custom tools. Direct and direct-model-only tools can be visible immediately; deferred tools are discoverable through `tool_search` when available.

What the runtime does:
- Claude: keeps a single `Tool` object as the owner of schema, prompt text, validation, permissions, execution, and display hooks; request code projects only the model-visible subset into API schema and can refresh the active tool set after tool execution.
- Codex: keeps tool spec and runtime handler tied through `ToolExecutor`, then separates the runtime registry from model-visible specs in `ToolRouter`; hidden/deferred/direct exposure controls whether a registered handler appears in the initial tool list.

What gets persisted:
- Claude: this mechanism does not establish durable persistence of tool registries. Deferred-tool discovery can be inferred from message-history tool-reference blocks during request filtering, and prompt-cache-break detection hashes the sent non-deferred tool schemas.
- Codex: this mechanism does not persist registries as standalone state. Tool specs appear in model request payloads; dynamic tools live in `TurnContext`; tool-search outputs and tool calls are represented as response/history items in adjacent paths covered by later mechanisms.

What the user/TUI sees:
- Claude: this mechanism is mostly model/API-facing. User-facing names and colors live on the `Tool` contract, but actual rendering belongs to later result/TUI mechanisms.
- Codex: this mechanism is mostly model/API-facing. Tool suggestions can add request-plugin-install/list tools to the visible set, but the display of tool calls and suggestions belongs to later mechanisms.

What debug/eval surfaces exist:
- Claude: debug and analytics surfaces include tool-search mode decisions, auto-threshold calculations, dynamic deferred-tool counts, context-size logging for MCP/non-MCP tool schema tokens, beta-field stripping logs, and prompt-cache-break detection excluding deferred tools.
- Codex: tests cover tool spec serialization, omission/preservation of `defer_loading`, tool-search wire shape, MCP direct/deferred exposure, namespace merging, dynamic tools, extension tools, and code-mode visibility. Tracing instruments router/spec planning functions.

Shared invariant:
- Model-visible tool schema is a projection of the runtime tool universe, not the runtime universe itself. The projection must be filtered by environment, permissions/policy, provider/model capability, feature flags, and deferred-discovery state before being attached to a model request.

Claude-specific design:
- Claude uses one `Tool` contract that owns both runtime behavior and model-facing schema/prompt text; `toolToAPISchema()` converts Zod or JSON schemas to Anthropic tool definitions and applies request overlays. ToolSearch uses `defer_loading` and message-history discovery to reduce the sent schema set.

Codex-specific design:
- Codex uses `ToolSpec` and `ToolExecutor` as explicit spec/runtime contracts, then `ToolRouter` holds both `model_visible_specs` and a runtime `ToolRegistry`. Exposure states (`Direct`, `Deferred`, `DirectModelOnly`, `Hidden`) are first-class, and namespace/code-mode/hosted tool projection is handled in a spec planning pass.

Gaps or unknowns:
- Claude `ToolSearchTool` execution and returned `tool_reference` blocks are intentionally not decomposed here; this card only covers how tool search changes schema projection.
- Codex tool-search execution and loading of returned specs are intentionally not decomposed here; this card only covers the visible `tool_search` spec and deferred metadata.
- Individual file, shell, web, task, subagent, and MCP tool schemas are not enumerated here because later mechanism cards cover those tool families.

Distilled harness rule:
- Treat tool exposure as a two-step process: first assemble the runtime-capable tool universe, then project only the provider/model-safe, policy-allowed, mode-appropriate, and discovery-appropriate schemas into the model request.

Anti-invention rule:
- Do not infer tool availability from a source tree list alone. Cite the actual active-tool assembly path and the final provider payload projection. If a tool is registered but hidden/deferred, mark it as registered-but-not-initially-model-visible instead of calling it available to the model.

Verification pattern:
- For Claude, trace `getAllBaseTools()` -> `getTools()` or `assembleToolPool()` -> `ToolUseContext.options.tools` -> `query()` `callModel()` args -> `isToolSearchEnabled()`/filtered tools -> `toolToAPISchema()` -> request tool schemas.
- For Codex, trace source inputs to `ToolRouterParams` -> `build_tool_router()` -> `add_tool_sources()` -> exposure filtering in `build_model_visible_specs_and_registry()` -> `ToolRouter::model_visible_specs()` -> `build_prompt()` -> `create_tools_json_for_responses_api()` -> `ResponsesApiRequest.tools`.

Skill text candidate:
- When analyzing tool registry and schema projection:
  - identify the runtime tool universe and the concrete source of each tool family;
  - identify filters before model exposure: permission denies, environment mode, feature gates, provider/model capabilities, simple/code modes, connector/app policy, and duplicate-name policy;
  - identify whether each tool is direct, hidden, deferred, direct-model-only, hosted, namespaced, dynamic, extension-provided, or MCP-provided;
  - cite the exact function that converts internal schema into provider wire shape;
  - cite the exact request-construction point where the projected schema list is attached to the model call;
  - keep runtime dispatch, permission approval, result serialization, and TUI rendering out of this mechanism unless the source uses them to decide model-visible schema.
