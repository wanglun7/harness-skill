# Mechanism 12: Tool Result Projection

Mechanism: Tool result projection: raw output, model output, display output, persisted output
Status: complete

Claude source:
- `src/Tool.ts:321-335` defines `ToolResult<T>` as native runtime `data` plus optional `newMessages`, `contextModifier`, and MCP metadata.
- `src/Tool.ts:557-560` requires each tool to map native output into an Anthropic `ToolResultBlockParam`.
- `src/Tool.ts:562-599` separates display/search text from model-facing serialization; the comment explicitly warns that rendered transcript text is not the same as `mapToolResultToToolResultBlockParam()`.
- `src/Tool.ts:601-667` defines separate tool-use display, result truncation, rejection, and error render hooks.
- `src/services/tools/toolExecution.ts:1271-1280` captures `structured_output` from a tool result as an attachment message.
- `src/services/tools/toolExecution.ts:1282-1301` converts raw `result.data` to a telemetry string, maps it once through `mapToolResultToToolResultBlockParam()`, and measures the mapped content size.
- `src/services/tools/toolExecution.ts:1403-1473` builds the user `tool_result` message, optionally runs large-result persistence on the mapped block, keeps `toolUseResult` for local rendering, keeps `mcpMeta` for SDK consumers, and records `sourceToolAssistantUUID`.
- `src/services/tools/toolExecution.ts:1540-1588` waits to add MCP tool results until post-tool hooks have had a chance to mutate the output, then appends extra messages and hook output.
- `src/services/tools/toolExecution.ts:1589-1737` builds error `tool_result` blocks with `is_error`, stores local `toolUseResult`, and preserves MCP error metadata when available.
- `src/utils/toolResultStorage.ts:26-34` defines the tool-result subdirectory, persisted-output wrapper tag, and cleared-content marker.
- `src/utils/toolResultStorage.ts:55-78` resolves the per-tool persistence threshold.
- `src/utils/toolResultStorage.ts:137-184` persists only text/text-block tool results under `tool-results/{toolUseId}.json|txt` and returns a preview.
- `src/utils/toolResultStorage.ts:189-241` wraps a large result with a persisted-output message and applies that path to freshly mapped or pre-mapped tool-result blocks.
- `src/utils/toolResultStorage.ts:272-333` handles empty output, skips persistence for image blocks, persists oversized text output, and logs `tengu_tool_result_persisted`.
- `src/utils/messages.ts:460-523` stores `toolUseResult`, `mcpMeta`, and `sourceToolAssistantUUID` on `UserMessage`; `mcpMeta` is documented as never sent to the model.
- `src/utils/messages.ts:2372-2410` identifies tool-result user messages and merges user messages while hoisting tool results.
- `src/utils/messages.ts:2522-2647` folds sibling content blocks into `tool_result.content` to satisfy API/wire constraints, with a special text-only rule for error results.
- `src/utils/messages.ts:4288-4323` rebuilds meta-context tool-result summaries from the native `toolUseResult` by calling the tool's mapper again.
- `src/utils/messages.ts:5118-5458` repairs or fails malformed tool-use/tool-result pairings and logs repair diagnostics.
- `src/utils/sessionStorage.ts:1025-1040` uses `sourceToolAssistantUUID` as the transcript parent for tool-result messages.
- `src/utils/sessionStorage.ts:3240-3354` documents that `toolUseResult` and `mcpMeta` serialize into transcript JSON and require depth-aware parsing.
- `src/components/Message.tsx:406-425` routes `tool_result` content blocks to `UserToolResultMessage`.
- `src/components/messages/UserToolResultMessage/UserToolSuccessMessage.tsx:52-102` renders successful results from local `message.toolUseResult`, validates against the tool output schema on resumed transcripts, calls `renderToolResultMessage()`, and appends PostToolUse progress.

Codex source:
- `codex-rs/tools/src/tool_output.rs:15-53` defines the `ToolOutput` projection contract: telemetry preview, success, external-context flag, model-facing `to_response_item()`, post-tool hook fields, and code-mode result.
- `codex-rs/tools/src/tool_output.rs:122-161` maps `JsonToolOutput` to function or custom tool-call output while preserving typed JSON for hooks/code mode.
- `codex-rs/tools/src/tool_output.rs:164-187` maps raw MCP `CallToolResult` to `McpToolCallOutput` for direct responses and full JSON for code mode.
- `codex-rs/tools/src/tool_output.rs:189-243` converts a `ResponseInputItem` into the code-mode JSON/string result shape.
- `codex-rs/core/src/tools/context.rs:65-140` wraps MCP output with wall-time, image-detail sanitization, function-output truncation, and raw `CallToolResult` code-mode preservation.
- `codex-rs/core/src/tools/context.rs:184-233` maps generic function tool output into model-facing response items and hook response values.
- `codex-rs/core/src/tools/context.rs:235-271` maps apply-patch output to model-facing text while returning an empty object to code mode.
- `codex-rs/core/src/tools/context.rs:307-435` stores raw exec output bytes, builds formatted model output, truncates by policy, and preserves raw or caller-limited output for code mode.
- `codex-rs/core/src/tools/context.rs:438-463` chooses function vs custom output and collapses single text content to a text body.
- `codex-rs/core/src/tools/registry.rs:159-205` packages `AnyToolResult`, exposes `into_response()` and `code_mode_result()`, and defines post-tool feedback that can replace model-visible output while preserving the original code-mode result.
- `codex-rs/core/src/tools/registry.rs:543-567` logs tool execution with `log_preview()` and `success_for_logging()`.
- `codex-rs/core/src/tools/registry.rs:573-604` builds and runs post-tool-use hook payloads from the result projection contract.
- `codex-rs/core/src/tools/registry.rs:606-667` applies post-tool blocking or model-visible feedback, then records completed/failed dispatch traces.
- `codex-rs/core/src/tools/registry.rs:684-708` invokes the runtime handler, marks memory polluted for external context, and packages the result with hook payload.
- `codex-rs/protocol/src/models.rs:804-840` defines model input output variants for function, MCP, custom, and tool-search results.
- `codex-rs/protocol/src/models.rs:1420-1468` converts `ResponseInputItem` outputs into persistent/history `ResponseItem` outputs, including MCP conversion into function-call output payload.
- `codex-rs/protocol/src/models.rs:1703-1810` defines `FunctionCallOutputPayload`: `body` is serialized as the wire output, while `success` stays internal metadata.
- `codex-rs/protocol/src/models.rs:1813-1950` converts MCP `CallToolResult` into function-call output, preferring structured content, converting image/text content items, and preserving success state.
- `codex-rs/core/src/session/turn.rs:1822-1847` drains completed tool futures, converts `ResponseInputItem` to `ResponseItem`, records conversation items, and marks memory pollution for external context.
- `codex-rs/core/src/session/mod.rs:2658-2670` records conversation items in state with truncation policy, persists rollout response items, and emits raw response items.
- `codex-rs/core/src/context_manager/history.rs:347-375` truncates function and custom tool-call outputs before storing history; other response items pass through unchanged.
- `codex-rs/core/src/context_manager/history.rs:426-439` applies truncation to text or content-item function output payloads.
- `codex-rs/core/src/tools/tool_dispatch_trace.rs:24-115` records direct tool results as `ResponseInputItem` and code-mode results as raw JSON values.
- `codex-rs/rollout-trace/src/tool_dispatch.rs:98-127` defines trace payload shapes for direct response, code-mode response, and error results.
- `codex-rs/protocol/src/protocol.rs:1293-1328` exposes display/lifecycle event variants for MCP, web/image, exec, permissions, and dynamic tools.
- `codex-rs/protocol/src/protocol.rs:2313-2361` carries MCP end-event results and dynamic tool response content/success for UI consumers.
- `codex-rs/protocol/src/protocol.rs:3227-3293` carries exec begin/end display data, including stdout/stderr/aggregated output and formatted model output.
- `codex-rs/tui/src/thread_transcript.rs:172-198` renders transcript summaries for MCP, dynamic, collab-agent, and subagent tool activities.
- `codex-rs/tui/src/chatwidget/protocol.rs:285-306` routes started command, file-change, MCP, web-search, image-generation, and collab-agent items to tool-specific UI handlers.

Runtime trace evidence:
- Claude success path: `tool.call()` returns `ToolResult.data`; `toolExecution.ts` maps `result.data` to a `ToolResultBlockParam`, possibly persists or wraps the mapped model block, and creates a user message whose model-facing `content` is the `tool_result` block while local fields keep `toolUseResult`, `mcpMeta`, and source assistant UUID (`src/services/tools/toolExecution.ts:1271-1473`).
- Claude MCP path: MCP results may be mutated by post-tool hooks before `addToolResult()` is called, so the final mapped model block is delayed until after those hooks (`src/services/tools/toolExecution.ts:1397-1415`, `src/services/tools/toolExecution.ts:1540-1542`).
- Claude error path: thrown errors are formatted into an error `tool_result` block and also stored as local `toolUseResult`, with optional MCP metadata (`src/services/tools/toolExecution.ts:1589-1737`).
- Codex direct path: a handler returns `Box<dyn ToolOutput>`, the registry logs preview/success, post hooks may wrap the result, `AnyToolResult.into_response()` calls `ToolOutput.to_response_item()`, and `turn.rs` records the resulting `ResponseItem` (`codex-rs/core/src/tools/registry.rs:543-667`, `codex-rs/core/src/tools/registry.rs:159-181`, `codex-rs/core/src/session/turn.rs:1822-1847`).
- Codex code-mode path: the same `ToolOutput` object can return `code_mode_result()` instead of a model-visible response item; dispatch tracing stores direct responses as response items and code-mode responses as JSON values (`codex-rs/tools/src/tool_output.rs:50-52`, `codex-rs/core/src/tools/tool_dispatch_trace.rs:87-100`).

What the model sees:
- Claude: the model sees Anthropic `tool_result` content blocks produced by each tool's `mapToolResultToToolResultBlockParam()` and then normalized/persisted/smooshed before provider submission. It does not see the local `toolUseResult` field or `mcpMeta`; `mcpMeta` is explicitly for SDK consumers and "never sent to model" (`src/Tool.ts:557-560`, `src/services/tools/toolExecution.ts:1403-1473`, `src/utils/messages.ts:460-523`, `src/utils/messages.ts:2522-2647`).
- Claude: if a tool result is empty, the model-visible block is replaced with a short completion marker; if it is oversized text, the model sees a persisted-output wrapper with a preview and file path instead of the full output (`src/utils/toolResultStorage.ts:272-333`, `src/utils/toolResultStorage.ts:189-198`).
- Codex: the model sees `ResponseInputItem::FunctionCallOutput`, `CustomToolCallOutput`, `ToolSearchOutput`, or MCP converted to function-call output. `FunctionCallOutputPayload.body` serializes as the wire output; `success` stays internal metadata (`codex-rs/protocol/src/models.rs:804-840`, `codex-rs/protocol/src/models.rs:1703-1810`).
- Codex: MCP direct output may be transformed before model injection: structured content is preferred, wall time is prepended, images may become input-image content items, and the function-output payload is truncated for context injection (`codex-rs/core/src/tools/context.rs:110-140`, `codex-rs/protocol/src/models.rs:1813-1950`).

What the runtime does:
- Claude: runtime keeps two projections in the same user message: `message.content` is the model/API payload, and `message.toolUseResult` is the native output used by local UI, transcript rendering, schema validation, and SDK mapping (`src/services/tools/toolExecution.ts:1456-1466`, `src/components/messages/UserToolResultMessage/UserToolSuccessMessage.tsx:52-102`, `src/utils/queryHelpers.ts:150-152`, `src/utils/queryHelpers.ts:213-215`).
- Claude: runtime may append attachment/new messages, preserve context modifiers, and repair pairing or sibling-layout problems before future provider calls (`src/services/tools/toolExecution.ts:1271-1280`, `src/services/tools/toolExecution.ts:1565-1588`, `src/utils/messages.ts:2372-2410`, `src/utils/messages.ts:5118-5458`).
- Codex: runtime centralizes result projection in the `ToolOutput` trait instead of per-message side fields. Each output type owns model-facing conversion, hook-facing data, code-mode result, logging preview, and success state (`codex-rs/tools/src/tool_output.rs:15-53`, `codex-rs/core/src/tools/context.rs:65-463`).
- Codex: runtime uses the same packaged `AnyToolResult` for direct model response, code-mode result, post-hook feedback replacement, lifecycle outcome, telemetry, and dispatch tracing (`codex-rs/core/src/tools/registry.rs:159-205`, `codex-rs/core/src/tools/registry.rs:543-667`).

What gets persisted:
- Claude: oversized text tool output can be written under the session tool-results directory and replaced in model context with a persisted-output wrapper; image content is not persisted by this path (`src/utils/toolResultStorage.ts:26-34`, `src/utils/toolResultStorage.ts:137-184`, `src/utils/toolResultStorage.ts:301-333`).
- Claude: transcript storage records tool-result messages under the matching assistant-message UUID, and the serialized JSON can contain `toolUseResult` and `mcpMeta` as top-level message fields (`src/services/tools/toolExecution.ts:1456-1466`, `src/utils/sessionStorage.ts:1025-1040`, `src/utils/sessionStorage.ts:3240-3354`).
- Codex: completed tool output is recorded as `ResponseItem` conversation history, persisted to rollout response items, and sent as raw response items; history truncation applies to function/custom tool-call output payloads (`codex-rs/core/src/session/turn.rs:1822-1847`, `codex-rs/core/src/session/mod.rs:2658-2670`, `codex-rs/core/src/context_manager/history.rs:347-375`, `codex-rs/core/src/context_manager/history.rs:426-439`).
- Codex: no central source equivalent to Claude's generic "persist huge arbitrary tool result to a sidecar file and replace the model payload with a preview" was found in the cited result-projection path; Codex-specific storage here is conversation/rollout recording plus truncation.

What the user/TUI sees:
- Claude: `tool_result` blocks are dispatched to `UserToolResultMessage`; successful rendering reads `message.toolUseResult`, validates it against the tool output schema when possible, and calls the tool's `renderToolResultMessage()` rather than rendering the model-visible block directly (`src/components/Message.tsx:406-425`, `src/components/messages/UserToolResultMessage/UserToolSuccessMessage.tsx:52-102`).
- Claude: tools may opt out of result rendering by omitting `renderToolResultMessage()` or returning `null`; transcript search has a separate `extractSearchText()` contract for visible text (`src/Tool.ts:562-599`).
- Codex: display is not a generic rendering of `ResponseInputItem`; protocol events and `TurnItem` summaries carry tool-specific display data. MCP/dynamic/exec end events include result or output fields, and TUI routes started/completed items to tool-specific handlers and transcript summaries (`codex-rs/protocol/src/protocol.rs:1293-1328`, `codex-rs/protocol/src/protocol.rs:2313-2361`, `codex-rs/protocol/src/protocol.rs:3227-3293`, `codex-rs/tui/src/chatwidget/protocol.rs:285-306`, `codex-rs/tui/src/thread_transcript.rs:172-198`).

What debug/eval surfaces exist:
- Claude: success and failure paths log `tengu_tool_use_success`, `tengu_tool_use_error`, OTLP `tool_result`, empty-result events, persisted-result events, and pairing-repair diagnostics (`src/services/tools/toolExecution.ts:1331-1395`, `src/services/tools/toolExecution.ts:1631-1689`, `src/utils/toolResultStorage.ts:287-294`, `src/utils/toolResultStorage.ts:323-333`, `src/utils/messages.ts:5437-5455`).
- Codex: `log_preview()` and `success_for_logging()` feed telemetry; dispatch traces preserve either model-facing response items or code-mode JSON values (`codex-rs/tools/src/tool_output.rs:15-53`, `codex-rs/core/src/tools/registry.rs:543-567`, `codex-rs/core/src/tools/tool_dispatch_trace.rs:24-115`, `codex-rs/rollout-trace/src/tool_dispatch.rs:98-127`).
- Codex: result-projection tests cover MCP raw code-mode preservation, MCP model-facing truncation, image/content-item preservation, tool-search output, log preview, post-tool feedback replacement, and direct/code-mode trace payload preservation (`codex-rs/core/src/tools/context_tests.rs:48-86`, `codex-rs/core/src/tools/context_tests.rs:138-232`, `codex-rs/core/src/tools/context_tests.rs:234-272`, `codex-rs/core/src/tools/context_tests.rs:321-388`, `codex-rs/core/src/tools/registry_tests.rs:321-371`, `codex-rs/core/src/tools/tool_dispatch_trace_tests.rs:103-143`).

Shared invariant:
- Tool result handling is a multi-projection boundary. The raw runtime output, model-visible output, user/TUI display, persisted/history representation, hook payload, telemetry preview, and eval/trace payload must be treated as separate artifacts tied by the same tool-call ID, not as one reusable string.

Claude-specific design:
- Claude stores native `toolUseResult` beside the model-visible `tool_result` block in the user message; local UI and transcript rendering use the native result, while provider payload construction uses `message.content` (`src/services/tools/toolExecution.ts:1456-1466`, `src/components/messages/UserToolResultMessage/UserToolSuccessMessage.tsx:52-102`).
- Claude has a generic large-text-result persistence path that writes sidecar files and replaces model-visible content with a persisted-output preview wrapper (`src/utils/toolResultStorage.ts:137-198`, `src/utils/toolResultStorage.ts:272-333`).
- Claude repairs malformed tool-use/tool-result pairing and content-sibling layouts inside message normalization, including synthetic error tool results in non-strict mode (`src/utils/messages.ts:2522-2647`, `src/utils/messages.ts:5118-5458`).

Codex-specific design:
- Codex makes result projection a trait-level runtime contract: every tool output must be able to produce model output, hook output, telemetry preview, success metadata, and code-mode value when applicable (`codex-rs/tools/src/tool_output.rs:15-53`).
- Codex separates direct model response from code-mode response. Post-tool feedback can replace the model-visible response while `code_mode_result()` still returns the original typed result (`codex-rs/core/src/tools/registry.rs:185-205`, `codex-rs/core/src/tools/registry.rs:645-652`, `codex-rs/core/src/tools/registry_tests.rs:321-371`).
- Codex stores function/custom tool outputs in conversation history through `ResponseItem` and truncates those payloads by policy instead of using a generic sidecar persistence wrapper (`codex-rs/core/src/session/mod.rs:2658-2670`, `codex-rs/core/src/context_manager/history.rs:347-439`).

Gaps or unknowns:
- A Codex source path for generic sidecar persistence of arbitrary oversized tool results was not found in this mechanism's traced path; only history/rollout persistence and truncation are cited here.
- Claude's debug/eval evidence here is implementation telemetry and repair logic; no dedicated result-projection test file was located during this cycle.
- Codex display behavior is distributed across protocol events and TUI handlers; this card does not claim that every model-visible `ResponseInputItem` has a single generic TUI renderer.

Distilled harness rule:
- Define an explicit tool-result projection boundary. A completed tool must produce at least a raw/runtime value, a model-addressable value linked to the original call ID, a display value or display hook, and a persistence/history representation. Only share data across those surfaces through deliberate conversion functions, never by assuming that the string shown to the user is the string sent back to the model.

Anti-invention rule:
- Do not invent a generic "tool result format" unless the source proves it. Claude proves a per-tool `mapToolResultToToolResultBlockParam()` plus local `toolUseResult`; Codex proves a `ToolOutput` trait with `to_response_item()` and `code_mode_result()`. If a mechanism exists in only one source, label it source-specific: Claude's sidecar persisted-output wrapper is source-specific; Codex's direct/code-mode split is source-specific.

Verification pattern:
- Re-read the ledger before starting this mechanism.
- For Claude, trace `tool.call()` output to `ToolResult.data`, then to `mapToolResultToToolResultBlockParam()`, `processToolResultBlock()`/`processPreMappedToolResultBlock()`, `createUserMessage()`, `toolUseResult`, transcript storage, and UI rendering.
- For Codex, trace a handler return value to `ToolOutput`, then to `AnyToolResult`, `into_response()` or `code_mode_result()`, `ResponseInputItem -> ResponseItem`, history recording, display events, and dispatch trace payloads.
- Verify model-visible, runtime, persisted, display, telemetry, and eval surfaces independently. A citation for one surface does not prove another surface.

Skill text candidate:
- When studying a coding-agent harness, make a result-projection table before copying any tool pattern: raw output, model output, display output, persisted/history output, hook output, telemetry/debug output, and eval/trace output.
- Require exact source evidence for each conversion edge. In Claude, the core edge is native `toolUseResult` to Anthropic `tool_result`; in Codex, the core edge is `ToolOutput` to `ResponseInputItem` or code-mode JSON.
- Treat large-output handling as source-specific until proven otherwise. Claude persists oversized text to sidecar files and shows the model a preview wrapper; Codex's cited path records/truncates function output in history.
- Keep user display separate from model continuation. Claude renders from `toolUseResult`; Codex displays through protocol events and turn items. Neither source supports blindly rendering or reusing the model payload as the UI contract.
