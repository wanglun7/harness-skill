# Mechanism 6: System prompt and instruction layering

Mechanism: System prompt and instruction layering

Status: complete

Claude source:
- `buildEffectiveSystemPrompt(...)` documents and implements prompt precedence: `overrideSystemPrompt` replaces everything; coordinator mode uses coordinator prompt plus optional append; main-thread agent prompt replaces default except proactive mode appends it after default; custom system prompt replaces default; otherwise default prompt is used; `appendSystemPrompt` is appended at the end except when override is set (`src/utils/systemPrompt.ts:28-122`).
- The default prompt comes from `getSystemPrompt(tools, model, additionalWorkingDirectories, mcpClients)`. In simple mode it returns a minimal single-block prompt with CWD/date; otherwise it builds static and dynamic sections (`src/constants/prompts.ts:444-461`).
- `getSystemPrompt(...)` has a proactive path that returns autonomous-agent identity, system reminders, memory, env, language, MCP instructions when not delta-backed, scratchpad, function-result-clearing, summarize-tool-results, and proactive sections (`src/constants/prompts.ts:466-489`).
- In the normal path, dynamic sections are named through `systemPromptSection(...)` or `DANGEROUS_uncachedSystemPromptSection(...)`, including session guidance, memory, model override, env info, language, output style, MCP instructions, scratchpad, function-result-clearing, summarize-tool-results, token budget, and brief sections (`src/constants/prompts.ts:491-559`).
- The returned default prompt places static sections first, then `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` when global cache scope is enabled, then resolved dynamic sections (`src/constants/prompts.ts:560-576`).
- `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` is defined as a marker separating static cross-org-cacheable prompt content from dynamic user/session-specific prompt content; comments explicitly tie it to `splitSysPromptPrefix` and `buildSystemPromptBlocks` (`src/constants/prompts.ts:105-115`).
- `systemPromptSection(...)` memoizes named system prompt sections until `/clear` or `/compact`; `DANGEROUS_uncachedSystemPromptSection(...)` recomputes volatile sections every turn; `clearSystemPromptSections()` clears prompt-section and beta-header state (`src/constants/systemPromptSections.ts:16-67`).
- Interactive REPL turns call `getSystemPrompt(...)`, load user/system context, then call `buildEffectiveSystemPrompt(...)`; the rendered system prompt is stored on `toolUseContext.renderedSystemPrompt` (`src/screens/REPL.tsx:2534-2544`, `src/screens/REPL.tsx:2770-2788`).
- `QueryEngine` headless setup calls `fetchSystemPromptParts(...)`, then builds a system prompt from either custom prompt or default prompt, optional memory mechanics prompt, and optional append prompt (`src/QueryEngine.ts:284-325`).
- `query.ts` appends `systemContext` to the already-built system prompt with `appendSystemContext(...)` before model calls (`src/query.ts:449-451`).
- `appendSystemContext(...)` adds key/value system context as an additional prompt block after the existing system prompt array (`src/utils/api.ts:437-447`).
- Before Anthropic dispatch, `claude.ts` prepends attribution and CLI/system-prefix blocks, appends advisor/chrome tool-search instruction blocks when active, and filters empty blocks (`src/services/api/claude.ts:1347-1369`).
- `getCLISyspromptPrefix(...)` selects one of three prefix strings based on provider, non-interactive mode, and whether append-system-prompt is present; `getAttributionHeader(...)` optionally emits an Anthropic billing/entrypoint/workload header string for the API body (`src/constants/system.ts:10-46`, `src/constants/system.ts:48-95`).
- `splitSysPromptPrefix(...)` recognizes attribution and CLI prefix blocks by content, optionally splits static/dynamic prompt content at the boundary marker, and assigns cache scopes; fallback/default mode joins all remaining prompt blocks into a single org-cacheable block (`src/utils/api.ts:296-435`).
- `buildSystemPromptBlocks(...)` converts split prompt sections into Anthropic `TextBlockParam[]` and attaches cache-control metadata where enabled and cache scope is not null (`src/services/api/claude.ts:3213-3237`).

Codex source:
- `BaseInstructions` is the base instruction object for a thread and corresponds to the Responses API `instructions` field; its default text is included from `prompts/base_instructions/default.md` (`codex-rs/protocol/src/models.rs:1196-1211`).
- `ModelInfo` carries `base_instructions` and optional `model_messages` metadata; `get_model_instructions(personality)` uses `model_messages.instructions_template` when present, replacing `{{ personality }}` with a personality-specific message, otherwise falls back to `base_instructions` and logs when a personality was requested without model messages (`codex-rs/protocol/src/openai_models.rs:350-470`).
- `ModelMessages` and `ModelInstructionsVariables` define the strongly typed template/variable layer for personality-aware model instructions (`codex-rs/protocol/src/openai_models.rs:474-530`).
- Config can supply `base_instructions` and `developer_instructions`; the base-instructions override can come from a model instructions file or inline `instructions`, and developer instructions come from config developer instructions (`codex-rs/core/src/config/mod.rs:669-673`, `codex-rs/core/src/config/mod.rs:3331-3345`, `codex-rs/core/src/config/mod.rs:3551-3559`).
- `SessionConfiguration` stores provider/collaboration mode, optional developer instructions, loaded `AGENTS.md`, personality, and base instructions as separate fields (`codex-rs/core/src/session/session.rs:50-69`).
- New thread creation persists session-level `base_instructions` in `SessionMeta` (`codex-rs/core/src/session/session.rs:528-546`; `codex-rs/protocol/src/protocol.rs:2863-2898`).
- `TurnContext` carries `developer_instructions`, rendered user instructions, personality, current date/timezone, model, permissions, and other turn state; construction copies developer instructions from session configuration and renders loaded `AGENTS.md` into `user_instructions` (`codex-rs/core/src/session/turn_context.rs:107-140`, `codex-rs/core/src/session/turn_context.rs:582-615`).
- `TurnContext::to_turn_context_item()` persists turn metadata but not the base instruction text itself; the protocol type includes model/personality/collaboration mode and related context (`codex-rs/core/src/session/turn_context.rs:397-420`, `codex-rs/protocol/src/protocol.rs:2988-3026`).
- `Session::get_base_instructions()` returns the current session configuration's base instructions as `BaseInstructions` (`codex-rs/core/src/session/mod.rs:1196-1201`).
- Before sampling, the turn loop records context updates with `record_context_updates_and_set_reference_context_item(...)`, then later builds prompt input from history (`codex-rs/core/src/session/turn.rs:150-170`, `codex-rs/core/src/session/turn.rs:221-245`).
- `build_initial_context(...)` builds ordered developer and contextual-user sections: model-switch instructions, permissions, configured developer instructions, collaboration-mode instructions, realtime updates, personality instructions when not already baked into base instructions, apps, skills, plugin/extension fragments, rendered user instructions, token budget, and environment context (`codex-rs/core/src/session/mod.rs:2845-3101`).
- `build_developer_update_item(...)` creates a `ResponseItem::Message` with role `developer`; `build_contextual_user_message(...)` creates role `user`; both wrap sections as `ContentItem::InputText` blocks (`codex-rs/core/src/context_manager/updates.rs:183-208`).
- `record_context_updates_and_set_reference_context_item(...)` records full initial context or settings diffs into conversation history, persists a `TurnContextItem` per real user turn, and updates the in-memory diff baseline (`codex-rs/core/src/session/mod.rs:3180-3216`).
- `build_prompt(...)` packages the model-bound input with tools, parallel-tool-call flag, `base_instructions`, personality, and output schema (`codex-rs/core/src/session/turn.rs:1016-1034`).
- `ModelClient::build_responses_request(...)` copies `prompt.base_instructions.text` into `ResponsesApiRequest.instructions`; developer/contextual-user instruction items remain in `input` via prompt history (`codex-rs/core/src/client.rs:780-830`).
- Config-lock persistence writes resolved base instructions as `instructions` and developer instructions separately (`codex-rs/core/src/session/config_lock.rs:107-114`).

Runtime trace evidence:
- Claude: `getSystemPrompt(...)` builds default prompt sections -> `buildEffectiveSystemPrompt(...)` applies override/agent/custom/default/append precedence -> `query.ts` appends system context -> `claude.ts` prepends attribution and CLI prefix plus source-specific advisor/chrome additions -> `splitSysPromptPrefix(...)` and `buildSystemPromptBlocks(...)` convert the ordered string array into Anthropic system text blocks. Evidence: `src/constants/prompts.ts:444-576`, `src/utils/systemPrompt.ts:28-122`, `src/query.ts:449-451`, `src/services/api/claude.ts:1347-1380`, `src/utils/api.ts:296-447`, `src/services/api/claude.ts:3213-3237`.
- Codex: configuration/model metadata resolves base instructions -> session/turn context carries developer/user instruction fields -> pre-sampling context injection records developer/contextual-user `ResponseItem`s into history -> `build_prompt(...)` carries `BaseInstructions` separately -> request construction copies `BaseInstructions.text` into Responses `instructions`. Evidence: `codex-rs/protocol/src/openai_models.rs:452-470`, `codex-rs/core/src/config/mod.rs:3331-3345`, `codex-rs/core/src/session/turn_context.rs:582-615`, `codex-rs/core/src/session/mod.rs:2845-3101`, `codex-rs/core/src/session/mod.rs:3180-3216`, `codex-rs/core/src/session/turn.rs:1016-1034`, `codex-rs/core/src/client.rs:780-830`.

What the model sees:
- Claude: Anthropic sees a `system` array of text blocks whose content order is attribution header, CLI/system prefix, effective system prompt array, appended system context, and any source-specific advisor/chrome instructions added at send time. Cache-control annotations may be attached to blocks but are not prompt text (`src/services/api/claude.ts:1347-1380`, `src/services/api/claude.ts:3213-3237`).
- Codex: OpenAI Responses sees base model/session instructions in the request's `instructions` field. Additional developer/contextual-user instruction layers are visible as `ResponseItem::Message` entries in `input` with roles `developer` and `user` (`codex-rs/core/src/client.rs:780-830`, `codex-rs/core/src/context_manager/updates.rs:183-208`).

What the runtime does:
- Claude: constructs a string-array prompt, applies precedence rules before the turn, appends runtime system context, then reshapes the array into cache-scoped Anthropic system blocks immediately before dispatch.
- Codex: keeps base instructions as a session-level field separate from history, injects developer/contextual-user instruction messages into conversation history before sampling, and uses a durable `TurnContextItem` baseline to avoid re-emitting unchanged context every turn.

What gets persisted:
- Claude: this card found rendered prompt storage on `toolUseContext.renderedSystemPrompt` and prompt/cache debug surfaces, but not durable transcript persistence of the system prompt as a normal message in the cited path (`src/screens/REPL.tsx:2536-2544`, `src/screens/REPL.tsx:2781-2788`).
- Codex: `SessionMeta` persists base instructions for the thread, `record_context_updates_and_set_reference_context_item(...)` records model-visible context instruction items into conversation history, and config-lock persistence can store resolved base/developer instructions (`codex-rs/core/src/session/session.rs:528-546`, `codex-rs/core/src/session/mod.rs:3180-3216`, `codex-rs/core/src/session/config_lock.rs:107-114`).

What the user/TUI sees:
- Claude: users can affect layers via custom system prompt, append system prompt, main-thread agent prompt, coordinator/proactive modes, and default prompt context; ant-only context analysis can expose system prompt sections (`src/utils/systemPrompt.ts:28-122`, `src/utils/analyzeContext.ts:936-955`, `src/utils/analyzeContext.ts:1350-1359`).
- Codex: normal TUI display sees protocol events and not the raw `instructions` field; prompt-debug can rebuild and return provider-bound prompt input, including context instruction items, through the same runtime path (`codex-rs/core/src/prompt_debug.rs:74-101`).

What debug/eval surfaces exist:
- Claude: `--dump-system-prompt` renders `getSystemPrompt([], model)` and exits for prompt sensitivity evals; `logAPIPrefix(...)` logs prompt-prefix metadata; prompt cache boundary events are logged by `splitSysPromptPrefix(...)`; context analysis can report per-section prompt details in ant builds (`src/entrypoints/cli.tsx:50-69`, `src/utils/api.ts:321-435`, `src/services/api/claude.ts:1371-1379`, `src/utils/analyzeContext.ts:936-955`).
- Codex: tests verify `get_model_instructions(...)` template/personality fallback behavior and session `get_base_instructions()` returns model base instructions; prompt-debug rebuilds prompt input through context injection, and rollout/config persistence records base/developer instruction state (`codex-rs/protocol/src/openai_models.rs:755-833`, `codex-rs/core/src/session/tests.rs:1153-1207`, `codex-rs/core/src/prompt_debug.rs:74-101`, `codex-rs/core/src/session/config_lock.rs:107-114`).

Shared invariant:
- Both references separate a stable/base instruction layer from turn/session-specific instruction layers and assemble them at a controlled pre-dispatch boundary. The exact transport shape differs: Claude uses Anthropic `system` blocks; Codex uses Responses `instructions` plus developer/user input items.

Claude-specific design:
- Claude's effective prompt precedence is explicit and source-specific: override replaces all; coordinator replaces default; agent usually replaces default; custom prompt replaces default; append goes last.
- Claude adds attribution and CLI/system prefix at API-send time, not inside `getSystemPrompt(...)`.
- Claude uses a dynamic boundary marker and named prompt-section cache to control prompt caching and recomputation.

Codex-specific design:
- Codex stores base instructions as session/thread state and passes them through the Responses `instructions` field.
- Codex injects developer instructions, collaboration/personality/app/skill/plugin/context fragments, and rendered `AGENTS.md` as `ResponseItem` history messages, not by concatenating them into `BaseInstructions.text`.
- Codex persists turn-context baselines separately so future turns can emit diffs rather than re-inject all instruction/context layers.

Gaps or unknowns:
- This card cites where Claude includes memory/MCP/output-style/system-context sections but does not trace how those source texts are discovered or rendered in detail; dynamic context assembly is mechanism 7, skills/plugins/custom instructions are mechanism 22, and memory lifecycle is mechanism 21.
- This card cites Codex skills/apps/plugins as instruction layers only at the `build_initial_context(...)` insertion points; their registries and loading behavior belong to mechanisms 10 and 22.
- Claude durable persistence of system prompt text was not established in the cited normal conversation path; only debug/rendered-context surfaces were found here.

Distilled harness rule:
- Make instruction layering explicit and inspectable: define the base instruction channel, define ordered override/append/context channels, then prove exactly which provider field or prompt-history item each layer enters.

Anti-invention rule:
- Do not collapse all instruction text into one generic "system prompt" unless the source does. Claude sends ordered Anthropic system blocks; Codex sends base instructions in `instructions` and supplemental instruction layers as developer/user `ResponseItem`s. Preserve that distinction.

Verification pattern:
- Find the base instruction source and default text.
- Find user/custom override and append inputs.
- Find agent/coordinator/personality/source-specific instruction layers.
- Find the function that orders the layers.
- Find whether dynamic/context sections are cached, recomputed, or diffed.
- Find how the ordered layers map to provider fields versus prompt-history messages.
- Find persistence and debug surfaces that can prove the rendered instruction shape.

Skill text candidate:
- For instruction layering, trace the prompt from configuration/model metadata to the final provider-visible fields. Record precedence in source order, then classify each layer as base instruction, override, append, developer message, contextual user message, or transport/cache metadata. Never translate Claude's `system` block architecture into Codex's Responses `instructions` path, or vice versa, without source evidence.
