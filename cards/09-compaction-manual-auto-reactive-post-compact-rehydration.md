# Mechanism 9: Compaction: manual, auto, reactive, post-compact rehydration

Mechanism: Compaction: manual, auto, reactive, post-compact rehydration

Status: complete

Claude source:
- `src/commands/compact/compact.ts:55-82` implements manual `/compact` by trying session-memory compaction first when no custom instructions are present, then clearing context caches, running post-compact cleanup, notifying prompt-cache break detection, marking post-compaction, suppressing the compact warning, and returning a `type: 'compact'` result.
- `src/commands/compact/compact.ts:85-94` routes manual `/compact` through the reactive path when reactive-only mode is enabled.
- `src/commands/compact/compact.ts:96-124` falls back to traditional manual compaction: run microcompact, call `compactConversation()`, reset the last summarized message id, suppress warnings, clear user context cache, run post-compact cleanup, and return display text.
- `src/commands/compact/compact.ts:143-220` defines `compactViaReactive()`: execute PreCompact hooks and cache-parameter construction concurrently, merge custom instructions, call `reactiveCompactOnPromptTooLong()`, map unsuccessful outcomes to user-facing errors, perform cleanup, and merge hook/post-compact display messages.
- `src/services/compact/autoCompact.ts:160-239` decides whether proactive auto-compaction should run, including recursion guards, feature/mode suppressions, token estimation minus snip savings, threshold lookup, debug logging, and `calculateTokenWarningState()`.
- `src/services/compact/autoCompact.ts:241-351` runs auto-compaction when needed: it respects disable/circuit-breaker state, tries session-memory compaction first, otherwise calls `compactConversation(..., suppressFollowUpQuestions: true, isAutoCompact: true, recompactionInfo)`, resets summary tracking, runs cleanup, and tracks failures.
- `src/services/compact/compact.ts:299-310` defines `CompactionResult` as a boundary marker, summary messages, attachments, hook results, optional messages to keep, display text, token counts, and compaction usage.
- `src/services/compact/compact.ts:325-338` defines the post-compact message ordering: boundary marker, summary messages, kept messages, attachments, and hook results.
- `src/services/compact/compact.ts:349-367` annotates compact boundaries with preserved-segment metadata so transcript loading can relink kept message chains.
- `src/services/compact/compact.ts:387-460` implements `compactConversation()` setup: validate message count, measure pre-compact tokens, run PreCompact hooks, enter compacting/requesting UI state, build a compact prompt user message, stream the compact summary, and retry prompt-too-long by trimming oldest groups.
- `src/services/compact/compact.ts:587-624` creates the compact boundary, carries pre-compact discovered-tool state in boundary metadata, and creates an invisible transcript-only compact-summary user message.
- `src/services/compact/postCompactCleanup.ts:31-77` resets microcompact, context-collapse, memory-file/user-context caches, system prompt sections, classifier/speculative state, tracing state, optional file-content cache, and session-message cache after compaction.
- `src/query.ts:453-535` calls the auto-compact dependency before sampling, records `tengu_auto_compact_succeeded`, resets tracking, yields `buildPostCompactMessages()`, and continues the current query using only those post-compact messages.
- `src/query.ts:1080-1160` handles reactive overflow recovery: after withheld prompt-too-long or media errors, it optionally drains context-collapse state, calls `reactiveCompact.tryReactiveCompact()`, yields post-compact messages, and retries with `hasAttemptedReactiveCompact: true`.
- `src/utils/messages.ts:4530-4555` creates a `system` `compact_boundary` message with trigger, pre-token count, optional user context, message count, and logical parent uuid.
- `src/utils/messages.ts:4608-4656` identifies compact boundaries and slices model-facing history from the last boundary onward, optionally filtering snipped messages.
- `src/utils/sessionStorage.ts:1025-1048` writes compact boundaries with `parentUuid: null` and `logicalParentUuid` pointing to the previous parent.
- `src/utils/sessionStorage.ts:1839-1932` applies preserved-segment relinks on transcript load, validates tail-to-head walks, skips stale segments when a newer boundary exists, splices preserved chains, and zeros stale assistant input usage to avoid immediate auto-compact loops after resume.
- `src/QueryEngine.ts:916-940` releases pre-compaction messages from mutable/local arrays after a compact boundary is yielded and emits an SDK `system compact_boundary` with compact metadata.
- `src/components/Message.tsx:231-245` renders compact boundaries as `CompactBoundaryMessage` in the TUI except fullscreen mode.
- `src/remote/sdkMessageAdapter.ts:126-140` reconstructs SDK compact-boundary messages into local system messages with `Conversation compacted` content and compact metadata.

Codex source:
- `codex-rs/protocol/src/protocol.rs:621-624` defines `Op::Compact` as a request for the agent to summarize current conversation context.
- `codex-rs/core/src/tasks/compact.rs:24-64` implements the manual compact task, choosing remote v2, remote, or local compaction from provider/feature state, emitting compact metrics, and synthesizing the local compact prompt as `UserInput::Text`.
- `codex-rs/core/src/tasks/mod.rs:154-164` emits a compact telemetry counter tagged by implementation type and whether compaction is manual.
- `codex-rs/core/src/session/turn.rs:798-815` runs pre-sampling auto-compaction when configured auto-compact or usable context-window limits are exhausted.
- `codex-rs/core/src/session/turn.rs:833-899` runs pre-sampling compaction against the previous model when compaction-compatibility hashes differ or when switching to a smaller context-window model after the previous model is over the new limit.
- `codex-rs/core/src/session/turn.rs:907-960` chooses remote v2, remote, or local inline auto-compaction and tags metrics as non-manual.
- `codex-rs/core/src/session/turn.rs:240-334` handles post-sampling/mid-turn compaction: if the model still needs follow-up or input is pending and the token limit is reached, it calls `run_auto_compact(... InitialContextInjection::BeforeLastUserMessage, ContextLimit, MidTurn)` and then continues the loop.
- `codex-rs/core/src/compact.rs:73-98` implements local inline auto-compaction by synthesizing the compact prompt and calling `run_compact_task_inner()` with `CompactionTrigger::Auto`.
- `codex-rs/core/src/compact.rs:100-124` implements local manual compaction by emitting `TurnStarted` and calling `run_compact_task_inner()` with `CompactionTrigger::Manual`, `UserRequested`, and `StandaloneTurn`.
- `codex-rs/core/src/compact.rs:126-190` wraps local compaction with metadata, analytics attempt tracking, PreCompact hooks, PostCompact hooks, and interrupted/error status handling.
- `codex-rs/core/src/compact.rs:197-330` performs local compaction: start a `ContextCompaction` turn item, append compact prompt to a cloned history, stream the compaction request to completion with retry/trimming behavior, collect the last assistant summary, build replacement history, advance the compact window, optionally inject initial context, persist a `CompactedItem`, recompute token usage, complete the turn item, and warn about long threads.
- `codex-rs/core/src/compact.rs:453-477` collects real user messages for compacted history and excludes existing summary messages.
- `codex-rs/core/src/compact.rs:489-534` inserts canonical initial context before the last real user message, summary, or compaction item.
- `codex-rs/core/src/compact.rs:536-605` builds compacted history from optional initial context, selected/truncated recent user messages, and the summary text as a user message.
- `codex-rs/core/src/compact.rs:608-650` drains the local compaction model stream, recording completed output items and usage.
- `codex-rs/core/src/compact_remote.rs:49-89` implements remote inline auto and manual compaction entry points with trigger/reason/phase metadata.
- `codex-rs/core/src/compact_remote.rs:245-292` installs remote compact endpoint output as replacement history, records an installed compaction trace checkpoint, persists a `CompactedItem`, recomputes usage, and completes the compaction turn item.
- `codex-rs/core/src/compact_remote.rs:296-352` processes remote compacted history by optionally building current initial context, retaining only allowed compacted items, and reinserting current context in the model-expected position.
- `codex-rs/core/src/compact_remote_v2.rs:56-99` implements remote-v2 inline auto and manual compaction entry points.
- `codex-rs/core/src/compact_remote_v2.rs:285-318` builds v2 compacted history, tracks retained images and usage, installs replacement history, records an installed trace checkpoint, persists a `CompactedItem`, and recomputes usage.
- `codex-rs/protocol/src/protocol.rs:1238-1244` exposes `EventMsg::ContextCompacted`; `codex-rs/protocol/src/protocol.rs:1929-1930` defines its empty event payload.
- `codex-rs/protocol/src/items.rs:223-238` defines `ContextCompactionItem` and maps it to the legacy `ContextCompacted` event.
- `codex-rs/protocol/src/protocol.rs:2942-2961` defines durable rollout items, including `RolloutItem::Compacted(CompactedItem { message, replacement_history, window_id })`.
- `codex-rs/protocol/src/protocol.rs:2963-2975` converts a `CompactedItem` into an assistant `ResponseItem` containing the compact message for legacy reconstruction.
- `codex-rs/core/src/session/mod.rs:2756-2784` replaces live history, persists `RolloutItem::Compacted`, optionally persists a `TurnContext` reference item, and queues `SessionStartSource::Compact`.
- `codex-rs/core/src/session/mod.rs:3129-3164` starts a requested new context window by replacing history with initial context, persisting an empty-message `CompactedItem` with replacement history/window id plus `TurnContext`, queueing compact session-start source, and recomputing token usage.
- `codex-rs/core/src/state/session.rs:159-170` records and consumes new-context-window requests, advancing the compact window and clearing prefill.
- `codex-rs/core/src/state/auto_compact_window.rs:15-61` stores compact window id, pending new-window request, and prefill baseline; `codex-rs/core/src/state/auto_compact_window.rs:63-82` records server-observed or estimated prefill.
- `codex-rs/core/src/session/rollout_reconstruction.rs:96-142` reconstructs rollouts by scanning newest-to-oldest for the newest surviving `replacement_history` checkpoint and treating compaction as clearing older baselines unless a newer `TurnContextItem` reestablishes one.
- `codex-rs/core/src/session/rollout_reconstruction.rs:261-330` materializes history by replacing with `replacement_history` when present, replaying the surviving suffix, and rebuilding legacy compactions without replacement history from user messages plus compact message.
- `codex-rs/core/src/session/turn.rs:1460-1484` excludes `ContextCompacted` events from realtime text.
- `codex-rs/app-server-protocol/src/protocol/thread_history.rs:1099-1102` maps `ContextCompactedEvent` into a `ThreadItem::ContextCompaction`.
- `codex-rs/app-server/src/bespoke_event_handling.rs:903-906` notes that core still fans out deprecated `ContextCompacted` events for legacy clients while v2 clients receive the canonical `ContextCompaction` item.

Runtime trace evidence:
- Claude manual trace: `/compact` command -> optional session-memory compaction -> optional reactive-only route -> microcompact -> `compactConversation()` -> `CompactionResult` -> `buildPostCompactMessages()` -> query/REPL consumes only boundary + summary + retained/attachment/hook messages.
- Claude proactive trace: `query()` builds messages after the latest compact boundary, calls `deps.autocompact()`, logs success metrics, yields post-compact messages, resets auto-compact tracking, and continues sampling with the post-compact array.
- Claude reactive trace: withheld prompt-too-long/media errors in `query()` -> optional context-collapse drain -> `reactiveCompact.tryReactiveCompact()` -> post-compact messages yielded -> retry state marks `hasAttemptedReactiveCompact`.
- Claude resume trace: persisted `compact_boundary` messages delimit history; loader relinks preserved segments using boundary metadata; `QueryEngine` drops pre-boundary mutable messages after yielding the boundary.
- Codex manual trace: `Op::Compact` -> `CompactTask` -> provider/feature branch -> local/remote/remote-v2 compaction -> `replace_compacted_history()` persists `CompactedItem`.
- Codex proactive trace: `run_pre_sampling_compact()` and previous-model compatibility checks call `run_auto_compact()` before normal sampling.
- Codex mid-turn trace: after sampling, if follow-up/pending input exists and token limits are reached, `run_auto_compact()` runs with `InitialContextInjection::BeforeLastUserMessage`, then the turn loop continues.
- Codex resume trace: rollout reconstruction uses the newest persisted `replacement_history` as the base history, replays newer rollout items, and only falls back to legacy summary rebuilding if a compact item lacks replacement history.

What the model sees:
- Claude: the compaction model call sees a compact prompt as a user summary request over the messages being summarized. After compaction, subsequent model calls see a transcript slice beginning at a system compact boundary plus a compact-summary user message marked transcript-only and any retained messages/attachments/hook results.
- Codex: local compaction sends `turn_context.compact_prompt()` as synthesized user input. After compaction, normal sampling sees the replaced history: selected recent user messages plus a summary for local compaction, or provider-returned compacted history for remote compaction, optionally with current initial context inserted before the last real user message for mid-turn paths.

What the runtime does:
- Claude: creates a structured `CompactionResult`, emits boundary/summary/rehydration messages in fixed order, resets caches and compact tracking, suppresses immediate warnings, and uses compact boundaries to slice future model-facing history.
- Codex: treats compaction as a session task or inline turn-loop operation, records analytics/trace metadata, swaps the session history in the context manager, advances compact-window ids, recomputes token usage, and queues compact-sourced session-start hooks.

What gets persisted:
- Claude: the transcript persists a `system compact_boundary` message with compact metadata and logical parent links; compact summaries are user messages marked `isCompactSummary` and `isVisibleInTranscriptOnly`; preserved-segment metadata can be stored on the boundary for resume relinking.
- Codex: rollout files persist `RolloutItem::Compacted(CompactedItem)`, including `message`, optional full `replacement_history`, and optional `window_id`; compaction may also persist a `TurnContext` reference item.

What the user/TUI sees:
- Claude: manual compaction returns command display text; compact progress can set SDK status to `compacting` and stream/requesting mode; the TUI renders a compact-boundary marker unless fullscreen mode suppresses it.
- Codex: manual compaction emits a turn lifecycle and a `ContextCompaction` item; local compaction emits a warning about long threads and multiple compactions; legacy clients may receive `ContextCompacted`, while v2 clients receive the canonical context-compaction item.

What debug/eval surfaces exist:
- Claude: debug logs include auto-compact threshold state; telemetry logs `tengu_auto_compact_succeeded` and `tengu_compact` metrics; prompt-cache break detection can be notified after compaction; reactive implementation call sites are present but the implementation file is absent in this source snapshot.
- Codex: compaction telemetry is tagged by implementation and manual/auto; compaction analytics records trigger/reason/implementation/phase/status; remote compaction records installed trace checkpoints with input and replacement history; test suites cover replacement history, manual/auto prompt behavior, remote compaction, resume, and context reinjection.

Shared invariant:
- A compaction mechanism must create an explicit durable boundary between pre-compact history and the post-compact model context, then make subsequent model calls use only the post-compact representation plus any explicitly rehydrated state.

Claude-specific design:
- Claude uses in-transcript compact-boundary system messages, transcript-only summary user messages, optional preserved-message segments, and `getMessagesAfterCompactBoundary()` slicing. Manual `/compact` can choose session-memory compaction before traditional summary compaction, and reactive compaction is integrated as overflow recovery from prompt-too-long/media errors.

Codex-specific design:
- Codex uses `CompactedItem.replacement_history` checkpoints in rollout storage, replaces the context manager history directly, advances compact-window ids, and reconstructs resume state by starting from the newest replacement-history checkpoint. Remote compaction endpoint output is filtered and reprocessed before installation.

Gaps or unknowns:
- Claude reactive compaction call sites are present, but no `reactiveCompact` implementation file is available in this source snapshot; only its integration contract can be cited here.
- Claude session-memory compaction is cited through the command/auto call sites in this card, not fully decomposed into its own summary algorithm.
- Codex remote compaction request/response payload internals are only summarized here to the extent needed for replacement-history installation; provider payload construction belongs to mechanism 5.
- The card does not claim that either reference has manual, proactive, reactive, and remote compaction in the same form. Reactive overflow compaction is Claude-specific in the cited snapshot; remote compact endpoint installation is Codex-specific.

Distilled harness rule:
- Implement compaction as a boundary-and-replacement protocol: trigger it from explicit user commands, proactive token checks, or reactive overflow recovery; summarize or obtain compacted history through a cited compaction path; install a minimal post-compact representation; persist a durable checkpoint; and rehydrate only source-proven state needed for correct future turns.

Anti-invention rule:
- Do not invent a universal compaction algorithm. If the source uses compact-boundary transcript slicing, cite that. If it uses rollout replacement history, cite that. If it only has call sites for a reactive implementation, mark the implementation unknown instead of filling in behavior from expectation.

Verification pattern:
- For Claude, trace a manual `/compact` from `compact.ts` into `compactConversation()`, then to `buildPostCompactMessages()`, `QueryEngine` boundary handling, transcript storage, and `getMessagesAfterCompactBoundary()`. Trace proactive auto-compaction from `shouldAutoCompact()` through `autoCompactIfNeeded()` into the `query()` post-compact branch. Trace reactive recovery only through the available `query()` and command call sites.
- For Codex, trace `Op::Compact` into `CompactTask`, then into local/remote/remote-v2 compaction, `replace_compacted_history()`, persisted `CompactedItem`, and rollout reconstruction. Separately trace auto-compaction from `run_pre_sampling_compact()` and the post-sampling `token_limit_reached && needs_follow_up` branch.

Skill text candidate:
- When analyzing a coding-agent compaction design:
  - identify every trigger separately: manual command, pre-turn auto threshold, post-turn/mid-turn threshold, model-switch compatibility, explicit new context window, and reactive overflow;
  - identify the compacting actor: local model summary call, remote compact endpoint, session-memory path, or source-specific overflow recovery;
  - prove the installed post-compact shape: compact boundary plus summary messages, or replacement-history checkpoint;
  - prove which state is rehydrated: attachments, hooks, current initial context, discovered tools, preserved message suffixes, turn context, compact window id, or session-start source;
  - prove what is persisted and how resume reconstructs from it;
  - keep user/TUI markers, model-visible compact summaries, persisted compact records, debug telemetry, and eval tests in separate notes.
