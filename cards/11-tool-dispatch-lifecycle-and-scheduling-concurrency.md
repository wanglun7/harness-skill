# Mechanism 11: Tool Dispatch Lifecycle and Scheduling/Concurrency

Mechanism: Tool dispatch lifecycle and scheduling/concurrency
Status: complete

Claude source:
- `src/query.ts:826-844` collects assistant `tool_use` blocks, marks follow-up needed, and feeds streamed tool blocks into `StreamingToolExecutor.addTool()`.
- `src/query.ts:847-862` polls `StreamingToolExecutor.getCompletedResults()` during model streaming and appends normalized user tool results.
- `src/query.ts:1366-1408` chooses streamed executor results or fallback `runTools()`, yields messages, records normalized user tool results, and carries updated `ToolUseContext`.
- `src/services/tools/toolOrchestration.ts:19-82` partitions and dispatches fallback tool execution batches.
- `src/services/tools/toolOrchestration.ts:91-116` groups consecutive concurrency-safe tool calls and makes unsafe calls single-item batches.
- `src/services/tools/toolOrchestration.ts:118-187` runs serial and concurrent batches, sets in-progress tool IDs, applies context modifiers, and caps concurrent execution through `all(..., getMaxToolUseConcurrency())`.
- `src/services/tools/StreamingToolExecutor.ts:19-40` defines queued/executing/completed/yielded states and the order-preserving concurrency contract.
- `src/services/tools/StreamingToolExecutor.ts:73-151` adds streamed tool calls, computes `isConcurrencySafe`, and starts queued tools when concurrency conditions allow.
- `src/services/tools/StreamingToolExecutor.ts:210-240` derives abort reason and per-tool interrupt behavior.
- `src/services/tools/StreamingToolExecutor.ts:265-325` marks a tool executing, creates a child abort controller, and invokes `runToolUse()`.
- `src/services/tools/StreamingToolExecutor.ts:332-405` collects tool messages/progress, cascades Bash sibling aborts, applies non-concurrent context modifiers, and resumes queue processing.
- `src/services/tools/StreamingToolExecutor.ts:412-490` yields progress immediately, yields completed results in order, blocks behind unsafe executing tools, and drains remaining tools.
- `src/services/tools/toolExecution.ts:337-490` resolves a tool by visible name/alias, handles unknown/aborted calls, and streams permission/call updates.
- `src/services/tools/toolExecution.ts:599-733` parses schema input, runs optional `validateInput()`, and returns model-visible error tool results on invalid input.
- `src/services/tools/toolExecution.ts:800-862` runs `PreToolUse` hooks, forwarding progress and stopping early when hooks block.
- `src/services/tools/toolExecution.ts:916-1104` resolves permission decisions, records allow/reject telemetry, returns rejection tool results, and may run permission-denied hooks.
- `src/services/tools/toolExecution.ts:1171-1222` starts execution tracing/session activity and calls `tool.call()` with context, permission metadata, `canUseTool`, parent assistant message, and progress callback.
- `src/services/tools/toolExecution.ts:1397-1473` attaches post-tool handling and returns a context modifier with the tool result message.
- `src/Tool.ts:379-405` defines `call()`, `inputSchema`, `isConcurrencySafe()`, and read-only/destructive metadata.
- `src/Tool.ts:407-416` defines per-tool interrupt behavior.
- `src/Tool.ts:483-503` defines `validateInput()` and `checkPermissions()`.

Codex source:
- `codex-rs/core/src/session/turn.rs:1016-1027` builds prompts with `parallel_tool_calls` from model capability.
- `codex-rs/core/src/session/turn.rs:1056-1070` builds the `ToolRouter` and `ToolCallRuntime` for a sampling request.
- `codex-rs/core/src/session/turn.rs:1896-1904` prepares ordered in-flight tool futures and active tool-argument diff consumers.
- `codex-rs/core/src/session/turn.rs:1958-2043` handles completed response items, queues tool futures, and tracks follow-up need.
- `codex-rs/core/src/session/turn.rs:2218-2234` consumes streamed tool-call input deltas through a tool-specific diff consumer.
- `codex-rs/core/src/session/turn.rs:2305-2311` waits for in-flight tools after the response stream, with tool-blocking timing.
- `codex-rs/core/src/session/turn.rs:1822-1847` drains ordered tool futures and records each tool response item into conversation history.
- `codex-rs/core/src/stream_events_utils.rs:303-314` defines completed-item handling as immediate history sync plus optional queued tool future.
- `codex-rs/core/src/stream_events_utils.rs:405-444` converts model output items into `ToolCall`, records the call item immediately, and queues `ToolCallRuntime.handle_tool_call()`.
- `codex-rs/core/src/stream_events_utils.rs:486-507` records direct model-facing error responses for nonfatal tool-call parse errors.
- `codex-rs/core/src/tools/router.rs:100-110` asks the registry whether a tool supports parallel calls or waits for runtime cancellation.
- `codex-rs/core/src/tools/router.rs:112-160` builds a `ToolCall` from function, client `tool_search`, or custom tool-call response items.
- `codex-rs/core/src/tools/router.rs:185-239` wraps a `ToolCall` into `ToolInvocation` and dispatches through the registry.
- `codex-rs/core/src/tools/parallel.rs:30-52` stores router/session/turn/diff-tracker state plus a shared `RwLock` scheduler.
- `codex-rs/core/src/tools/parallel.rs:62-79` maps tool-dispatch failures into either fatal errors or model-visible failure responses.
- `codex-rs/core/src/tools/parallel.rs:81-178` spawns tool dispatch tasks, uses read locks for parallel-safe calls and write locks for exclusive calls, handles cancellation, and emits aborted lifecycle notifications.
- `codex-rs/core/src/tools/parallel.rs:213-234` constructs model-visible abort output for cancelled calls.
- `codex-rs/core/src/tools/registry.rs:380-388` exposes per-tool parallel and cancellation behavior.
- `codex-rs/core/src/tools/registry.rs:403-439` increments active-turn tool-call accounting and starts a dispatch trace.
- `codex-rs/core/src/tools/registry.rs:440-489` rejects unsupported tools or incompatible payloads with telemetry and trace failures.
- `codex-rs/core/src/tools/registry.rs:491-537` emits start lifecycle notifications and runs pre-tool hooks that may block or update invocation input.
- `codex-rs/core/src/tools/registry.rs:539-568` runs the handler under telemetry result logging.
- `codex-rs/core/src/tools/registry.rs:573-604` runs post-tool hooks and records additional contexts.
- `codex-rs/core/src/tools/registry.rs:606-667` emits finish lifecycle, applies post-tool blocking/feedback, and records completed or failed dispatch trace.
- `codex-rs/core/src/tools/registry.rs:684-708` invokes the concrete runtime handler and packages `AnyToolResult`.
- `codex-rs/core/src/tools/lifecycle.rs:12-31` notifies extension lifecycle contributors on tool start.
- `codex-rs/core/src/tools/lifecycle.rs:33-85` notifies lifecycle finish/aborted outcomes.
- `codex-rs/core/src/tools/tool_dispatch_trace.rs:24-60` starts, completes, and fails dispatch traces.
- `codex-rs/core/src/tools/tool_dispatch_trace.rs:62-115` maps direct vs code-mode dispatch requests and payload/result forms into rollout-trace records.

Runtime trace evidence:
- Claude stream path: provider assistant message -> `query.ts` extracts `tool_use` blocks -> `StreamingToolExecutor.addTool()` -> `runToolUse()` -> `streamingToolExecutor.getCompletedResults()` during streaming -> `getRemainingResults()` after streaming -> normalized user `tool_result` messages feed the next turn (`src/query.ts:826-862`, `src/query.ts:1366-1408`).
- Claude fallback path: collected `toolUseBlocks` -> `runTools()` -> `partitionToolCalls()` -> serial/concurrent batch execution -> yielded `MessageUpdate` with `newContext` (`src/services/tools/toolOrchestration.ts:19-82`, `src/services/tools/toolOrchestration.ts:91-187`).
- Codex path: `ResponseEvent::OutputItemDone` -> `handle_output_item_done()` -> `ToolRouter::build_tool_call()` -> `ToolCallRuntime.handle_tool_call()` future -> `ToolRouter.dispatch_tool_call_with_terminal_outcome()` -> `ToolRegistry.dispatch_any_with_terminal_outcome()` -> `drain_in_flight()` records output (`codex-rs/core/src/session/turn.rs:1958-2043`, `codex-rs/core/src/stream_events_utils.rs:405-444`, `codex-rs/core/src/tools/router.rs:112-239`, `codex-rs/core/src/tools/parallel.rs:81-178`, `codex-rs/core/src/tools/registry.rs:403-667`, `codex-rs/core/src/session/turn.rs:1822-1847`).

What the model sees:
- Claude: the model sees `tool_result` user messages produced by `runToolUse()` for unknown tools, validation failures, permission denials, aborts, and successful calls; dispatch itself also preserves assistant `tool_use` IDs so results are tied back to the emitted tool call (`src/services/tools/toolExecution.ts:337-490`, `src/services/tools/toolExecution.ts:599-733`, `src/services/tools/toolExecution.ts:995-1104`, `src/services/tools/toolExecution.ts:1456-1473`).
- Codex: the model sees `ResponseInputItem` outputs produced from tool futures; failures become function/custom/tool-search outputs through `ToolCallRuntime.failure_response()`, and cancellation becomes aborted output (`codex-rs/core/src/tools/parallel.rs:62-79`, `codex-rs/core/src/tools/parallel.rs:186-234`, `codex-rs/core/src/session/turn.rs:1828-1833`).

What the runtime does:
- Claude: dispatch validates the input schema, runs tool-specific validation, starts pre-tool hooks, resolves permission, invokes `tool.call()`, streams progress, runs post-tool hooks, and returns context modifiers (`src/services/tools/toolExecution.ts:599-733`, `src/services/tools/toolExecution.ts:800-862`, `src/services/tools/toolExecution.ts:916-1222`, `src/services/tools/toolExecution.ts:1397-1473`).
- Claude scheduling: fallback dispatch batches consecutive concurrency-safe tools and serializes unsafe tools; streaming dispatch starts tools as they arrive, allows overlapping safe tools, blocks unsafe tools behind any executing tool, and caps fallback concurrent batches with `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` defaulting to 10 (`src/services/tools/toolOrchestration.ts:8-11`, `src/services/tools/toolOrchestration.ts:91-187`, `src/services/tools/StreamingToolExecutor.ts:73-151`).
- Codex: dispatch parses response items into `ToolCall`, wraps them in `ToolInvocation`, enters registry dispatch, runs lifecycle notifications/hooks/handler/post-hooks, and records the result or error path (`codex-rs/core/src/tools/router.rs:112-239`, `codex-rs/core/src/tools/registry.rs:403-667`).
- Codex scheduling: each tool dispatch is spawned as a task; parallel-safe tools acquire a read lock, exclusive tools acquire a write lock, so multiple safe tools may overlap while unsafe tools serialize with all others (`codex-rs/core/src/tools/parallel.rs:81-133`).

What gets persisted:
- Claude: the central dispatch layer yields `Message` objects and `query.ts` normalizes yielded tool messages into `toolResults` for the next API call; this card does not claim a storage backend from the dispatch code alone (`src/query.ts:847-862`, `src/query.ts:1384-1408`).
- Codex: completed model tool-call items are recorded immediately before execution; drained tool response items are recorded into conversation history after in-flight execution resolves (`codex-rs/core/src/stream_events_utils.rs:432-433`, `codex-rs/core/src/session/turn.rs:1828-1833`).

What the user/TUI sees:
- Claude: the dispatcher updates in-progress tool IDs and interruptible-tool state, emits progress messages immediately, and returns synthetic rejection/cancellation/error tool results for user-interrupt or sibling-error cases (`src/services/tools/toolOrchestration.ts:126-148`, `src/services/tools/StreamingToolExecutor.ts:254-270`, `src/services/tools/StreamingToolExecutor.ts:332-386`, `src/services/tools/StreamingToolExecutor.ts:412-490`).
- Codex: central dispatch emits tool argument diff events for tools with diff consumers and lifecycle notifications for extension contributors; non-tool item start/complete rendering stays outside dispatch, while tool-specific display events are handler-specific (`codex-rs/core/src/session/turn.rs:2218-2234`, `codex-rs/core/src/tools/lifecycle.rs:12-85`, `codex-rs/core/src/stream_events_utils.rs:445-482`).

What debug/eval surfaces exist:
- Claude: dispatch logs debug strings for unknown tools, validation failures, slow hooks, slow permission decisions, permission rejections, and success/error telemetry/OTel events (`src/services/tools/toolExecution.ts:368-395`, `src/services/tools/toolExecution.ts:632-663`, `src/services/tools/toolExecution.ts:865-869`, `src/services/tools/toolExecution.ts:937-966`, `src/services/tools/toolExecution.ts:1331-1395`).
- Claude: no `runTools`/`StreamingToolExecutor` test files were found by `rg -n "runTools|StreamingToolExecutor|isConcurrencySafe|toolOrchestration" src -g '*test*' -g '*spec*'` in the referenced source tree during this cycle.
- Codex: dispatch spans are annotated with `#[instrument]`; registry dispatch starts `ToolDispatchTrace`, telemetry wraps handler execution, lifecycle behavior has direct tests, cancellation behavior has direct tests, and rollout dispatch tracing has tests for direct/code-mode, unsupported, and incompatible payload paths (`codex-rs/core/src/tools/router.rs:112-239`, `codex-rs/core/src/tools/parallel.rs:62-178`, `codex-rs/core/src/tools/registry.rs:403-667`, `codex-rs/core/src/tools/registry_tests.rs:373-452`, `codex-rs/core/src/tools/parallel.rs:404-544`, `codex-rs/core/src/tools/tool_dispatch_trace_tests.rs:61-123`, `codex-rs/core/src/tools/tool_dispatch_trace_tests.rs:149-210`).

Shared invariant:
- A model-emitted tool call is never executed by name alone. Both harnesses first convert the model item into an internal dispatch object, validate/route it against the current tool registry, run guard hooks/permission gates where defined, schedule it according to per-tool concurrency metadata, and return a model-addressable result tied to the original call ID.

Claude-specific design:
- Claude has two execution modes for the same tool lifecycle: streamed dispatch can begin tool execution while the model stream is still arriving, and fallback dispatch runs collected tool-use blocks after streaming. Both share `runToolUse()` for validation/permission/call behavior (`src/query.ts:826-862`, `src/query.ts:1366-1383`, `src/services/tools/toolExecution.ts:337-490`).
- Claude concurrency is data-dependent per tool input: `isConcurrencySafe(input)` is called after schema parse, consecutive safe calls are batched, unsafe calls are single-item batches, and streamed safe calls can overlap if no unsafe execution blocks them (`src/services/tools/toolOrchestration.ts:91-116`, `src/services/tools/StreamingToolExecutor.ts:104-151`).
- Claude synthetic cancellation is source-specific: a Bash error aborts sibling subprocesses, while other tool failures do not cascade (`src/services/tools/StreamingToolExecutor.ts:354-363`).

Codex-specific design:
- Codex separates parsing/routing, scheduling, and registry dispatch into `ToolRouter`, `ToolCallRuntime`, and `ToolRegistry` respectively (`codex-rs/core/src/tools/router.rs:112-239`, `codex-rs/core/src/tools/parallel.rs:30-178`, `codex-rs/core/src/tools/registry.rs:403-667`).
- Codex uses an `RwLock` scheduler: parallel-safe calls use read access; exclusive calls use write access (`codex-rs/core/src/tools/parallel.rs:113-133`).
- Codex records model tool-call items immediately and drains tool outputs after streaming completes, preserving ordered tool futures with `FuturesOrdered` (`codex-rs/core/src/session/turn.rs:1896-1897`, `codex-rs/core/src/stream_events_utils.rs:432-443`, `codex-rs/core/src/session/turn.rs:2305-2311`).
- Codex exposes lifecycle contributors and rollout dispatch traces as first-class dispatch surfaces (`codex-rs/core/src/tools/lifecycle.rs:12-85`, `codex-rs/core/src/tools/tool_dispatch_trace.rs:24-115`).

Gaps or unknowns:
- Claude dispatch tests were not located in the referenced source tree during this cycle; the mechanism card therefore relies on implementation citations rather than test citations for Claude.
- Tool-specific display behavior is intentionally not generalized here; later mechanism cards cover file/shell/web tools and result projection.
- Permission policy taxonomy and sandbox behavior are dispatch gates here but belong to later permission/sandbox mechanisms.

Distilled harness rule:
- Build a three-layer dispatch path: model item -> internal tool call/invocation -> scheduler -> registry/runtime handler. The scheduler must use explicit per-tool concurrency metadata, and the registry/runtime handler must be the only place that runs validation, hooks, permission gates, lifecycle events, handler execution, post-hooks, telemetry, and model-addressable result construction.

Anti-invention rule:
- Do not add generic "parallel tools" or "tool lifecycle" abstractions unless the reference source shows the exact scheduling primitive and lifecycle surface. If the mechanism appears only in one source, label it source-specific: Claude's streamed executor and Bash sibling abort are Claude-specific; Codex's `RwLock` scheduler, lifecycle contributors, and rollout dispatch trace are Codex-specific.

Verification pattern:
- Re-read the current ledger before starting this mechanism.
- For Claude, trace `query.ts` from `tool_use` collection to either `StreamingToolExecutor` or `runTools()`, then trace both into `runToolUse()` and the `Tool` type fields.
- For Codex, trace `turn.rs` response handling into `stream_events_utils::handle_output_item_done()`, then `ToolRouter::build_tool_call()`, `ToolCallRuntime`, `ToolRegistry`, lifecycle, and dispatch trace.
- Verify concurrency claims only against `isConcurrencySafe()`/batching/`all()` in Claude and `supports_parallel_tool_calls()`/`RwLock` read-write scheduling in Codex.
- Verify display, persistence, and debug/eval claims separately; do not infer them from the existence of a tool call.

Skill text candidate:
- When studying or reusing tool dispatch, require exact source evidence for four things before writing a rule: how the model call is parsed, how it is scheduled, where the handler is invoked, and how completion/failure is returned to the turn loop.
- Keep dispatch lifecycle separate from result projection. Dispatch may create a model-addressable result object, but the exact raw/model/display/persisted output shape belongs in the result-projection mechanism.
- Treat concurrency as a registry/tool property, not a global promise pool. Claude proves this with input-sensitive `isConcurrencySafe()` and serial/safe batching; Codex proves it with registry `supports_parallel_tool_calls()` and `RwLock` scheduling.
- Treat lifecycle and trace hooks as runtime surfaces, not model-visible behavior. The model sees linked tool outputs; the runtime may additionally emit progress, lifecycle callbacks, telemetry, traces, and TUI events.
