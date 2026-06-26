# Mechanism 7: Dynamic context assembly

Mechanism: Dynamic context assembly

Status: complete

Claude source:
- `src/context.ts:36-111` builds a memoized git-status snapshot with branch, default branch, short status, recent commits, and optional truncation.
- `src/context.ts:113-149` defines memoized `getSystemContext()`, which conditionally includes `gitStatus` and a gated cache-breaker injection.
- `src/context.ts:152-188` defines memoized `getUserContext()`, which conditionally loads CLAUDE.md/rules content through `getMemoryFiles()`/`getClaudeMds()` and always adds `currentDate`.
- `src/utils/claudemd.ts:1-26` documents memory-file discovery order and include semantics; `src/utils/claudemd.ts:89-92` defines the instruction header and max memory character count; `src/utils/claudemd.ts:424-436` reads memory files with expected-missing-file tolerance.
- `src/utils/queryContext.ts:30-74` centralizes concurrent loading of `getSystemPrompt()`, `getUserContext()`, and `getSystemContext()`, and skips default prompt/system context when a custom system prompt replaces the default.
- `src/screens/REPL.tsx:2767-2800` loads fresh tools, default system prompt, user context, and system context for a normal REPL turn; it augments user context with coordinator context and, behind feature gates, terminal-focus state.
- `src/screens/REPL.tsx:2533-2544` reloads prompt/user/system context when moving a foreground query into a background session.
- `src/screens/REPL.tsx:3797-3810` reads CLAUDE.md/rules files at startup, logs loaded files, and seeds `readFileState`.
- `src/QueryEngine.ts:284-325` does the same context loading for SDK/headless turns and optionally layers memory-mechanics prompt when a custom prompt and memory override are present.
- `src/query.ts:449-451` appends system context to the system prompt, and `src/query.ts:659-661` prepends user context to the messages passed to the model.
- `src/utils/api.ts:437-447` serializes system context as `key: value` lines appended to system prompt blocks; `src/utils/api.ts:449-474` serializes user context as a meta user message inside `<system-reminder>`.
- `src/utils/api.ts:479-499` logs context size metrics for git status and CLAUDE.md; `src/utils/analyzeContext.ts:937-955` rebuilds effective prompt/context surfaces for analysis.

Codex source:
- `codex-rs/core/src/session/turn_context.rs:105-150` defines the per-turn dynamic fields, including environments, cwd, current date, timezone, developer/user instructions, collaboration mode, permissions, network, and features.
- `codex-rs/core/src/session/turn_context.rs:397-420` converts `TurnContext` into persisted `TurnContextItem` baseline fields.
- `codex-rs/core/src/session/turn_context.rs:575-615` initializes a turn with local date/timezone, loaded AGENTS-derived user instructions, approval policy, permission profile, network, and related state.
- `codex-rs/core/src/session/mod.rs:2845-3101` builds initial context as developer sections, contextual-user sections, and separate developer sections, including permissions, developer instructions, collaboration mode, realtime/personality updates, apps/skills/plugins summaries, extension fragments, loaded user instructions, token-budget metadata, environment context, and multi-agent hints.
- `codex-rs/core/src/session/mod.rs:3172-3216` decides whether to inject full initial context or only settings diffs, records model-visible context items, persists a `TurnContextItem`, and advances the in-memory reference baseline.
- `codex-rs/core/src/context_manager/updates.rs:21-40` creates environment update items only when environment context changes from the previous baseline.
- `codex-rs/core/src/context_manager/updates.rs:166-180` emits model-switch instructions when the model changes.
- `codex-rs/core/src/context_manager/updates.rs:183-208` converts developer/contextual-user text sections into `ResponseItem::Message` values with role `developer` or `user`.
- `codex-rs/core/src/context_manager/updates.rs:210-244` builds steady-state settings update items for model, permissions, collaboration mode, realtime, personality, and environment.
- `codex-rs/core/src/context/environment_context.rs:19-27` defines the environment context fields; `codex-rs/core/src/context/environment_context.rs:422-437` builds them from `TurnContext`; `codex-rs/core/src/context/environment_context.rs:520-580` renders them as a contextual user fragment with cwd/shell/date/timezone/network/filesystem/subagent markup.
- `codex-rs/core/src/session/turn.rs:150-166` records context updates before skill/plugin injection and before the first sampling request.
- `codex-rs/core/src/session/turn.rs:221-245` sends `clone_history().for_prompt(...)` to sampling, so recorded context items are prompt input.
- `codex-rs/core/src/session/mod.rs:2658-2670` records items into history, persists rollout response items, and emits raw response items.
- `codex-rs/protocol/src/protocol.rs:2983-3026` defines durable `TurnContextItem` fields for resume/fork replay baselines.
- `codex-rs/protocol/src/protocol.rs:481-535` defines client-supplied additional context and its `Untrusted` vs `Application` kind.
- `codex-rs/core/src/state/additional_context.rs:15-36` diffs client-supplied additional context and converts changed untrusted entries to user fragments and application entries to developer fragments.
- `codex-rs/context-fragments/src/fragment.rs:37-88` defines fragments that own their response role, markers, body, render output, and `ResponseItem` conversion.
- `codex-rs/context-fragments/src/additional_context.rs:21-92` renders additional context as role `user` external tags or role `developer` XML-like tags with per-value truncation.
- `codex-rs/core/src/hook_runtime.rs:595-615` records hook additional contexts as developer-role context messages via `HookAdditionalContext`; `codex-rs/core/src/context/hook_additional_context.rs:14-29` defines that role/body.
- `codex-rs/core/src/prompt_debug.rs:74-101` rebuilds prompt input from a session by recording context updates and user input, then returning `prompt.input`.

Runtime trace evidence:
- Claude REPL trace: `REPL.tsx` loads context at turn start (`query_context_loading_start`/`query_context_loading_end`), passes `userContext` and `systemContext` to `query()`, and `query()` applies `appendSystemContext()` plus `prependUserContext()` immediately before `deps.callModel()`.
- Claude SDK/headless trace: `QueryEngine.ask()` calls `fetchSystemPromptParts()`, merges coordinator user context, builds `systemPrompt`, and passes the same `userContext`/`systemContext` into `query()`.
- Codex turn trace: `run_turn()` calls `record_context_updates_and_set_reference_context_item()` before sampling, then later builds `sampling_request_input` from cloned history and passes it to `run_sampling_request()`.
- Codex prompt-debug trace: `build_prompt_input_from_session()` uses the same context-recording path before exposing prompt input for inspection.

What the model sees:
- Claude: `systemContext` appears as appended system-prompt text lines. `userContext` appears before conversation messages as a meta user message with `<system-reminder>` and per-key `# key` sections.
- Codex: initial dynamic context appears as one or more `developer` messages and one contextual `user` message in conversation history. Later turns may see only diff messages. Environment context is a contextual user fragment wrapped by the environment-context markers and rendered with XML-like fields.

What the runtime does:
- Claude: collects dynamic context per session through memoized functions, reloads it at turn-entry points, augments it in REPL/SDK call sites, and injects it at API-call construction time rather than as regular conversation history.
- Codex: materializes context into `ResponseItem` messages before sampling, records those items into history, uses a persisted `TurnContextItem` as the diff baseline, and emits full context only when no baseline exists or when a context window is re-established.

What gets persisted:
- Claude: the context memoization and `readFileState` are runtime state; the model-visible `prependUserContext()` message is constructed for the model request and is not shown here as a transcript item. Startup preloads CLAUDE.md/rules file raw bytes into `readFileState` for file-tool safety.
- Codex: context messages recorded by `record_conversation_items()` are persisted as rollout response items, and one `TurnContextItem` is persisted per real user turn so resume/fork can recover the current context baseline.

What the user/TUI sees:
- Claude: the TUI does not display the injected `<system-reminder>` as a user-authored message in the cited path. Debug logs can show which CLAUDE.md/rules files loaded at startup.
- Codex: clients can receive `RawResponseItem` events for recorded context items via `send_raw_response_items()`. Warnings may be emitted while rendering skills instructions, but detailed skills behavior belongs to mechanism 22.

What debug/eval surfaces exist:
- Claude: diagnostic logs record `git_status_*`, `system_context_*`, and `user_context_*` events; `logContextMetrics()` measures git-status and CLAUDE.md context size; `analyzeContext` rebuilds effective prompt/context surfaces.
- Codex: prompt debug rebuilds model prompt input through the same recording path; persisted rollout response items and `TurnContextItem` create replayable context evidence.

Shared invariant:
- Dynamic context must enter the model through explicit, source-auditable prompt artifacts. The harness must preserve role boundaries: instruction-like context belongs in system/developer surfaces; situational or untrusted context belongs in user/contextual-user surfaces.

Claude-specific design:
- Claude separates dynamic context into two maps: `systemContext` appended to the system prompt and `userContext` prepended as a meta user reminder. The context is memoized for the conversation, and a custom system prompt suppresses the default prompt and system context but still loads user context.

Codex-specific design:
- Codex makes dynamic context durable by turning it into conversation `ResponseItem`s and separately persisting `TurnContextItem` snapshots for diffing. It supports fragment slots, extension contributors, client-supplied additional context, hook additional context, and steady-state diff messages.

Gaps or unknowns:
- Claude persistence of the model-visible context injection is not established by the cited paths; the card only claims request-time injection plus runtime/cache state.
- Codex `build_settings_update_items()` explicitly notes that it does not yet cover every model-visible item emitted by full initial context.
- Skills/plugins/apps content is cited only as a source of dynamic sections; their detailed loading and projection belong to later mechanisms.

Distilled harness rule:
- Build a typed context assembly boundary that returns role-separated artifacts, then inject them through one auditable path immediately before sampling. If context must survive resume/fork or diff across turns, persist both the emitted context items and a compact baseline snapshot used for future diffing.

Anti-invention rule:
- Do not invent a generic "context provider" model unless the target source has one. Claude supports a two-map request-time injection pattern; Codex supports persisted response-item context plus baseline diffing. Treat these as separate designs unless source code proves convergence.

Verification pattern:
- For Claude, trace `getUserContext()`/`getSystemContext()` to the REPL or SDK caller, then to `query()`, then to `appendSystemContext()` and `prependUserContext()` in the model-call arguments.
- For Codex, trace `TurnContext` creation to `record_context_updates_and_set_reference_context_item()`, then to `build_initial_context()` or `build_settings_update_items()`, then to `record_conversation_items()`, `TurnContextItem`, and `clone_history().for_prompt(...)`.

Skill text candidate:
- When analyzing dynamic context, first identify whether the harness injects context ephemerally at request construction or persists it into prompt history. Then map every context source to a role and a lifecycle:
  - source collection: date/time, cwd/workspace, permissions, git status, project/user instruction files, runtime focus state, extension/hook/client context;
  - assembly boundary: function that returns role-separated context or prompt items;
  - model-visible projection: system/developer/user/contextual-user message shape;
  - persistence boundary: request-only, runtime cache, transcript/history item, or durable baseline snapshot;
  - update policy: full re-injection, memoization, or diff from a stored baseline.
- Never summarize dynamic context as "the app adds environment info" without citing the source function, the role it becomes, and whether it is persisted.
