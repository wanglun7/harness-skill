# Mechanism 5: Message normalization and provider payload construction

Mechanism: Message normalization and provider payload construction

Status: complete

Claude source:
- `query.ts` appends runtime system context to the supplied system prompt, prepends user context to `messagesForQuery`, and passes both into `deps.callModel(...)`; the fetch override can be `createDumpPromptsFetch(...)` for prompt capture (`src/query.ts:449-451`, `src/query.ts:582-590`, `src/query.ts:659-708`).
- `normalizeMessagesForAPI(messages, tools)` is the main transcript-to-provider-message boundary. It accepts internal `Message[]` and returns only API user/assistant messages (`src/utils/messages.ts:1989-1992`).
- Before per-message conversion, Claude reorders attachments and removes virtual user/assistant messages because they are display-only and must not reach the API (`src/utils/messages.ts:1996-2001`).
- Synthetic API-error messages are scanned to target preceding meta user messages for document/image/request-too-large stripping; progress messages, ordinary system messages, and synthetic API errors are then filtered out (`src/utils/messages.ts:2003-2074`).
- Local-command system messages are converted into user messages so the model can reference prior command output (`src/utils/messages.ts:2077-2092`).
- User messages strip or filter `tool_reference` blocks depending on tool-search availability and current available tool names, remove targeted media blocks after size errors, optionally add a tool-reference turn-boundary sibling, and merge consecutive user messages (`src/utils/messages.ts:2094-2199`).
- Assistant messages normalize `tool_use` inputs with `normalizeToolInputForAPI`, canonicalize tool names, preserve tool-search fields only when tool search is enabled, otherwise rebuild standard `tool_use` blocks with `{type,id,name,input}` (`src/utils/messages.ts:2201-2244`).
- Assistant messages with the same provider message id are merged across interleaved tool-result messages, while different assistant ids are left separate (`src/utils/messages.ts:2246-2267`).
- Attachments are normalized into user messages, optionally system-reminder wrapped, and merged into the previous user message when possible (`src/utils/messages.ts:2269-2290`).
- Post-passes relocate tool-reference siblings, filter orphaned thinking-only messages, strip trailing thinking from the last assistant, remove whitespace-only assistant messages, ensure non-empty assistant content, optionally merge/smoosh system-reminder siblings, sanitize error tool-result media, append history snip id tags, and validate images before sending (`src/utils/messages.ts:2295-2369`).
- User-message merge helpers normalize string content to text blocks, hoist `tool_result` blocks to the front to avoid API ordering errors, and insert a newline when adjacent text blocks are joined (`src/utils/messages.ts:2372-2387`, `src/utils/messages.ts:2411-2449`, `src/utils/messages.ts:2466-2515`).
- `claude.ts` builds tool schemas before normalization, logs pre-normalization count, runs `normalizeMessagesForAPI`, then performs model-aware post-processing: strip tool-search-only fields, repair tool_use/tool_result pairing, remove advisor blocks without the beta header, and trim excess media (`src/services/api/claude.ts:1248-1267`, `src/services/api/claude.ts:1269-1319`).
- Deferred tools can be injected as an ephemeral meta user message after fingerprinting and before the request is built (`src/services/api/claude.ts:1322-1345`).
- System prompt blocks and all tool schemas are finalized before request construction; prompt cache break detection records the exact cache-affecting system/tool/model/beta/body state, and LLM request tracing starts with `messagesForAPI` (`src/services/api/claude.ts:1357-1396`, `src/services/api/claude.ts:1460-1503`).
- `paramsFromContext(...)` constructs the Anthropic request payload: normalized model, cache-breakpointed messages, system blocks, tools, tool choice, beta headers, metadata, max tokens, thinking, optional temperature, context management, extra body params, output config, and speed (`src/services/api/claude.ts:1538-1728`).
- Immediately before dispatch, Claude calls `paramsFromContext`, captures the API request for bug reports, records request checkpoints, and sends `{ ...params, stream: true }` through `anthropic.beta.messages.create(...)` (`src/services/api/claude.ts:1797-1836`).
- `toolToAPISchema(...)` converts tool definitions into Anthropic tool schemas, caching base name/description/input_schema/strict/eager-streaming fields and overlaying per-request `defer_loading` and `cache_control` (`src/utils/api.ts:119-220`, `src/utils/api.ts:223-265`).
- `appendSystemContext(...)` and `prependUserContext(...)` are the explicit helpers that attach system/user context to the provider-bound inputs (`src/utils/api.ts:437-474`).

Codex source:
- User input becomes provider-compatible response items via `Session::response_item_from_user_input(...)`, which chooses local image preparation mode and calls `ResponseInputItem::from_user_input(...)`, then converts it to `ResponseItem` (`codex-rs/core/src/session/mod.rs:2642-2656`).
- `ResponseInputItem` is the accepted input-side enum for user messages and tool outputs; `ContentItem` carries input text/image content (`codex-rs/protocol/src/models.rs:804-857`).
- `ResponseInputItem::from_user_input(...)` converts `UserInput::Text` to `ContentItem::InputText`, remote images to `InputImage`, local images to processed/deferred image content or an error placeholder, and drops `Skill`/`Mention` input because tool bodies are injected later in core (`codex-rs/protocol/src/models.rs:1539-1595`).
- `impl From<ResponseInputItem> for ResponseItem` maps input messages and tool-output inputs into history/prompt `ResponseItem` variants, including MCP output conversion into a function-call output payload (`codex-rs/protocol/src/models.rs:1420-1470`).
- `record_conversation_items(...)` prepares items for history, records them into `ContextManager` with the turn truncation policy, persists rollout response items, and emits raw response item events (`codex-rs/core/src/session/mod.rs:2658-2670`).
- `ContextManager::record_items(...)` only records API-message items, processes each item, and appends it to oldest-to-newest history; `for_prompt(...)` consumes a history snapshot, runs normalization, and returns model-bound `Vec<ResponseItem>` (`codex-rs/core/src/context_manager/history.rs:90-114`).
- `normalize_history(...)` enforces provider-prompt invariants: every call has an output, every output has a matching call, and images are stripped when the model lacks image input support (`codex-rs/core/src/context_manager/history.rs:323-336`).
- `ensure_call_outputs_present(...)` inserts synthetic aborted outputs after missing function/tool-search/custom/local-shell calls, while `remove_orphan_outputs(...)` drops unmatched outputs except server tool-search outputs or outputs with no call id where allowed (`codex-rs/core/src/context_manager/normalize.rs:14-118`, `codex-rs/core/src/context_manager/normalize.rs:120-192`).
- `strip_images_when_unsupported(...)` replaces unsupported message/tool-output image content with text placeholders and clears image-generation results (`codex-rs/core/src/context_manager/normalize.rs:291-340`).
- The turn loop builds `sampling_request_input` by cloning history and calling `for_prompt(&turn_context.model_info.input_modalities)` immediately before sampling (`codex-rs/core/src/session/turn.rs:221-228`).
- `build_prompt(...)` packages provider-bound input with model-visible tool specs, parallel-tool-call support, base instructions, personality, and optional output schema settings (`codex-rs/core/src/session/turn.rs:1016-1034`).
- `try_run_sampling_request(...)` passes the `Prompt` to `ModelClientSession::stream(...)` together with model info, telemetry, reasoning settings, service tier, response metadata, and inference trace context (`codex-rs/core/src/session/turn.rs:1849-1895`).
- `Prompt` is the request payload object for one model turn; `get_formatted_input_for_request(...)` clones prompt input and strips image detail fields for Responses Lite (`codex-rs/core/src/client_common.rs:17-40`, `codex-rs/core/src/client_common.rs:56-106`).
- `ModelClient::build_responses_request(...)` extracts instructions from `prompt.base_instructions.text`, formats input, clears metadata for non-OpenAI providers, JSON-serializes tools, builds reasoning/include/text controls, attaches prompt cache key/service tier/client metadata, and returns `ResponsesApiRequest` (`codex-rs/core/src/client.rs:780-830`).
- `ResponsesApiRequest` is the canonical Responses payload with model, instructions, input, tools, tool choice, parallel tool calls, reasoning, store, stream, include, service tier, prompt cache key, text controls, and client metadata. It can be converted into `ResponseCreateWsRequest` for websocket transport (`codex-rs/codex-api/src/common.rs:182-253`).
- HTTP streaming builds the request, records inference trace start with that request, and sends it through `ApiResponsesClient::stream_request(...)`; websocket streaming builds the same logical request, converts it to a websocket payload, and records either the logical request or websocket request depending on warmup reuse (`codex-rs/core/src/client.rs:1302-1320`, `codex-rs/core/src/client.rs:1407-1486`).
- `ModelClientSession::stream(...)` selects Responses websocket when supported/healthy and falls back to HTTP Responses otherwise (`codex-rs/core/src/client.rs:1610-1668`).
- `ApiResponsesClient::stream(...)` encodes the JSON body and posts it to the `responses` endpoint with `Accept: text/event-stream` (`codex-rs/codex-api/src/endpoint/responses.rs:118-172`).
- `prompt_debug::build_prompt_input_from_session(...)` exposes a debug surface that records optional input, builds prompt input through the same `for_prompt` and `build_prompt` path, and returns `prompt.input` (`codex-rs/core/src/prompt_debug.rs:74-101`).

Runtime trace evidence:
- Claude: `query.ts` prepares provider inputs with `appendSystemContext` and `prependUserContext` -> `claude.ts` builds tool schemas and calls `normalizeMessagesForAPI` -> model-aware repair/post-processing and synthetic deferred-tool injection run -> `paramsFromContext` creates Anthropic payload -> `captureAPIRequest` and `anthropic.beta.messages.create({ ...params, stream: true })` dispatch it. Evidence: `src/query.ts:449-451`, `src/query.ts:659-708`, `src/services/api/claude.ts:1248-1345`, `src/services/api/claude.ts:1538-1836`.
- Codex: user/tool/history items are recorded as `ResponseItem`s -> `ContextManager::for_prompt` normalizes the cloned history snapshot -> `build_prompt` wraps input/tools/base instructions -> `build_responses_request` creates `ResponsesApiRequest` -> HTTP/websocket transport serializes or converts the same logical request and starts inference tracing. Evidence: `codex-rs/core/src/session/mod.rs:2642-2670`, `codex-rs/core/src/context_manager/history.rs:90-114`, `codex-rs/core/src/session/turn.rs:221-245`, `codex-rs/core/src/session/turn.rs:1016-1034`, `codex-rs/core/src/client.rs:780-830`, `codex-rs/core/src/client.rs:1302-1320`, `codex-rs/core/src/client.rs:1407-1486`.

What the model sees:
- Claude: Anthropic sees only the normalized `messages` returned by `normalizeMessagesForAPI` after `claude.ts` post-processing, optional synthetic deferred-tools meta message, system blocks, tool schemas, and request options assembled by `paramsFromContext` (`src/utils/messages.ts:1989-2369`, `src/services/api/claude.ts:1269-1345`, `src/services/api/claude.ts:1699-1728`).
- Codex: OpenAI Responses sees `ResponsesApiRequest.input` from `Prompt::get_formatted_input_for_request`, `instructions` from `BaseInstructions`, JSON tools from model-visible tool specs, and request controls in `ResponsesApiRequest` (`codex-rs/core/src/client_common.rs:56-106`, `codex-rs/core/src/client.rs:780-830`, `codex-rs/codex-api/src/common.rs:182-253`).

What the runtime does:
- Claude: performs aggressive API-bound cleanup at send time: removes display-only/virtual/progress/error artifacts, converts local-command system messages to user messages, normalizes tool-use blocks, repairs pairing/order/media/thinking edge cases, builds system/tool/cache controls, and captures the final request before dispatch.
- Codex: records prompt-capable `ResponseItem`s into history as they happen, then normalizes a cloned history snapshot immediately before sampling; the request builder applies model/provider formatting such as Responses Lite image-detail stripping and non-OpenAI metadata removal.

What gets persisted:
- Claude: this mechanism itself prepares provider payloads; source evidence here shows request capture for bug reports via `captureAPIRequest(params, options.querySource)` and optional dump-prompts fetch setup, not transcript persistence (`src/query.ts:582-590`, `src/services/api/claude.ts:1797-1799`).
- Codex: `record_conversation_items(...)` persists response items to rollout and raw-response-item streams before later prompt construction; inference trace can persist the exact request payload via `InferenceTraceAttempt::record_started(...)` (`codex-rs/core/src/session/mod.rs:2658-2670`, `codex-rs/rollout-trace/src/inference.rs:169-190`).

What the user/TUI sees:
- Claude: most normalization output is not a direct TUI surface. The user-visible distinction is that virtual/display-only messages and synthetic API-error artifacts are explicitly blocked from provider payloads (`src/utils/messages.ts:1996-2001`, `src/utils/messages.ts:2056-2074`).
- Codex: prompt construction is separate from display events. `prompt_debug::build_prompt_input_from_session(...)` is a debug/API surface for inspecting the provider-bound prompt input, while normal TUI clients see protocol events and raw response items from other paths (`codex-rs/core/src/prompt_debug.rs:74-101`, `codex-rs/core/src/session/mod.rs:2658-2670`).

What debug/eval surfaces exist:
- Claude: logs `tengu_api_before_normalize` and `tengu_api_after_normalize`, records query checkpoints for message normalization and API send, captures API requests for bug reports, and records prompt-cache state when cache-break detection is enabled (`src/services/api/claude.ts:1259-1267`, `src/services/api/claude.ts:1317-1325`, `src/services/api/claude.ts:1460-1503`, `src/services/api/claude.ts:1797-1805`).
- Codex: inference tracing records the model-visible request payload as `RawPayloadKind::InferenceRequest`; HTTP and websocket paths call `record_started(...)` before dispatch, and `prompt_debug` can rebuild prompt input through the same normalization path (`codex-rs/rollout-trace/src/inference.rs:169-190`, `codex-rs/core/src/client.rs:1311-1320`, `codex-rs/core/src/client.rs:1470-1486`, `codex-rs/core/src/prompt_debug.rs:74-101`).

Shared invariant:
- Provider payload construction is a named boundary after transcript/history collection and before transport dispatch. At that boundary, both systems remove or repair artifacts that are valid internally but invalid or misleading for the model, then trace/capture the exact model-visible request shape.

Claude-specific design:
- Claude centralizes most send-time cleanup in `normalizeMessagesForAPI(...)` and then adds model-aware repairs in `claude.ts`.
- Claude has source-specific cleanups for virtual REPL inner-tool messages, synthetic API errors, tool-search `tool_reference` blocks, `caller` fields, thinking-only assistant artifacts, history snip id tags, and Anthropic media limits.
- Claude builds an Anthropic Messages payload with separate `messages`, `system`, `tools`, beta headers, cache breakpoints, thinking/context management, and optional fast/output config fields.

Codex-specific design:
- Codex uses `ResponseItem` as the durable history/prompt currency and normalizes a cloned `ContextManager` snapshot through `for_prompt(...)`.
- Codex repairs missing/orphaned tool outputs at history-normalization time and strips images according to `model_info.input_modalities`.
- Codex request construction is split between `Prompt`, `build_responses_request(...)`, and transport-specific HTTP/websocket wrappers; websocket may send a compressed transport payload while the inference trace records the logical request when needed.

Gaps or unknowns:
- Claude request capture implementation (`captureAPIRequest`) was not deeply traced here; this card cites the call site because detailed debug-bundle behavior belongs to mechanism 26.
- Codex `prepare_conversation_items_for_history(...)` is outside this card's deep trace; this card cites the record boundary but leaves detailed history repair/resume behavior for mechanisms 12 and 23.
- Tool schema projection details are only cited where they enter provider payload construction. The full tool registry/model-visible schema mechanism belongs to mechanism 10.
- System prompt content and instruction layering are intentionally not expanded beyond payload placement; that is mechanism 6.

Distilled harness rule:
- Build an explicit provider-payload boundary that consumes durable internal history, applies source-proven provider compatibility normalization, then constructs a typed request object that can be captured before transport dispatch.

Anti-invention rule:
- Do not invent a universal normalizer. Claude's `normalizeMessagesForAPI` is a source-specific send-time pipeline; Codex's normalization is split across input conversion, history recording, `ContextManager::for_prompt`, `Prompt`, and `build_responses_request`. Reuse only the invariant unless both sources show the same mechanism.

Verification pattern:
- Find the internal transcript/history item type.
- Find how user input and tool outputs become that internal type.
- Find the last function that prepares history before model sampling.
- Find all filters/repairs that remove internal-only, display-only, malformed, or provider-unsupported content.
- Find the typed provider request object and every field copied into it.
- Find where model/provider-specific transformations happen.
- Find the exact call site that captures/logs/traces the request.
- Find the transport call that serializes or sends the request.

Skill text candidate:
- When analyzing provider payload construction, trace from stored/internal message items to the final typed provider request. Separate input conversion, history normalization, request formatting, transport wrapping, persistence, display, and tracing. A reusable harness rule may say "normalize before provider dispatch"; it must not copy Claude-only cleanup passes or Codex-only `ResponseItem` assumptions unless the target source proves equivalent structures.
