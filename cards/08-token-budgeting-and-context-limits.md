# Mechanism 8: Token budgeting and context limits

Mechanism: Token budgeting and context limits

Status: complete

Claude source:
- `src/utils/context.ts:8-25` defines default context window and max-output-token constants.
- `src/utils/context.ts:31-98` resolves context-window size from disable gates, explicit `[1m]` model suffix, model capability `max_input_tokens`, beta headers, experiment state, ant-only model metadata, and a default 200k window.
- `src/utils/context.ts:118-144` computes context-window used/remaining percentages from the latest usage input/cache token counts.
- `src/utils/tokens.ts:39-66` derives a context-window token count from API response usage, including input, cache, and output tokens.
- `src/utils/tokens.ts:68-112` derives final context tokens from the last response for `task_budget.remaining`, preferring `usage.iterations[-1]` when present and excluding cache tokens.
- `src/utils/tokens.ts:114-136` warns that output-token-only counts are not valid for threshold checks.
- `src/utils/tokens.ts:201-260` defines `tokenCountWithEstimation()` as the canonical threshold count: last real API usage plus rough estimates for messages added since, with special handling for split parallel-tool-call assistant records.
- `src/services/compact/autoCompact.ts:28-49` computes an effective context window by subtracting reserved output space from model context window and honoring `CLAUDE_CODE_AUTO_COMPACT_WINDOW`.
- `src/services/compact/autoCompact.ts:62-145` defines warning/error/auto-compact/blocking buffers and returns `isAtBlockingLimit` from `calculateTokenWarningState()`.
- `src/query.ts:592-647` blocks the query with `PROMPT_TOO_LONG_ERROR_MESSAGE` when `tokenCountWithEstimation(messagesForQuery) - snipTokensFreed` reaches the hard blocking limit and automatic overflow recovery does not own the path.
- `src/services/api/errors.ts:62-77` defines and recognizes prompt-too-long API error messages.
- `src/services/api/claude.ts:2279-2292` converts provider `model_context_window_exceeded` stop reason into an API error message on the max-output-token recovery path.
- `src/utils/tokenBudget.ts:1-28` parses user-entered output-token budgets from shorthand/verbose text; `src/utils/tokenBudget.ts:66-73` builds the model-visible continuation nudge.
- `src/bootstrap/state.ts:724-743` stores per-turn output-token budget state and computes turn output tokens from cumulative output usage.
- `src/query/tokenBudget.ts:13-93` tracks budget continuation state and continues until 90% of budget, unless diminishing output deltas stop continuation.
- `src/query.ts:1308-1332` applies the token-budget decision by injecting a meta user nudge message and continuing the loop.
- `src/screens/REPL.tsx:2893-2896` snapshots a parsed token budget at turn start; `src/screens/REPL.tsx:2953-2968` captures and resets budget info at turn end.
- `src/components/Spinner.tsx:261-272` renders live budget progress text.
- `src/utils/attachments.ts:3828-3844` exposes output-token usage as an attachment when the token-budget feature is enabled.
- `src/services/api/claude.ts:470-501` defines `output_config.task_budget` request shape and beta header; `src/services/api/claude.ts:1571-1575` attaches it to provider requests.

Codex source:
- `codex-rs/protocol/src/openai_models.rs:342-398` defines model metadata fields for `context_window`, `max_context_window`, `auto_compact_token_limit`, and `effective_context_window_percent`.
- `codex-rs/protocol/src/openai_models.rs:428-443` resolves the model context window and derives/clamps the auto-compact token limit to 90% of the context window.
- `codex-rs/core/src/session/turn_context.rs:203-210` computes the effective model context window by multiplying resolved model context window by `effective_context_window_percent`.
- `codex-rs/codex-api/src/sse/responses.rs:110-134` parses completed response usage into `TokenUsage` with input, cached input, output, reasoning output, and total tokens.
- `codex-rs/core/src/client.rs:1823-1840` records completed-response token usage in session telemetry and inference trace.
- `codex-rs/core/src/session/turn.rs:2162-2177` records response token usage into session state when a completed event arrives and schedules a token-count event.
- `codex-rs/core/src/context_manager/history.rs:249-258` appends new usage into `TokenUsageInfo`; `codex-rs/core/src/context_manager/history.rs:294-314` computes active context tokens from last usage plus locally added items and non-last reasoning items when needed.
- `codex-rs/core/src/context_manager/history.rs:130-150` estimates token count with byte-based heuristics from base instructions and history items.
- `codex-rs/core/src/session/mod.rs:1155-1194` exposes total usage, full token info, and estimated token count from session state.
- `codex-rs/core/src/session/turn.rs:736-796` defines `AutoCompactTokenStatus`, active context tokens, scoped token count, scoped limit, full context window limit, and the combined `token_limit_reached` predicate.
- `codex-rs/core/src/session/turn.rs:798-815` checks the token-limit status before sampling and triggers compaction if the configured budget or usable window is exhausted.
- `codex-rs/core/src/session/turn.rs:230-322` compares tokens before/after sampling, logs post-sampling token status, records remaining token-budget context, and triggers mid-turn compaction when a follow-up is needed and token limits are reached.
- `codex-rs/core/src/session/turn.rs:1096-1108` maps `ContextWindowExceeded` from sampling to a full-context token state.
- `codex-rs/protocol/src/protocol.rs:1672-1720` exposes `ContextWindowExceeded` as a client-visible error kind that affects turn status.
- `codex-rs/protocol/src/protocol.rs:1994-2072` defines `TokenUsage`, `TokenUsageInfo`, usage appending, and full-context-window fill behavior.
- `codex-rs/protocol/src/protocol.rs:2146-2188` defines `tokens_in_context_window()` and remaining-percent calculation after subtracting a 12k baseline for fixed prompt/tool overhead.
- `codex-rs/core/src/session/mod.rs:3218-3348` updates token info, notifies token-usage extension contributors, recomputes estimated usage, emits `TokenCount` events, and marks token usage full on context-window overflow.
- `codex-rs/core/src/context/token_budget_context.rs:4-42` renders an initial developer-role `<token_budget>` fragment containing thread id, context window id, and tokens left.
- `codex-rs/core/src/context/token_budget_context.rs:44-81` renders developer-role remaining-token updates.
- `codex-rs/core/src/session/token_budget.rs:6-43` records remaining-token context when usage crosses 25%, 50%, or 75% of the model context window.
- `codex-rs/core/src/session/mod.rs:3033-3045` injects initial token-budget metadata into the context when the feature is enabled.
- `codex-rs/tui/src/chatwidget.rs:1109-1140` applies token usage info to the bottom pane as remaining percentage when context window is known, otherwise used tokens.
- `codex-rs/tui/src/bottom_pane/footer.rs:1008-1020` renders `N% context left`, `N used`, or default `100% context left`.

Runtime trace evidence:
- Claude trace: REPL parses budget text and snapshots output-token baseline, `query()` checks context threshold before calling the model, provider requests may include `output_config.task_budget`, and token-budget continuation injects a meta user nudge when output-token progress is below target.
- Claude overflow trace: local blocking yields a synthetic assistant API error `Prompt is too long`; provider `model_context_window_exceeded` yields a context-window-limit API error on the max-output-token recovery path.
- Codex trace: completed SSE usage is converted to `TokenUsage`, recorded into session history state, used by `auto_compact_token_status()`, emitted as `TokenCount`, and displayed in the TUI.
- Codex limit trace: pre-sampling and post-sampling code both check `token_limit_reached`; context-window overflow from sampling sets total tokens to the full model context window and returns `ContextWindowExceeded`.

What the model sees:
- Claude: the model may see an API-side `output_config.task_budget` when enabled. For local output-token budget continuation, the model sees a meta user message such as "Stopped at N% of token target ... Keep working - do not summarize."
- Codex: when the token-budget feature is enabled, the model sees developer-role `<token_budget>` fragments with current context-window id/tokens-left and later remaining-token updates at threshold crossings.

What the runtime does:
- Claude: resolves model context size, estimates active prompt size from prior API usage plus unsent message estimates, reserves manual-compact space, and blocks or recovers before/after model calls depending on overflow owner. Separately, it tracks output-token budgets by comparing current turn output tokens to a user-specified budget.
- Codex: stores provider usage in session history, computes active context tokens from server usage plus local estimates, derives scoped and full-window token limits, checks limits before and after sampling, and emits token-count events for clients.

What gets persisted:
- Claude: the cited budget state is process/session runtime state; `output_token_usage` can be attached as a message attachment. The card does not establish durable persistence of token-budget snapshots from the cited paths.
- Codex: `TokenUsageInfo` is held in context manager/session state and emitted through `TokenCount` events; session comments state resume/fork reconstruction seeds token state from the last persisted rollout token-count event. Token-budget remaining updates are recorded as conversation items.

What the user/TUI sees:
- Claude: prompt-too-long appears through assistant API error UI; spinner can show budget progress and target completion text; attachments can report turn/session output-token usage against budget.
- Codex: the bottom pane shows context remaining percentage when model context window is known, otherwise used tokens; `ContextWindowExceeded` is a client-visible error kind.

What debug/eval surfaces exist:
- Claude: debug logging records token-budget continuation counts and percentages; context visualization and analysis code elsewhere consume token counts, but this card cites only the runtime/display surfaces used by the limit path.
- Codex: trace logs record total usage, scoped usage, estimated token count, limit values, full-window overflow, and follow-up state after sampling; telemetry records completed SSE usage; token usage contributors receive updated `TokenUsageInfo`; inference traces record completed usage.

Shared invariant:
- Context-limit decisions must use context-window usage, not only generated output tokens. Output-token budgets are a separate progress-control mechanism and must not be confused with prompt/context-window limits.

Claude-specific design:
- Claude's canonical threshold count is `tokenCountWithEstimation()`: it trusts the last real API usage and estimates only local messages added afterward. It has a separate user-facing output-token budget shorthand that can force continuation via meta user nudges.

Codex-specific design:
- Codex stores token usage in `TokenUsageInfo`, treats context-window status as session state, exposes it through `TokenCount` events, and injects developer-role token-budget context into the prompt when the feature is enabled.

Gaps or unknowns:
- Claude comments note an open question about whether server budget countdown counts cache tokens the same way as client-side final-window calculation.
- Codex `ContextManager::estimate_token_count()` is explicitly a coarse byte-based lower bound, not tokenizer-accurate.
- Full compaction behavior, replacement history, and rehydration are intentionally deferred to mechanism 9.

Distilled harness rule:
- Keep three counters separate: provider-reported context usage, local estimated pending prompt growth, and optional user/model-visible output-token budget. Use provider usage plus estimates for context-window safety, and expose budget status to the model only through an explicit role-marked prompt artifact.

Anti-invention rule:
- Do not claim that token budgeting is one universal feature. Claude's output-token budget and API task budget are source-specific; Codex's `<token_budget>` developer fragments and `TokenCount` events are source-specific. Generalize only the invariant that context-window limits and output-budget progress must be distinct.

Verification pattern:
- For Claude, trace `getContextWindowForModel()` and `tokenCountWithEstimation()` into `calculateTokenWarningState()`, then into the pre-call blocking branch in `query()`. Trace output-token budgets separately from `parseTokenBudget()` to `snapshotOutputTokensForTurn()`, `checkTokenBudget()`, meta user continuation, and `configureTaskBudgetParams()`.
- For Codex, trace `ModelInfo` window fields to `TurnContext::model_context_window()`, completed SSE usage to `TokenUsage`, `record_token_usage_info()`, `auto_compact_token_status()`, `TokenCount` event emission, and TUI `context_window_line()`. Trace model-visible budget text from `TokenBudgetContext` and `maybe_record_token_budget_remaining_context()`.

Skill text candidate:
- When analyzing token limits, first classify every token number by purpose:
  - context-window capacity from model metadata or context resolver;
  - provider-reported usage from completed responses;
  - local estimated usage for pending/unreported messages;
  - reserved headroom for model output or manual recovery;
  - optional output-token budget requested by the user or API.
- Then prove where each number is used: blocking, compaction trigger, retry/recovery, model-visible prompt hint, user/TUI display, telemetry, or persisted state.
- Never use an output-token counter as evidence for context-window safety unless the source explicitly says it is the context-window count.
