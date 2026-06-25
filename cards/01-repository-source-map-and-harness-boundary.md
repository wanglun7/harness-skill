# Mechanism 1: Repository/source map and harness boundary definition

Mechanism: Repository/source map and harness boundary definition

Status: complete

Claude source:
- CLI bootstrap and command dispatch starts in `src/entrypoints/cli.tsx`. `main()` is explicitly the bootstrap entrypoint, checks special flags, and dynamically loads deeper paths after fast-path handling (`src/entrypoints/cli.tsx:28-33`, `src/entrypoints/cli.tsx:36-41`, `src/entrypoints/cli.tsx:44-70`).
- Process/session initialization is separated in `src/entrypoints/init.ts`; `init` enables config, safe env, graceful shutdown, analytics, OAuth account info, IDE detection, repository detection, remote settings, mTLS/proxy, and API preconnect (`src/entrypoints/init.ts:57-88`, `src/entrypoints/init.ts:108-159`).
- Headless/SDK execution enters `runHeadless()` in `src/cli/print.ts`, which accepts prompt input, app state accessors, commands, tools, MCP configs, agents, and SDK/headless options (`src/cli/print.ts:455-493`). `print.ts` imports `ask` from `src/QueryEngine.js` (`src/cli/print.ts:91-92`), and `ask()` is documented as a non-interactive convenience wrapper around `QueryEngine` (`src/QueryEngine.ts:1179-1186`).
- Conversation lifecycle and cross-turn state are owned by `QueryEngine`. The class comment says it owns query lifecycle and session state, supports headless/SDK and future REPL use, and preserves state across `submitMessage()` calls (`src/QueryEngine.ts:175-184`). Its config includes cwd, tools, commands, MCP clients, agents, app state, prompts, model options, turn/budget limits, JSON schema, partial messages, and abort/permission plumbing (`src/QueryEngine.ts:130-173`).
- The lower-level turn loop is `query()` / `queryLoop()` in `src/query.ts`. `QueryParams` defines messages, system prompt, user/system context, permission function, tool context, query source, max turns, and injected deps (`src/query.ts:181-199`). `query()` yields stream/request/message/tombstone/tool-summary events and returns a terminal value (`src/query.ts:219-239`).
- Tool boundary is the `Tool` shape in `src/Tool.ts`. A `Tools` set is explicitly defined as a readonly collection to make tool assembly/filtering traceable across the codebase (`src/Tool.ts:697-701`). Tool APIs separate model result serialization from TUI rendering through methods such as `mapToolResultToToolResultBlockParam`, `renderToolResultMessage`, `renderToolUseMessage`, progress, queued, rejected, error, and grouped render hooks (`src/Tool.ts:557-694`).
- Interactive display boundary is React/Ink state. `src/components/App.tsx` is the top-level wrapper for interactive sessions and provides FPS metrics, stats context, and app state to children (`src/components/App.tsx:15-19`, `src/components/App.tsx:27-54`).
- Persistence boundary is JSONL transcript storage in `src/utils/sessionStorage.ts`. `isTranscriptMessage()` is documented as the single source of truth for persisted transcript messages; progress messages are explicitly UI-only and excluded from the JSONL/parent chain (`src/utils/sessionStorage.ts:128-146`, `src/utils/sessionStorage.ts:180-185`). Transcript paths are rooted under the session project dir or project dir plus session id (`src/utils/sessionStorage.ts:198-205`).

Codex source:
- Top-level CLI dispatch is in `codex-rs/cli/src/main.rs`. `MultitoolCli` forwards no-subcommand invocations to the interactive CLI and defines subcommands including `Exec`, `Review`, `McpServer`, `AppServer`, `RemoteControl`, `Resume`, `Fork`, debug, sandbox, and cloud-related commands (`codex-rs/cli/src/main.rs:90-120`, `codex-rs/cli/src/main.rs:122-211`).
- Core harness library boundary is `codex-rs/core/src/lib.rs`. It identifies itself as the root of `codex-core`, denies direct stdout/stderr in library code, exposes `session` internally, and re-exports thread/session/rollout-facing types such as `CodexThread`, `ThreadManager`, `RolloutRecorder`, and `ModelClient` (`codex-rs/core/src/lib.rs:1-18`, `codex-rs/core/src/lib.rs:21-31`, `codex-rs/core/src/lib.rs:112-121`, `codex-rs/core/src/lib.rs:149-187`).
- Core session boundary is `codex-rs/core/src/session/session.rs`. `Session` is documented as the context for an initialized model agent; it has at most one running task at a time and can be interrupted by user input (`codex-rs/core/src/session/session.rs:23-26`). Its fields hold thread identity, event sender, agent status, state, feature invariants, conversation manager, active turn, input queue, guardian review manager, services, and sub-id counter (`codex-rs/core/src/session/session.rs:26-47`).
- Session configuration separates provider, collaboration mode, developer instructions, loaded instruction files, personality, base instructions, compaction prompt, approval policy, permission profile state, environment selections, workspace roots, state home, session source, fork/parent thread ids, dynamic tools, and shell override (`codex-rs/core/src/session/session.rs:49-110`).
- Turn runtime boundary is `run_turn()` in `codex-rs/core/src/session/turn.rs`. It runs pre-sampling compaction, records context updates, builds skills/plugins, runs hooks, records inputs/injections, tracks config analytics, creates a turn diff tracker, loops over pending input, prepares model request input from history, builds Responses metadata, and calls sampling (`codex-rs/core/src/session/turn.rs:140-245`).
- Provider streaming boundary inside the turn is `try_run_sampling_request()`, which obtains an inference trace and calls `client_session.stream(...)` with the prompt, model info, telemetry, reasoning settings, service tier, metadata, and inference trace (`codex-rs/core/src/session/turn.rs:1849-1895`).
- Tool boundary is `codex-rs/core/src/tools/mod.rs`, which declares tool submodules for hosted specs, lifecycle, network approval, orchestrator, parallel, registry, router, runtimes, sandboxing, spec-plan, and dispatch tracing (`codex-rs/core/src/tools/mod.rs:1-17`). It re-exports `ToolRouter` (`codex-rs/core/src/tools/mod.rs:25-25`) and contains model-facing exec-output formatting separate from raw execution output (`codex-rs/core/src/tools/mod.rs:60-87`).
- Protocol boundary is `codex-rs/protocol/src/protocol.rs`, documented as the protocol between a client and an agent, using a Submission Queue / Event Queue pattern for asynchronous communication (`codex-rs/protocol/src/protocol.rs:1-4`). It defines shared tags for model-visible context blocks and user-message prefix constants (`codex-rs/protocol/src/protocol.rs:93-109`), and `Submission` as the user-to-agent request envelope with correlation id, operation, client user message id, and trace carrier (`codex-rs/protocol/src/protocol.rs:148-159`).
- Interactive TUI boundary is `codex-rs/tui/src/lib.rs`, which denies direct stdout/stderr in TUI library code (`codex-rs/tui/src/lib.rs:1-5`), imports app-server clients and protocol/config types (`codex-rs/tui/src/lib.rs:23-59`), and declares rendering/input/status/streaming/chatwidget/session modules (`codex-rs/tui/src/lib.rs:89-188`). TUI CLI options are in `codex-rs/tui/src/cli.rs`, including prompt, resume/fork controls, approval policy, web search, and alternate-screen mode (`codex-rs/tui/src/cli.rs:8-76`).
- Non-interactive exec boundary is `codex-rs/exec/src/lib.rs`. It states stdout constraints for default and JSON modes, denies accidental stdout printing, and imports app-server request/notification types, core config/state, protocol rollout/session items, and event processors (`codex-rs/exec/src/lib.rs:1-15`, `codex-rs/exec/src/lib.rs:16-107`). `ExecRunArgs` binds in-process app-server start args, state DB, config, prompt/output mode, model provider, schema path, and sandbox bypass flag (`codex-rs/exec/src/lib.rs:203-219`).
- Persistence/rollout boundary is exposed from `codex-rs/core/src/rollout.rs`, which re-exports rollout recorder/session/thread lookup types and adapts `Config` to `RolloutConfigView` with codex home, sqlite home, cwd, model provider, and memory generation setting (`codex-rs/core/src/rollout.rs:1-24`, `codex-rs/core/src/rollout.rs:26-46`). `Session::new()` initializes thread persistence via `CreateThreadParams`/`ResumeThreadParams` unless config is ephemeral (`codex-rs/core/src/session/session.rs:521-560`).

Runtime trace evidence:
- Claude interactive/headless boundary trace: `entrypoints/cli.tsx::main()` -> `entrypoints/init.ts::init()` for full startup -> `cli/print.ts::runHeadless()` for print/SDK/headless -> `QueryEngine.ts::ask`/`QueryEngine.submitMessage()` -> `query.ts::query()` / `queryLoop()` -> tools and API services. Evidence: `src/entrypoints/cli.tsx:28-70`, `src/entrypoints/init.ts:57-159`, `src/cli/print.ts:455-493`, `src/QueryEngine.ts:175-184`, `src/QueryEngine.ts:1179-1186`, `src/query.ts:219-239`.
- Claude display/persistence trace: interactive sessions are wrapped by `App` state providers (`src/components/App.tsx:15-19`), while persisted transcript membership is controlled by `isTranscriptMessage()` and excludes progress messages (`src/utils/sessionStorage.ts:128-146`, `src/utils/sessionStorage.ts:180-185`).
- Codex interactive/non-interactive boundary trace: `cli/src/main.rs::MultitoolCli` routes to `codex_tui::Cli` for interactive or `codex_exec::Cli` for exec (`codex-rs/cli/src/main.rs:90-127`); TUI and exec both use app-server/protocol/core surfaces (`codex-rs/tui/src/lib.rs:23-59`, `codex-rs/exec/src/lib.rs:16-107`); core `Session` owns conversation state and `run_turn()` drives the model/tool loop (`codex-rs/core/src/session/session.rs:23-47`, `codex-rs/core/src/session/turn.rs:140-245`).
- Codex provider trace: `run_turn()` builds request input from session history and calls `run_sampling_request`; `try_run_sampling_request()` calls `client_session.stream(...)` (`codex-rs/core/src/session/turn.rs:221-245`, `codex-rs/core/src/session/turn.rs:1849-1895`).

What the model sees:
- Claude: model-visible material is downstream of `QueryParams`: messages, system prompt, user context, system context, tool context, query source, and limits (`src/query.ts:181-199`). Tool result projection to the provider is through `mapToolResultToToolResultBlockParam`, not the React renderer (`src/Tool.ts:557-560`).
- Codex: model-visible material is downstream of session configuration and turn request construction: base/developer/loaded instruction configuration, dynamic tools, history converted with `for_prompt`, Responses metadata, and context tags in protocol constants (`codex-rs/core/src/session/session.rs:49-110`, `codex-rs/core/src/session/turn.rs:221-245`, `codex-rs/protocol/src/protocol.rs:93-109`).

What the runtime does:
- Claude: bootstraps CLI flags, initializes config/network/telemetry/auth/repository state, runs headless/SDK or interactive surfaces, owns conversation state in `QueryEngine`, and delegates each turn to `query()`/`queryLoop()`.
- Codex: routes top-level CLI modes, keeps stdout/stderr out of core/TUI libraries, uses core `Session` as the agent context, runs turn lifecycle in `run_turn()`, and communicates client/agent activity through protocol submission/event surfaces.

What gets persisted:
- Claude: transcript entries are user/assistant/attachment/system only; progress is excluded as ephemeral UI state. Transcript path uses the current session id under project/session directory (`src/utils/sessionStorage.ts:128-146`, `src/utils/sessionStorage.ts:180-205`).
- Codex: rollout/thread persistence is a first-class boundary. Core re-exports rollout reader/recorder functions (`codex-rs/core/src/rollout.rs:1-24`), `Config` supplies home/sqlite/cwd/provider/memory fields to rollout (`codex-rs/core/src/rollout.rs:26-46`), and `Session::new()` creates or resumes live threads unless ephemeral (`codex-rs/core/src/session/session.rs:521-560`).

What the user/TUI sees:
- Claude: interactive sessions are React/Ink trees rooted by `App`, with app state, stats, and FPS contexts (`src/components/App.tsx:15-19`). Tool definitions own TUI render hooks separately from model result serialization (`src/Tool.ts:566-694`). Headless stream-json/text output is handled in `cli/print.ts::runHeadless()` (`src/cli/print.ts:455-493`).
- Codex: interactive UI is the `codex_tui` library with rendering/input/status/streaming/chatwidget modules (`codex-rs/tui/src/lib.rs:89-188`) and CLI flags for prompt, resume/fork, approvals, web search, and alt-screen (`codex-rs/tui/src/cli.rs:8-76`). Non-interactive exec constrains stdout to final message or JSONL and sends other output to stderr (`codex-rs/exec/src/lib.rs:1-5`).

What debug/eval surfaces exist:
- Claude: CLI includes a feature-gated `--dump-system-prompt` path for prompt sensitivity evals (`src/entrypoints/cli.tsx:50-70`), startup profiling checkpoints in CLI/init (`src/entrypoints/cli.tsx:44-48`, `src/entrypoints/init.ts:57-60`), and diagnostic/debug logging during init (`src/entrypoints/init.ts:57-66`, `src/entrypoints/init.ts:136-150`).
- Codex: top-level CLI includes debug tooling as a subcommand (`codex-rs/cli/src/main.rs:168-173`), core/TUI/exec route output through tracing/event processors instead of direct prints (`codex-rs/core/src/lib.rs:1-6`, `codex-rs/tui/src/lib.rs:1-5`, `codex-rs/exec/src/lib.rs:1-15`), and turn sampling carries trace context through `inference_trace_context` before streaming (`codex-rs/core/src/session/turn.rs:1876-1895`).

Shared invariant:
- Both references separate the harness into entrypoint/CLI dispatch, initialization/config, conversation/session state, turn loop, tool boundary, display/output boundary, persistence boundary, and debug/trace surfaces. The common invariant is not a specific module layout; it is that model-facing data, runtime state, user display, and persisted transcript/rollout are separate surfaces with explicit crossing points.

Claude-specific design:
- TypeScript/Bun/React-Ink layout under a single `src` tree.
- `QueryEngine` is a class wrapper over the lower-level generator loop, with comments stating it was extracted from `ask()` for headless/SDK and future REPL use (`src/QueryEngine.ts:175-184`).
- Tool definitions directly include React rendering hooks and model-facing serialization in one `Tool` type (`src/Tool.ts:557-694`).
- Transcript persistence explicitly excludes progress messages and treats them as UI-only (`src/utils/sessionStorage.ts:128-146`, `src/utils/sessionStorage.ts:180-185`).

Codex-specific design:
- Rust multi-crate workspace with separate `cli`, `core`, `protocol`, `tui`, `exec`, rollout/state, and app-server surfaces.
- Core and TUI libraries deny direct stdout/stderr writes (`codex-rs/core/src/lib.rs:1-6`, `codex-rs/tui/src/lib.rs:1-5`); exec separately constrains stdout for final/JSON output (`codex-rs/exec/src/lib.rs:1-5`).
- `Session` is the initialized model-agent context and is explicitly single-active-task/interruptible (`codex-rs/core/src/session/session.rs:23-26`).
- Protocol names the client-agent queue boundary as Submission Queue / Event Queue (`codex-rs/protocol/src/protocol.rs:1-4`).

Gaps or unknowns:
- This card does not yet enumerate the exact model/provider abstraction; that is mechanism 2.
- This card does not yet prove exact stop conditions or streaming interleaving; those are mechanisms 3 and 4.
- Claude interactive REPL entrypoint was not exhaustively traced in this card because mechanism 1 only needs the source map and harness boundary, and `QueryEngine`/`runHeadless`/`App` give sufficient boundary evidence for the first mechanism.
- No live runtime command trace was executed; runtime evidence here is a static source trace through entrypoint and turn-loop functions.

Distilled harness rule:
- Define the harness boundary by explicit crossing points: CLI/mode dispatch, startup initialization, session/conversation owner, turn-loop owner, provider-stream call, tool registry/projection, UI/output renderer, persistence writer, and debug/trace surface. Do not treat a repository directory or product name as the boundary unless source functions/types show how data crosses it.

Anti-invention rule:
- Do not invent a universal architecture from one reference. If a boundary exists only in Claude as a class/generator/React tool renderer, label it Claude-specific. If a boundary exists only in Codex as a multi-crate protocol/session/rollout split, label it Codex-specific.

Verification pattern:
- Start with entrypoints and CLI dispatch.
- Trace into initialization/config.
- Identify the session/conversation owner by source comments and fields.
- Identify the turn-loop function and provider stream call.
- Identify tool boundary methods/types and separate model-facing serialization from UI rendering.
- Identify persistence membership rules and file/thread storage surfaces.
- Identify TUI/headless output boundaries and debug/trace surfaces.
- Record only citations that name a file plus line range.

Skill text candidate:
- Before comparing or reusing any coding-agent harness mechanism, build a source map card. For each reference, cite the exact entrypoint, init/config boundary, session owner, turn-loop owner, provider-stream call, tool boundary, display/output boundary, persistence boundary, and debug/trace surface. Then state which boundary facts are shared and which are source-specific. Do not continue to provider abstraction until this card is complete.
