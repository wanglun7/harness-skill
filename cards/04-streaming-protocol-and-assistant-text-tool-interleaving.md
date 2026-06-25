# Mechanism 4: Streaming protocol and assistant text/tool interleaving

Mechanism: Streaming protocol and assistant text/tool interleaving

Status: complete

Claude source:
- The streaming API path holds per-response assembly state: `newMessages`, `partialMessage`, `contentBlocks`, usage, cost, stop reason, and fallback state (`src/services/api/claude.ts:1761-1770`).
- The stream comes from the Anthropic SDK raw message stream. After retry setup, `claude.ts` receives a `Stream<BetaRawMessageStreamEvent>`, resets assembly state, and then iterates raw chunks (`src/services/api/claude.ts:1509-1525`, `src/services/api/claude.ts:1848-1867`).
- On the first chunk, Claude logs stream start, records query checkpoints, and ends the query profile before processing the chunk (`src/services/api/claude.ts:1969-1977`).
- `message_start` initializes `partialMessage`, time-to-first-token, and usage; `content_block_start` initializes an immutable local content block slot by index (`src/services/api/claude.ts:1979-2052`).
- Tool-call arguments are streamed into content-block state. `input_json_delta` is accepted only for `tool_use` or `server_tool_use` blocks, verifies the accumulated input is a string, and appends `delta.partial_json` (`src/services/api/claude.ts:2087-2112`).
- Assistant text and thinking are streamed into separate block fields. `text_delta` appends to a `text` block; `thinking_delta` appends to a `thinking` block; `signature_delta` writes the thinking or connector-text signature (`src/services/api/claude.ts:2113-2161`).
- At `content_block_stop`, Claude requires both the content block and `partialMessage`, normalizes that single completed block into an `AssistantMessage`, pushes it into `newMessages`, and yields the completed assistant message (`src/services/api/claude.ts:2171-2211`).
- `message_delta` arrives after content-block stop; Claude mutates the already-yielded last message in place to attach final usage and stop reason so lazy transcript serialization sees final values, then updates cost and may yield synthetic API-error messages for max-token/context-window terminal conditions (`src/services/api/claude.ts:2229-2276`).
- After processing every raw part, Claude yields a `StreamEvent` wrapper `{ type: 'stream_event', event: part, ttftMs? }` (`src/services/api/claude.ts:2295-2304`).
- In `query.ts`, model stream messages are post-processed before external yield. For assistant messages, Claude may clone only the yielded message to backfill observable tool input fields while leaving the original assistant message untouched for API replay/prompt caching (`src/query.ts:742-787`).
- Recoverable assistant error messages may be withheld from the user stream but still pushed to `assistantMessages` so recovery checks can inspect them (`src/query.ts:788-825`).
- Completed assistant messages are stored in `assistantMessages`. Any `tool_use` blocks in the assistant message are appended to `toolUseBlocks`, set `needsFollowUp = true`, and, when streaming tool execution is enabled, are immediately handed to `StreamingToolExecutor.addTool()` (`src/query.ts:826-845`).
- While the model stream is still active, `query.ts` polls `streamingToolExecutor.getCompletedResults()` and yields completed tool progress/results interleaved with later assistant stream output; normalized tool-result user messages are accumulated for model feedback (`src/query.ts:847-862`).
- If streaming fallback happens, partial assistant messages are tombstoned, pending tool results are discarded, and a fresh streaming tool executor is created to prevent orphaned tool results from old tool-use IDs (`src/query.ts:708-740`).
- After model streaming ends, Claude drains any remaining streaming tool results or falls back to `runTools(...)`; every yielded tool update is normalized into next-turn user/tool-result messages (`src/query.ts:1366-1408`).
- `StreamingToolExecutor` tracks tools as queued/executing/completed/yielded, stores progress separately, and starts tools when concurrency rules allow (`src/services/tools/StreamingToolExecutor.ts:19-40`, `src/services/tools/StreamingToolExecutor.ts:73-124`, `src/services/tools/StreamingToolExecutor.ts:126-151`).
- `StreamingToolExecutor` converts missing/discarded/interrupted/sibling-error paths into synthetic `tool_result` user messages with the original assistant message UUID (`src/services/tools/StreamingToolExecutor.ts:153-230`).
- Tool execution collects non-progress messages as results, stores progress messages separately for immediate yielding, applies non-concurrent context modifiers after completion, and marks tool status complete (`src/services/tools/StreamingToolExecutor.ts:320-405`).
- `getCompletedResults()` yields pending progress immediately and completed tool results in tracked order; `getRemainingResults()` waits for execution/progress and drains remaining results (`src/services/tools/StreamingToolExecutor.ts:407-440`, `src/services/tools/StreamingToolExecutor.ts:449-490`).
- `QueryEngine.submitMessage()` records transcript messages before yielding SDK messages. Assistant transcript writes are intentionally fire-and-forget so later `message_delta` mutations can update usage/stop reason before lazy serialization (`src/QueryEngine.ts:720-731`).
- In `QueryEngine`, assistant/progress/user messages are normalized into SDK messages, while `stream_event` messages update usage/stop reason and are yielded to SDK consumers only when `includePartialMessages` is true (`src/QueryEngine.ts:757-828`).
- `normalizeMessage()` maps completed assistant messages to SDK assistant messages, progress messages to agent/tool-progress SDK shapes, and user/tool-result messages to SDK user messages with `tool_use_result` metadata (`src/utils/queryHelpers.ts:102-222`).
- VCR fixtures can record and replay outputs containing `AssistantMessage | StreamEvent | SystemAPIErrorMessage`; cached stream events do not add cost (`src/services/vcr.ts:88-125`, `src/services/vcr.ts:163-172`, `src/services/vcr.ts:279-288`).

Codex source:
- Codex normalizes provider stream events into a `ResponseEvent` enum with created, item-added, item-done, completed, text delta, tool-call input delta, reasoning deltas, model verification, moderation, and rate-limit variants (`codex-rs/codex-api/src/common.rs:72-105`).
- The SSE parser maps `response.output_item.done` to `ResponseEvent::OutputItemDone`, `response.output_text.delta` to `OutputTextDelta`, `response.custom_tool_call_input.delta` to `ToolCallInputDelta`, reasoning text/summary deltas to reasoning delta variants, `response.completed` to `Completed`, and `response.output_item.added` to `OutputItemAdded` (`codex-rs/codex-api/src/sse/responses.rs:298-325`, `codex-rs/codex-api/src/sse/responses.rs:326-345`, `codex-rs/codex-api/src/sse/responses.rs:393-418`).
- `try_run_sampling_request()` receives `ResponseEvent`s from `client_session.stream(...)`, records response telemetry, records TTFT, and matches each event in a receive loop (`codex-rs/core/src/session/turn.rs:1849-1955`, `codex-rs/core/src/session/turn.rs:1956-2288`).
- On `OutputItemAdded`, Codex creates an optional `ToolArgumentDiffConsumer` for `CustomToolCall` names, clears it for function calls, converts non-tool response items to `TurnItem`s, may seed assistant text parser state from raw output text, emits `ItemStarted` unless plan-mode defers it, and marks the item active for future deltas (`codex-rs/core/src/session/turn.rs:2051-2121`).
- `handle_non_tool_response_item()` maps message/reasoning/web-search/image-generation response items into `TurnItem`s and runs finalization; tool output items are treated as unexpected stream outputs for this path (`codex-rs/core/src/stream_events_utils.rs:517-553`).
- `finalize_turn_item()` strips hidden citations/proposed-plan markup from `AgentMessage` turn items and stores parsed memory citation when present (`codex-rs/core/src/stream_events_utils.rs:555-586`).
- On `OutputTextDelta`, Codex emits `AgentMessageContentDelta` for the active item. For agent messages, it parses assistant text deltas first, strips hidden citations/plan segments, and in plan mode may split deltas into normal assistant text versus `PlanDelta` events (`codex-rs/core/src/session/turn.rs:1544-1642`, `codex-rs/core/src/session/turn.rs:2186-2217`).
- On `ToolCallInputDelta`, Codex sends the partial argument delta to the active tool's diff consumer; any generated protocol event is sent immediately (`codex-rs/core/src/session/turn.rs:2218-2235`).
- `ToolArgumentDiffConsumer` is an optional per-tool interface that consumes streamed argument diffs and may emit protocol events; the registry/router/runtime expose `create_diff_consumer()` down to individual tool handlers (`codex-rs/core/src/tools/registry.rs:140-157`, `codex-rs/core/src/tools/registry.rs:316-318`, `codex-rs/core/src/tools/registry.rs:373-378`, `codex-rs/core/src/tools/parallel.rs:55-60`).
- Apply-patch is a concrete source-specific diff consumer: it parses streamed patch deltas, buffers updates by interval, emits `PatchApplyUpdated`, flushes pending updates on finish, and registers its consumer from the apply-patch handler (`codex-rs/core/src/tools/handlers/apply_patch.rs:71-132`, `codex-rs/core/src/tools/handlers/apply_patch.rs:475-482`).
- On reasoning summary/raw reasoning deltas, Codex emits `ReasoningContentDelta`, `AgentReasoningSectionBreak`, or `ReasoningRawContentDelta` events against the active item (`codex-rs/core/src/session/turn.rs:2236-2288`).
- On `OutputItemDone`, Codex finishes any active argument diff consumer, flushes assistant text parser state for completed agent messages, runs plan-mode completion handling if needed, then calls `handle_output_item_done()` to persist/complete the item, queue any tool future, update last-agent-message, and OR item-level follow-up into sampling state (`codex-rs/core/src/session/turn.rs:1956-2050`).
- `handle_output_item_done()` records tool-call response items immediately and queues tool execution futures with `needs_follow_up = true`; non-tool items are converted to completed `TurnItem`s, emitted as completed, and recorded into history/rollout (`codex-rs/core/src/stream_events_utils.rs:303-315`, `codex-rs/core/src/stream_events_utils.rs:404-515`).
- `record_completed_response_item()` records completed response items into conversation history and rollout; it also records memory-citation usage and mailbox deferral facts (`codex-rs/core/src/stream_events_utils.rs:190-244`).
- `emit_turn_item_started()` and `emit_turn_item_completed()` send structured `ItemStarted`/`ItemCompleted` protocol events with turn id, item payload, and timestamps (`codex-rs/core/src/session/mod.rs:1851-1880`; event fields in `codex-rs/protocol/src/protocol.rs:1736-1773`).
- `AgentMessageContentDelta`, `PlanDelta`, `ReasoningContentDelta`, and `ReasoningRawContentDelta` are explicit protocol event structs for streaming display deltas (`codex-rs/protocol/src/protocol.rs:1374-1383`, `codex-rs/protocol/src/protocol.rs:1795-1848`).
- `send_event_raw()` persists every protocol event as `RolloutItem::EventMsg`, records it in rollout trace, and delivers it to clients; `record_conversation_items()` separately appends completed response items to prompt history, persists response items, and emits `RawResponseItem` events (`codex-rs/core/src/session/mod.rs:1831-1849`, `codex-rs/core/src/session/mod.rs:2626-2670`, `codex-rs/core/src/session/mod.rs:2835-2843`).

Runtime trace evidence:
- Claude: Anthropic raw stream -> `claude.ts` accumulates `contentBlocks` and yields both completed `AssistantMessage`s and raw `stream_event`s -> `query.ts` stores assistant messages, detects `tool_use`, starts/polls `StreamingToolExecutor`, and interleaves ready tool progress/results -> `QueryEngine` records completed messages and optionally exposes partial stream events. Evidence: `src/services/api/claude.ts:1979-2304`, `src/query.ts:742-862`, `src/query.ts:1366-1408`, `src/QueryEngine.ts:720-828`.
- Codex: Responses SSE/websocket event -> `ResponseEvent` -> `try_run_sampling_request()` emits `ItemStarted` and delta protocol events while an item is active -> `OutputItemDone` finalizes/persists completed response items and queues tool futures -> protocol events and raw/completed response items are persisted through separate event/history paths. Evidence: `codex-rs/codex-api/src/common.rs:72-105`, `codex-rs/codex-api/src/sse/responses.rs:298-418`, `codex-rs/core/src/session/turn.rs:1956-2288`, `codex-rs/core/src/stream_events_utils.rs:190-244`, `codex-rs/core/src/stream_events_utils.rs:404-515`, `codex-rs/core/src/session/mod.rs:1831-1880`.

What the model sees:
- Claude: the model does not see `StreamEvent` wrappers or partial deltas. It sees completed assistant messages built at `content_block_stop` and, on follow-up turns, completed/synthetic tool-result user messages normalized from tool updates (`src/services/api/claude.ts:2171-2211`, `src/query.ts:826-845`, `src/query.ts:847-862`, `src/query.ts:1366-1408`).
- Codex: the next model prompt is built from completed `ResponseItem`s recorded into history, not from `AgentMessageContentDelta` or other display deltas. Tool-call response items are recorded immediately, and tool outputs become response items through the tool execution path before follow-up sampling (`codex-rs/core/src/stream_events_utils.rs:190-244`, `codex-rs/core/src/stream_events_utils.rs:404-515`, `codex-rs/core/src/session/mod.rs:2626-2670`).

What the runtime does:
- Claude: reconstructs one assistant message per completed content block, emits raw stream events after processing each chunk, and can start tool execution as soon as a completed `tool_use` block is yielded, before the full model stream has ended.
- Codex: turns provider stream events into a protocol item lifecycle: start item, stream text/reasoning/tool-argument display deltas, complete item, record completed response item, and queue tool execution futures.

What gets persisted:
- Claude: `QueryEngine` records completed assistant/progress/user messages into transcript; `stream_event` messages only update usage/stop reason and are yielded to SDK consumers when `includePartialMessages` is enabled, with no transcript push in that branch (`src/QueryEngine.ts:720-828`). VCR can persist/replay stream events in fixtures (`src/services/vcr.ts:88-125`).
- Codex: protocol streaming events such as item starts, deltas, and completions are persisted as `RolloutItem::EventMsg` through `send_event_raw()`; completed model response items are also recorded into conversation history and persisted as rollout response items through `record_conversation_items()` (`codex-rs/core/src/session/mod.rs:1831-1849`, `codex-rs/core/src/session/mod.rs:2626-2670`).

What the user/TUI sees:
- Claude: default SDK/headless output sees completed normalized assistant/user/progress messages. Partial raw provider events are visible only when `includePartialMessages` is true; then `QueryEngine` yields `stream_event` SDK messages with the raw event, session id, and UUID (`src/QueryEngine.ts:788-828`). Streaming tool progress can be exposed as SDK `tool_progress` messages by `normalizeMessage()` (`src/utils/queryHelpers.ts:120-202`).
- Codex: clients see structured protocol events: `ItemStarted`, `AgentMessageContentDelta`, `PlanDelta`, `ReasoningContentDelta`, `ReasoningRawContentDelta`, tool-specific events such as `PatchApplyUpdated`, `ItemCompleted`, and `RawResponseItem` (`codex-rs/core/src/session/turn.rs:1544-1642`, `codex-rs/core/src/session/turn.rs:2051-2235`, `codex-rs/core/src/session/mod.rs:1851-1880`, `codex-rs/core/src/session/mod.rs:2835-2843`).

What debug/eval surfaces exist:
- Claude: streaming errors are logged with specific `tengu_streaming_error` categories when deltas arrive for missing/wrong block types (`src/services/api/claude.ts:2053-2164`, `src/services/api/claude.ts:2171-2191`). Stream start/stall/no-events paths emit debug logs/checkpoints/events (`src/services/api/claude.ts:1950-1977`, `src/services/api/claude.ts:2340-2375`). VCR fixtures can include stream events and assistant messages (`src/services/vcr.ts:88-125`).
- Codex: `SessionTelemetry::record_responses()` records response event type and tool name on item-added/item-done events; `responses_type()` classifies text/tool/reasoning deltas (`codex-rs/otel/src/events/session_telemetry.rs:401-418`, `codex-rs/otel/src/events/session_telemetry.rs:1195-1206`). TTFT records output item/text/reasoning deltas but excludes tool input deltas and completion (`codex-rs/core/src/turn_timing.rs:18-26`, `codex-rs/core/src/turn_timing.rs:330-346`). The SSE parser has test coverage for `ToolCallInputDelta` mapping from `response.custom_tool_call_input.delta` (`codex-rs/codex-api/src/sse/responses.rs:816-838`).

Shared invariant:
- Both references separate partial display streaming from completed prompt/history artifacts. Text/tool deltas can be displayed immediately, but model-visible continuation uses completed assistant/tool items with stable IDs and normalized tool-result pairings.

Claude-specific design:
- Claude emits raw provider chunks as `stream_event` after processing each chunk and only exposes them to SDK consumers under `includePartialMessages`.
- Claude constructs an `AssistantMessage` at each `content_block_stop`, so a single provider response can yield multiple assistant messages, each containing one normalized completed content block.
- Claude can start executing tools during model streaming after a completed `tool_use` content block, and can interleave ready tool progress/results before the model stream fully ends.
- Claude preserves original assistant messages for API replay/prompt caching and uses cloned yielded messages for observable tool-input backfill.

Codex-specific design:
- Codex maps provider events into a first-class protocol item lifecycle instead of forwarding raw provider stream chunks.
- Codex streams assistant text as `AgentMessageContentDelta` against an active `TurnItem` and completes the item separately with `ItemCompleted`.
- Codex handles streamed tool-call argument diffs through optional per-tool diff consumers; apply-patch is a concrete source-specific consumer that emits `PatchApplyUpdated`.
- Codex has plan-mode stream parsing that splits assistant text into normal assistant deltas and proposed-plan item/delta events.

Gaps or unknowns:
- Claude imports `AssistantMessage`, `Message`, and `StreamEvent` from `src/types/message.js`, but no corresponding source file was present under the inspected `src/types` tree. This card therefore cites producer/consumer code paths rather than a missing type-definition file.
- This card cites Codex SSE mapping and runtime consumption; websocket-specific mapping was not deeply traced because the selected evidence already establishes the shared `ResponseEvent` boundary consumed by `turn.rs`.
- Detailed tool dispatch scheduling/concurrency belongs to mechanism 11. This card cites only the streaming interleaving boundary and the source-specific Claude `StreamingToolExecutor` behavior needed to explain interleaving.
- Detailed provider payload construction belongs to mechanism 5.

Distilled harness rule:
- Treat streaming as two linked but separate contracts: a display contract for partial deltas and a history/model contract for completed assistant/tool items. Preserve stable item/tool-call IDs across both, and only feed the model completed normalized items with matching tool results.

Anti-invention rule:
- Do not assume every harness should forward raw provider stream events. Claude forwards raw `stream_event`s optionally; Codex translates provider events into its own protocol item/delta/completion events. Do not generalize Claude's streaming tool execution into Codex unless source shows equivalent early tool execution before item completion.

Verification pattern:
- Find the provider stream event enum or raw event type.
- Find where text/tool/reasoning deltas are accumulated or translated.
- Find when a completed assistant/item object is created.
- Find how tool-call argument deltas are represented before tool execution.
- Find whether tool execution can start before the full model response ends.
- Find which events are user/TUI-visible and which are model-visible on the next prompt.
- Find what is persisted as transcript/history versus protocol/trace events.
- Find debug/eval hooks that record or replay streaming events.

Skill text candidate:
- For streaming protocol analysis, cite the event boundary first, then trace one text delta, one tool-call argument delta, and one completed assistant/tool item through runtime, display, persistence, and next-prompt surfaces. Never merge display deltas with model-visible history: source must prove when a streamed fragment becomes a completed message or response item.
