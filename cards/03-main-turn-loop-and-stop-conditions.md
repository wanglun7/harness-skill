# Mechanism 3: Main turn loop and stop conditions

Mechanism: Main turn loop and stop conditions

Status: complete

Claude source:
- The public query surface is an async generator that yields stream/request/message/control messages and returns a `Terminal` result (`src/query.ts:219-238`). Its `QueryParams` include the starting messages, system/user/system contexts, tool permission callback, tool context, fallback model, query source, `maxTurns`, and optional task budget/deps (`src/query.ts:181-199`).
- The loop state is explicit and mutable across iterations: messages, tool context, auto-compact tracking, max-output recovery count, reactive compact guard, output-token override, pending tool summary, stop-hook activity, `turnCount`, and a test-visible `transition` (`src/query.ts:201-217`, `src/query.ts:265-280`).
- `queryLoop()` is a `while (true)` loop that destructures state at the top of each iteration (`src/query.ts:241-321`).
- Claude does not use `stop_reason === 'tool_use'` as the turn continuation signal; it records tool-use blocks during streaming and uses `needsFollowUp` as the loop-exit signal (`src/query.ts:554-558`).
- Before each model call, the loop computes the runtime main-loop model and passes messages, full system prompt, thinking config, tools, abort signal, model, fallback model, query source, agent definitions, MCP state, tracking, output-token override, and task budget into `deps.callModel()` (`src/query.ts:570-579`, `src/query.ts:657-705`).
- If aborted after streaming, the loop drains or synthesizes matching tool results, optionally yields a user interruption message, and returns `aborted_streaming` (`src/query.ts:1011-1051`).
- If `needsFollowUp` is false, the loop tries terminal recovery paths before completion: context-collapse drain, reactive compaction, max-output-token escalation/recovery, API-error terminal handling, stop hooks, and token-budget continuation (`src/query.ts:1062-1357`).
- Stop hooks return only two control fields: `blockingErrors` and `preventContinuation` (`src/query/stopHooks.ts:60-80`). Blocking errors are model-visible meta user messages that continue the loop; prevent-continuation stops the turn (`src/query/stopHooks.ts:257-294`, `src/query/stopHooks.ts:325-455`).
- If `needsFollowUp` is true, the loop runs or drains tools, yields tool updates, normalizes tool-result messages into next-iteration user messages, and may update the tool context (`src/query.ts:1366-1408`).
- If aborted during tool execution, it yields an interruption message unless this is a submit-interrupt path and returns `aborted_tools`; it also emits `max_turns_reached` if the abort crosses the turn cap (`src/query.ts:1484-1515`).
- A hook-stopped continuation returns `hook_stopped` (`src/query.ts:1518-1520`).
- Before recursive continuation, the loop increments `turnCount`, emits a `max_turns_reached` attachment and returns `max_turns` if the cap is exceeded, otherwise writes the next `State` with messages plus assistant/tool results and `transition: { reason: 'next_turn' }` (`src/query.ts:1678-1728`).
- `QueryEngine.submitMessage()` consumes `query(...)` as the SDK runtime loop and passes `maxTurns`, `taskBudget`, and `querySource: 'sdk'` (`src/QueryEngine.ts:675-686`). It increments SDK `turnCount` when user/tool-result messages appear, captures assistant/provider `stop_reason`, maps max-turn attachments to `error_max_turns`, maps API retry systems to SDK retry events, maps USD budget exhaustion to `error_max_budget_usd`, and finally yields either `error_during_execution` or `success` result events (`src/QueryEngine.ts:753-809`, `src/QueryEngine.ts:837-875`, `src/QueryEngine.ts:943-955`, `src/QueryEngine.ts:971-1002`, `src/QueryEngine.ts:1051-1155`).

Codex source:
- A regular Codex turn is driven by `run_turn(...)`, which returns `Option<String>` for the last agent message and receives session, turn context, turn extension data, initial input, an optional prewarmed `ModelClientSession`, and a cancellation token (`codex-rs/core/src/session/turn.rs:140-147`).
- At turn start, `run_turn()` creates or reuses the turn-scoped model client session, may run pre-sampling compaction, records context updates, builds skills/plugins, runs hooks, records inputs, merges connector selection, stores previous-turn settings, and records injected items (`codex-rs/core/src/session/turn.rs:148-189`).
- The outer turn loop is an explicit `loop`. It drains pending input only when allowed, records newly drained inputs, builds sampling request input from cloned history, creates Responses metadata, then calls `run_sampling_request(...)` (`codex-rs/core/src/session/turn.rs:200-247`).
- `SamplingRequestResult` has only `needs_follow_up` and `last_agent_message` (`codex-rs/core/src/session/turn.rs:1283-1287`).
- After sampling, Codex combines model/tool follow-up with pending user input: `needs_follow_up = model_needs_follow_up || has_pending_input`. If a new context window starts or auto-compaction runs while follow-up is needed, the loop continues with controlled pending-input drain behavior (`codex-rs/core/src/session/turn.rs:249-323`).
- If no follow-up is needed, Codex runs turn stop hooks. A blocking stop hook can record a prompt message and continue the loop; a stop request breaks; otherwise legacy after-agent hooks may return `None`, and normal completion breaks (`codex-rs/core/src/session/turn.rs:325-380`).
- `CodexErr::TurnAborted` exits the loop without error emission from `run_turn()` because abort is reported by task lifecycle; invalid-image and other errors emit turn error lifecycle and user-visible error events, then break (`codex-rs/core/src/session/turn.rs:382-414`).
- `try_run_sampling_request()` opens the model stream through `client_session.stream(...)`, initializes in-flight tool futures, `needs_follow_up`, last-agent-message state, active streamed item state, trace spans, and plan-mode parsers (`codex-rs/core/src/session/turn.rs:1849-1905`).
- The stream receive loop treats cancellation as `CodexErr::TurnAborted`, stream errors as errors, and missing `response.completed` as a stream error (`codex-rs/core/src/session/turn.rs:1915-1949`).
- On completed output items, Codex handles the item, may enqueue a tool future, may update the last agent message, ORs item-level `needs_follow_up` into the sampling result, and can preempt for mailbox mail with `needs_follow_up: true` (`codex-rs/core/src/session/turn.rs:1956-2049`).
- On `ResponseEvent::Completed`, Codex flushes assistant text segments, records token usage, asks to emit token count and turn diff, sets follow-up when `end_turn` is explicitly false, and returns `SamplingRequestResult` (`codex-rs/core/src/session/turn.rs:2168-2184`).
- After the stream loop, Codex drains all in-flight tool futures, emits token count only after blocking tools resolve, turns late cancellation into `TurnAborted`, optionally emits a turn diff, then returns the sampling outcome (`codex-rs/core/src/session/turn.rs:2310-2336`).
- Turns are hosted as session tasks. `SessionTask::run()` is documented to execute until completion or cancellation and return an optional final agent message for `Session::on_task_finished()` (`codex-rs/core/src/tasks/mod.rs:210-232`).
- `Session::spawn_task()` first aborts existing tasks with `TurnAbortReason::Replaced`, clears connector selection, and delegates to `start_task()` (`codex-rs/core/src/tasks/mod.rs:308-317`). `start_task()` marks turn start, creates a cancellation token, emits turn-start lifecycle, starts a Tokio task span, runs the task, flushes rollout before completion, and calls `on_task_finished()` if not cancelled (`codex-rs/core/src/tasks/mod.rs:319-425`).
- `abort_all_tasks()`/`abort_turn_if_active()` cancel active tasks and emit abort lifecycle (`codex-rs/core/src/tasks/mod.rs:486-519`). `handle_task_abort()` cancels the task token, waits briefly, aborts the handle, runs task-specific abort cleanup, records and flushes an interrupted-turn marker for interrupted turns, and sends `TurnAborted` (`codex-rs/core/src/tasks/mod.rs:800-868`).
- Normal task completion emits `TurnComplete` with turn id, last agent message, completion timestamp, duration, and time-to-first-token, then clears the active turn and may emit thread-idle lifecycle (`codex-rs/core/src/tasks/mod.rs:731-766`).
- Protocol terminal events are structured: `TurnCompleteEvent` carries `turn_id`, optional `last_agent_message`, `completed_at`, `duration_ms`, and `time_to_first_token_ms`; `TurnAbortedEvent` carries optional `turn_id`, `reason`, `completed_at`, and `duration_ms`; abort reasons are `Interrupted`, `Replaced`, `ReviewEnded`, and `BudgetLimited` (`codex-rs/protocol/src/protocol.rs:1933-1948`, `codex-rs/protocol/src/protocol.rs:3864-3884`).
- `Session::send_event()` records terminal and nonterminal events to rollout trace/tool-call trace, sends the event to clients, notifies parent agents for terminal child turns, mirrors text to realtime, and clears realtime handoff on completion (`codex-rs/core/src/session/mod.rs:1656-1688`). `send_event_raw()` persists events as `RolloutItem::EventMsg` before broadcasting (`codex-rs/core/src/session/mod.rs:1831-1835`).
- Conversation items are separately appended to history, persisted to rollout, and sent to raw response observers (`codex-rs/core/src/session/mod.rs:2626-2670`). `flush_rollout()` is the durability barrier for live thread writes (`codex-rs/core/src/session/mod.rs:1106-1114`).
- Agent status is derived from lifecycle events: `TurnStarted` means running, `TurnComplete` means completed, interrupted/budget-limited aborts mean interrupted, other aborts mean errored (`codex-rs/core/src/agent/status.rs:4-20`).

Runtime trace evidence:
- Claude: `QueryEngine.submitMessage()` -> `query(...)` -> `queryLoop()` `while (true)` -> `deps.callModel(...)` streaming -> if streamed output sets `needsFollowUp`, run tools and continue with assistant plus tool-result messages; if no follow-up, run recovery/stop-hook/token-budget checks and return a `Terminal` reason. Evidence: `src/QueryEngine.ts:675-686`, `src/query.ts:219-321`, `src/query.ts:554-558`, `src/query.ts:657-705`, `src/query.ts:1062-1357`, `src/query.ts:1366-1728`.
- Codex: `Session::spawn_task()` / `start_task()` -> task `run()` -> `run_turn()` `loop` -> `run_sampling_request()` / `try_run_sampling_request()` stream loop -> `SamplingRequestResult { needs_follow_up, last_agent_message }` -> continue, compact, stop-hook retry, break, or error -> task lifecycle emits `TurnComplete` or `TurnAborted`. Evidence: `codex-rs/core/src/tasks/mod.rs:308-425`, `codex-rs/core/src/session/turn.rs:140-414`, `codex-rs/core/src/session/turn.rs:1849-2336`, `codex-rs/core/src/tasks/mod.rs:731-868`.

What the model sees:
- Claude: on normal follow-up, the next model request includes prior messages plus assistant output and normalized tool-result user messages (`src/query.ts:1366-1408`, `src/query.ts:1714-1728`). It may also see meta user messages for max-output-token recovery, stop-hook blocking, and token-budget continuation (`src/query.ts:1223-1251`, `src/query.ts:1282-1305`, `src/query.ts:1308-1340`). It does not see `max_turns_reached` after terminal exit as a new model input.
- Codex: the next sampling request is built from recorded conversation history via `clone_history().for_prompt(...)` each loop (`codex-rs/core/src/session/turn.rs:221-228`). Follow-up-causing tool outputs and hook prompt messages are recorded into conversation items before subsequent sampling (`codex-rs/core/src/session/mod.rs:2626-2670`, `codex-rs/core/src/session/turn.rs:325-345`). Turn lifecycle events such as `TurnComplete` and `TurnAborted` are protocol/display/persistence events, not prompt items by this mechanism.

What the runtime does:
- Claude: owns continuation inside the `queryLoop()` generator. It uses source-level state transitions, yields intermediate stream/tool/control messages, returns terminal reasons, and lets `QueryEngine` map the yielded stream into SDK results.
- Codex: owns continuation inside `run_turn()` but hosts the turn inside a cancellable `SessionTask`. Sampling returns a compact `SamplingRequestResult`; task lifecycle, not `run_turn()` itself, emits user/client terminal events.

What gets persisted:
- Claude: `QueryEngine` records transcript messages during iteration and flushes session storage before result emission on max-turn, budget, and final-result paths when persistence is enabled (`src/QueryEngine.ts:720-731`, `src/QueryEngine.ts:837-850`, `src/QueryEngine.ts:971-980`, `src/QueryEngine.ts:1070-1080`).
- Codex: `send_event_raw()` persists emitted events as `RolloutItem::EventMsg` (`codex-rs/core/src/session/mod.rs:1831-1835`); `record_conversation_items()` appends prompt/response items to history and persists response items (`codex-rs/core/src/session/mod.rs:2626-2670`); task completion flushes rollout before emitting completion (`codex-rs/core/src/tasks/mod.rs:395-423`); abort records and flushes an interrupted-turn marker before `TurnAborted` for interrupted turns (`codex-rs/core/src/tasks/mod.rs:831-848`).

What the user/TUI sees:
- Claude: the stream yields assistant messages, tool progress/results, interruption messages, hook progress/attachments, and final SDK result events. Max turns surface as `error_max_turns`; budget exhaustion surfaces as `error_max_budget_usd`; API retries surface as `api_retry`; final execution maps to success or error result (`src/QueryEngine.ts:837-875`, `src/QueryEngine.ts:943-955`, `src/QueryEngine.ts:971-1002`, `src/QueryEngine.ts:1082-1155`).
- Codex: the protocol sends started/item/delta/diff/token/error/terminal events. The terminal user/client status is `TurnComplete` or `TurnAborted`; status tracking derives running/completed/interrupted/errored from those events (`codex-rs/core/src/tasks/mod.rs:731-868`, `codex-rs/protocol/src/protocol.rs:1933-1948`, `codex-rs/protocol/src/protocol.rs:3864-3884`, `codex-rs/core/src/agent/status.rs:4-20`).

What debug/eval surfaces exist:
- Claude: `query.ts` stores `transition` on loop state so tests can assert recovery paths without inspecting message contents (`src/query.ts:213-216`). It emits query checkpoints around setup/streaming/recursive-call and logs events for streaming tool execution, max-token escalation, token-budget completion, and stop-hook cancellation/errors (`src/query.ts:560-568`, `src/query.ts:657-659`, `src/query.ts:1204-1206`, `src/query.ts:1316-1354`, `src/query.ts:1714-1725`, `src/query/stopHooks.ts:282-294`, `src/query/stopHooks.ts:456-472`).
- Codex: `run_turn()` and `try_run_sampling_request()` are trace-instrumented, including turn id, model, stream request, receive loop, token status, and request reasoning fields (`codex-rs/core/src/session/turn.rs:269-284`, `codex-rs/core/src/session/turn.rs:1849-1895`, `codex-rs/core/src/session/turn.rs:1915-1928`). Task spans include thread id, turn id, model, reasoning effort, and token usage fields (`codex-rs/core/src/tasks/mod.rs:378-394`). Rollout reconstruction treats `TurnComplete` and `TurnAborted` as persisted turn boundaries (`codex-rs/core/src/session/rollout_reconstruction.rs:145-154`).

Shared invariant:
- Both references make continuation an explicit runtime decision, not an implicit provider stop-reason assumption. A turn continues only when the runtime has model/tool/pending-input/recovery/hook work to feed back into the next sampling request; otherwise it exits through a structured terminal path.

Claude-specific design:
- The main loop is an async generator that directly yields model/tool/control messages and returns a `Terminal` reason.
- `needsFollowUp` is set from observed tool-use blocks and deliberately replaces `stop_reason === 'tool_use'` as the loop-exit signal.
- Stop-hook blocking, token-budget continuation, and max-output recovery are implemented as meta user messages inserted into the next model request.
- Max-turn enforcement is a generator-level terminal branch that yields a `max_turns_reached` attachment for the SDK layer to convert into `error_max_turns`.

Codex-specific design:
- The main loop is inside `run_turn()`, but user/client terminal signaling is owned by the session-task lifecycle.
- Sampling continuation is summarized as `SamplingRequestResult { needs_follow_up, last_agent_message }`.
- Pending user input can be folded into `needs_follow_up`, so a turn can continue even when the model itself did not require tool follow-up.
- Abort reasons are protocol enums, and replacement of an active task is represented as `TurnAbortReason::Replaced`.

Gaps or unknowns:
- Claude imports the `Terminal`/`Continue` types from `./query/transitions.js`, but that source file was not present in the inspected `src/query` snapshot. This card therefore cites concrete return sites and state transitions in `query.ts` rather than a missing type definition file.
- Codex stop-hook internals are only cited at the outer loop behavior here; detailed hook behavior belongs to later mechanisms where hooks/instructions/permissions are examined.
- Detailed streaming text/tool interleaving is intentionally deferred to mechanism 4.
- Detailed message normalization and provider payload construction are intentionally deferred to mechanism 5.

Distilled harness rule:
- Implement the main turn loop as a source-visible state machine with explicit continuation and terminal branches. Cite the loop entry, carried state, model sampling call, follow-up signal, tool/result feedback path, recovery continuations, stop-hook behavior, max-turn/abort/error exits, persistence boundary, and user/client terminal event mapping separately.

Anti-invention rule:
- Do not claim a generic stop condition such as "provider stop reason controls the loop" unless source proves it. Claude explicitly rejects `stop_reason === 'tool_use'` as reliable; Codex collapses stream/tool/pending-input state into `SamplingRequestResult.needs_follow_up`. Preserve those source-specific mechanisms.

Verification pattern:
- Find the public turn/query entrypoint.
- Find the loop state and loop construct.
- Find the sampling/model-call boundary.
- Find the runtime follow-up signal and every source path that sets it.
- Find the tool-result or prompt-item feedback path into the next iteration.
- Find no-follow-up handling: recovery, stop hooks, budget checks, and completion.
- Find abort/cancellation handling during stream and tool phases.
- Find max-turn or replacement behavior if present.
- Find persistence and user/client terminal event mapping.
- Mark any missing type definitions or deferred submechanisms as gaps instead of filling them in.

Skill text candidate:
- For main turn loop analysis, never summarize from behavior alone. Trace one turn from its public entrypoint into the loop, then enumerate exact continuation and terminal branches. Separate "model sees next prompt items" from "runtime emits events" and from "persistence records transcript/rollout." When comparing harnesses, treat Claude's generator-return terminal reasons and Codex's task-lifecycle terminal events as different designs with a shared invariant: explicit runtime-controlled continuation.
