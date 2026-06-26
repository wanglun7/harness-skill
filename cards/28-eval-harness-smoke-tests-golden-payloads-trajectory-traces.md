# Mechanism 28: Eval harness, smoke tests, golden payloads, trajectory traces

Mechanism: Eval harness, smoke tests, golden payloads, trajectory traces
Status: complete

Claude source:
- No first-class test/eval tree was found under the provided `src` by `find ... rg '(__tests__|\.test\.|\.spec\.|eval|fixture|fixtures|golden|snapshot|smoke|trajectory|e2e|vitest|jest)'`; `find ... -name '*test.ts' ...` only found `src/ink/hit-test.ts`, which is source code, not a test harness.
- Testing-only permission tool and registry gate: `src/tools/testing/TestingPermissionTool.tsx:1-73`, `src/tools.ts:58`, `src/tools.ts:225-245`.
- Query testability seams and reducer-oriented config snapshot: `src/query/deps.ts:8-40`, `src/query/config.ts:8-46`.
- VCR fixture recording/replay, fixture hashing, CI missing-fixture behavior, and path normalization: `src/services/vcr.ts:23-86`, `src/services/vcr.ts:88-120`, `src/services/vcr.ts:300-335`.
- VCR hash guards in query/tool/message normalization: `src/query.ts:742-790`, `src/services/tools/toolExecution.ts:775-782`, `src/utils/messages.ts:2318-2369`, `src/utils/messages.ts:2411-2425`.
- Eval-harness feature-flag overrides and deterministic precedence: `src/services/analytics/growthbook.ts:159-209`, `src/services/analytics/growthbook.ts:317-417`, `src/services/analytics/growthbook.ts:670-712`, `src/services/analytics/growthbook.ts:734-775`.
- SWE-bench metadata fields and analytics mapping: `src/services/analytics/metadata.ts:469-496`, `src/services/analytics/metadata.ts:707-724`, `src/services/analytics/metadata.ts:902-920`.
- Test-mode analytics suppression: `src/services/analytics/config.ts:11-27`, `src/services/analytics/config.ts:29-38`.
- Stop/eval guard for invalid model responses and API errors: `src/query.ts:1168-1175`, `src/query.ts:1258-1264`.

Codex source:
- Core integration-test aggregator and common harness modules: `codex-rs/core/tests/all.rs:1-7`, `codex-rs/core/tests/common/lib.rs:27-37`, `codex-rs/core/tests/common/lib.rs:45-74`, `codex-rs/core/tests/common/lib.rs:256-353`.
- Core test builder, local/remote test environments, prompt submission helpers, and `TestCodexHarness`: `codex-rs/core/tests/common/test_codex.rs:115-190`, `codex-rs/core/tests/common/test_codex.rs:250-620`, `codex-rs/core/tests/common/test_codex.rs:700-890`, `codex-rs/core/tests/common/test_codex.rs:893-1040`, `codex-rs/core/tests/common/test_codex.rs:1129`.
- Provider mock/request capture and SSE/WebSocket test servers: `codex-rs/core/tests/common/responses.rs:38-81`, `codex-rs/core/tests/common/responses.rs:101-304`, `codex-rs/core/tests/common/responses.rs:412-550`, `codex-rs/core/tests/common/responses.rs:982-1035`, `codex-rs/core/tests/common/responses.rs:1435-1475`, `codex-rs/core/tests/common/streaming_sse.rs:13-61`, `codex-rs/core/tests/common/streaming_sse.rs:86-173`.
- Provider-payload/context golden formatting and model-visible layout snapshots: `codex-rs/core/tests/common/context_snapshot.rs:12-63`, `codex-rs/core/tests/common/context_snapshot.rs:65-227`, `codex-rs/core/tests/suite/model_visible_layout.rs:33-47`, `codex-rs/core/tests/suite/model_visible_layout.rs:81-200`.
- Prompt-debug payload construction and test: `codex-rs/core/src/prompt_debug.rs:24-102`, `codex-rs/core/tests/suite/prompt_debug_tests.rs:16-70`.
- Core snapshot/golden examples for compaction and shell runtime artifacts: `codex-rs/core/tests/suite/compact.rs:66-180`, `codex-rs/core/tests/suite/shell_snapshot.rs:32-180`.
- App-server JSON-RPC integration harness and v2 suite: `codex-rs/app-server/tests/common/test_app_server.rs:117-276`, `codex-rs/app-server/tests/common/test_app_server.rs:279-360`, `codex-rs/app-server/tests/common/test_app_server.rs:443-560`, `codex-rs/app-server/tests/suite/mod.rs:1-5`, `codex-rs/app-server/tests/suite/v2/mod.rs:1-80`, `codex-rs/app-server/tests/suite/v2/turn_start.rs:1-177`.
- App-server mock model helpers and executable plugin analytics smoke: `codex-rs/app-server/tests/common/mock_model_server.rs:12-65`, `codex-rs/app-server/tests/common/responses.rs:5-105`, `codex-rs/app-server-test-client/src/plugin_analytics_smoke.rs:30-95`, `codex-rs/app-server-test-client/src/plugin_analytics_smoke.rs:97-157`, `codex-rs/app-server-test-client/src/plugin_analytics_smoke.rs:166-221`.
- Rollout-trace writer/replay and inference/protocol-event tests: `codex-rs/rollout-trace/src/writer.rs:31-141`, `codex-rs/rollout-trace/src/writer.rs:157-265`, `codex-rs/rollout-trace/src/inference.rs:404-523`, `codex-rs/rollout-trace/src/protocol_event.rs:406-408`, `codex-rs/rollout-trace/src/protocol_event_tests.rs:13-46`.
- TUI/terminal rendering regression harness: `codex-rs/tui/tests/all.rs:1-10`, `codex-rs/tui/tests/test_backend.rs:1-4`, `codex-rs/tui/tests/suite/vt100_history.rs:22-39`, `codex-rs/tui/tests/suite/vt100_history.rs:41-164`, `codex-rs/tui/tests/suite/status_indicator.rs:1-24`.
- TypeScript SDK proxy/client tests: `sdk/typescript/tests/responsesProxy.ts:22-64`, `sdk/typescript/tests/responsesProxy.ts:93-148`, `sdk/typescript/tests/responsesProxy.ts:150-224`, `sdk/typescript/tests/testCodex.ts:23-87`, `sdk/typescript/tests/runStreamed.test.ts:14-62`, `sdk/typescript/tests/runStreamed.test.ts:64-156`, `sdk/typescript/tests/runStreamed.test.ts:158-200`.
- Portable apply-patch fixture format and runner: `codex-rs/apply-patch/tests/fixtures/scenarios/README.md:1-17`, `codex-rs/apply-patch/tests/suite/scenarios.rs:10-63`, `codex-rs/apply-patch/tests/suite/scenarios.rs:65-126`.

Runtime trace evidence:
- Claude source evidence is source-limited: the provided `src` tree exposes no broad test suite, but it does expose a VCR fixture mechanism, test-only tool surface, dependency-injection seams, deterministic feature-flag overrides for eval harnesses, SWE-bench metadata, and source comments that protect fixture hashes.
- Claude `TestingPermissionTool` is model-visible only under the test registry gate: its prompt says it is used for end-to-end testing, it always asks permission, returns success data, and maps that data into a `tool_result`; `tools.ts` only adds it when `NODE_ENV === 'test'` (`TestingPermissionTool.tsx:1-73`, `tools.ts:244`).
- Claude `query()` can be tested by injected narrow I/O dependencies: `QueryDeps` covers model call, microcompact, autocompact, and uuid; comments state this replaces repeated module spies and keeps signatures tied to production functions (`query/deps.ts:8-40`). `QueryConfig` snapshots immutable runtime gates to make a future pure reducer shape tractable (`query/config.ts:8-46`).
- Claude VCR hashes normalized API-facing messages, stores/reads JSON fixtures under `fixtures/`, records missing fixtures only outside locked CI unless `VCR_RECORD` is set, and normalizes environment-specific paths so hashes match across platforms (`services/vcr.ts:23-86`, `services/vcr.ts:88-120`, `services/vcr.ts:300-335`).
- Claude runtime explicitly preserves VCR fixture hashes: query clones observable tool-use inputs only when fields are added, tool execution avoids mutating call input when backfill would alter transcript text, and message normalization gates transformations that would change fixture hashes (`query.ts:742-790`, `toolExecution.ts:775-782`, `messages.ts:2318-2369`, `messages.ts:2411-2425`).
- Claude eval determinism for feature flags is source-specific. `CLAUDE_INTERNAL_FC_OVERRIDES` bypasses remote eval and disk cache for ant users, env overrides win over local config overrides, remote eval payloads are transformed/cached, and cached getters check env overrides before config/cache (`growthbook.ts:159-209`, `growthbook.ts:317-417`, `growthbook.ts:670-775`).
- Claude evaluation metadata includes SWE-bench ids from `SWE_BENCH_RUN_ID`, `SWE_BENCH_INSTANCE_ID`, and `SWE_BENCH_TASK_ID`, then maps them into core analytics fields (`metadata.ts:469-496`, `metadata.ts:707-724`, `metadata.ts:902-920`).
- Codex has a first-class multi-layer test harness. `core/tests/all.rs` aggregates suite modules; `core/tests/common/lib.rs` configures deterministic process ids and `INSTA_WORKSPACE_ROOT`; event wait helpers consume `CodexThread` events with timeouts (`all.rs:1-7`, `common/lib.rs:45-74`, `common/lib.rs:256-353`).
- Codex `TestCodexBuilder` constructs isolated temp homes/cwds, local or remote test environments, mock provider base URLs, extension registries, config mutators, workspace setup hooks, resume-from-rollout support, WebSocket/streaming test servers, and `ThreadManager` instances wired to test auth and state (`test_codex.rs:115-190`, `test_codex.rs:250-620`). `TestCodex` submits real `Op::UserInput` and waits for turn start/complete events (`test_codex.rs:700-890`).
- Codex provider mocks are inspectable. `ResponseMock` captures all requests, exposes decoded JSON, instructions text, input groups, image URLs, tool definitions, call outputs, headers, and paths; SSE helpers mount one-shot or ordered response sequences with expected call counts (`responses.rs:38-81`, `responses.rs:101-304`, `responses.rs:982-1035`, `responses.rs:1435-1475`).
- Codex streaming tests can deterministically gate chunks. `StreamingSseServer` records request bodies and serves queued `/v1/responses` streams chunk-by-chunk, optionally waiting on per-chunk gates before writing bytes (`streaming_sse.rs:13-61`, `streaming_sse.rs:86-173`).
- Codex golden payload tests use redacted/structured snapshots rather than raw ad hoc strings. `context_snapshot` formats provider `input` items by item type, role, content kind, function calls, tool outputs, local shell calls, reasoning summaries, and compaction markers; model-visible layout tests feed real turns through the harness and `insta::assert_snapshot!` on labeled provider requests (`context_snapshot.rs:12-227`, `model_visible_layout.rs:81-200`).
- Codex prompt-debug is a no-inference model-visible payload harness. It creates an ephemeral thread, records context/user input, builds `prompt.input`, shuts the thread down, and the test asserts AGENTS instructions plus the user message are present (`prompt_debug.rs:24-102`, `prompt_debug_tests.rs:16-70`).
- Codex app-server tests run an actual child `codex-app-server` process over JSON-RPC stdio, isolate `CODEX_HOME`, disable plugin startup tasks by default, forward stderr for failures, initialize with client info/capabilities, and expose typed request helpers for thread/turn/config/process/plugin surfaces (`test_app_server.rs:117-360`, `test_app_server.rs:443-560`). The v2 suite enumerates endpoint-level coverage (`v2/mod.rs:1-80`).
- Codex smoke tests can cross process and analytics boundaries. `plugin_analytics_smoke` starts a loopback Responses server, spawns a stdio client with capture-file env, runs plugin install/config toggles and a model turn, waits for analytics JSONL, and validates expected plugin events (`plugin_analytics_smoke.rs:30-95`, `plugin_analytics_smoke.rs:97-157`).
- Codex rollout-trace tests verify replayable trajectory evidence. The writer creates a trace bundle, writes raw payload files before events, appends typed rollout/thread/turn/inference events, then `replay_bundle` checks reduced rollout status, thread ids, execution status, and raw payload refs (`writer.rs:31-141`, `writer.rs:170-265`). Inference tests verify UUID request headers, replayable inference attempts, raw request/response payload counts, and trace-only reasoning content preservation (`inference.rs:404-523`).
- Codex TUI tests use a VT100 backend and fixtures to test terminal output separately from model/runtime tests. `vt100_history` inserts history lines and checks wrapping, CJK/emoji, ANSI spans, cursor restoration, and word-wrap regressions; `status_indicator` checks ANSI escape sanitization (`tui/tests/all.rs:1-10`, `vt100_history.rs:22-164`, `status_indicator.rs:1-24`).
- Codex TypeScript SDK tests use a local HTTP Responses proxy that records requests and emits scripted SSE events. Tests assert streamed thread events, history continuation/resume payloads, and output schema request fields through the public SDK (`responsesProxy.ts:22-148`, `testCodex.ts:23-87`, `runStreamed.test.ts:14-200`).
- Codex apply-patch fixtures are portable golden filesystem tests: each scenario has `input/`, `patch.txt`, and `expected/`; the runner copies input into a temp dir, runs the `apply_patch` binary, snapshots actual and expected directory trees, and asserts exact final state (`apply-patch/tests/fixtures/scenarios/README.md:1-17`, `apply-patch/tests/suite/scenarios.rs:10-126`).

What the model sees:
- Claude model can see the `TestingPermission` tool only when the runtime is in test mode and the registry includes it; the prompt says it asks permission before execution and is used for end-to-end testing (`TestingPermissionTool.tsx:12-20`, `tools.ts:244`).
- Claude VCR observes normalized API-facing messages after filtering meta user messages and dehydration; VCR fixtures are evidence about model payloads, not additional model input (`services/vcr.ts:88-120`).
- Claude GrowthBook overrides, SWE-bench metadata, analytics suppression, and test dependency injection are not model-visible.
- Codex model sees whatever the harness sends through real turn submission or `build_prompt_input`. Provider request capture and `context_snapshot` format the model-visible provider payload for assertion, while `prompt_debug` builds the prompt input without sampling (`responses.rs:101-304`, `context_snapshot.rs:57-227`, `prompt_debug.rs:24-102`).
- Codex app-server and SDK tests can assert model-visible payloads by inspecting mock Responses requests; the user/app JSON-RPC or SDK events are separate from those provider requests (`app-server/tests/suite/v2/turn_start.rs:179-214`, `sdk/typescript/tests/runStreamed.test.ts:64-200`).

What the runtime does:
- Claude runtime reads/writes VCR fixtures when test/forced VCR mode is active, otherwise delegates to the live function; it blocks missing fixtures in CI unless recording is explicitly enabled (`services/vcr.ts:23-86`).
- Claude runtime exposes narrow dependency injection around `query()` and uses `NODE_ENV === 'test'` to alter specific behaviors such as test tool availability and analytics suppression (`query/deps.ts:8-40`, `tools.ts:244`, `analytics/config.ts:11-27`).
- Codex runtime tests exercise actual session/thread/turn machinery: builders create temp homes/cwds, configure mock provider base URLs, submit `Op::UserInput`, and await protocol events from a real `CodexThread` (`test_codex.rs:365-620`, `test_codex.rs:833-890`).
- Codex app-server tests exercise a child process over stdio JSON-RPC; SDK tests exercise the JavaScript wrapper against a local Responses proxy and the compiled Codex executable path (`app-server/tests/common/test_app_server.rs:203-276`, `sdk/typescript/tests/testCodex.ts:6-87`).
- Codex rollout-trace tests exercise writer/reducer behavior in temp dirs; trace writing is separate from provider mocks and app-server smoke tests (`writer.rs:170-265`, `inference.rs:443-493`).

What gets persisted:
- Claude VCR persists JSON fixture files under `CLAUDE_CODE_TEST_FIXTURES_ROOT ?? cwd` in a `fixtures/` directory, with filenames derived from content hashes (`services/vcr.ts:49-56`, `services/vcr.ts:112-115`). GrowthBook remote eval values may be cached to global config, and SWE-bench ids persist through analytics events (`growthbook.ts:397-417`, `metadata.ts:912-920`).
- Claude `TestingPermissionTool` itself does not persist state; it returns a tool result.
- Codex core/app-server tests create temporary homes, workspaces, state DBs, mock server request logs, provider request captures, and snapshot files. `INSTA_WORKSPACE_ROOT` is configured for snapshot tests (`common/lib.rs:56-74`).
- Codex rollout-trace persists trace bundles with manifest, raw JSONL events, payload files, and replay-derived rollout state during tests (`writer.rs:49-83`, `writer.rs:85-141`, `writer.rs:240-263`).
- Codex app-server smoke persists analytics JSONL to the capture file path, and apply-patch scenario tests persist temp working trees only for the duration of the test (`plugin_analytics_smoke.rs:30-95`, `scenarios.rs:30-63`).

What the user/TUI sees:
- Claude test-only permission tooling produces the normal permission dialog path by always returning `behavior: 'ask'`; its render methods return `null`, so the useful visible effect is the permission request, not a custom tool display (`TestingPermissionTool.tsx:36-60`).
- Claude VCR, feature overrides, and SWE-bench metadata are not ordinary TUI surfaces.
- Codex core tests observe protocol `EventMsg` values rather than TUI rows; app-server tests observe JSON-RPC responses/notifications/errors; TUI tests observe VT100 terminal contents; SDK tests observe public `ThreadEvent` streams (`common/lib.rs:256-353`, `test_app_server.rs:279-360`, `vt100_history.rs:41-164`, `runStreamed.test.ts:14-62`).
- Codex plugin analytics smoke prints a validation summary and capture-file path after successful validation (`plugin_analytics_smoke.rs:87-94`).

What debug/eval surfaces exist:
- Claude eval/debug surfaces for this mechanism are source-specific: VCR fixtures, `TestingPermission` tool, `QueryDeps`, `QueryConfig`, GrowthBook eval overrides, SWE-bench analytics fields, and test-mode analytics suppression.
- Claude source comments identify fixture-hash fragility as an eval constraint: query/tool/message transformations are guarded so resume, SDK harness tests, or VCR lookups do not drift (`query.ts:765-770`, `toolExecution.ts:775-782`, `messages.ts:2345-2351`, `messages.ts:2414-2421`).
- Codex eval/debug surfaces include core integration suites, provider mock request capture, ordered/gated SSE, context snapshots, `insta` snapshots, prompt-debug tests, app-server JSON-RPC suites, app-server smoke client, rollout-trace replay tests, TUI VT100 tests, SDK SSE proxy tests, and portable apply-patch fixtures.
- Codex source contains large snapshot directories under `codex-rs/core/tests/suite/snapshots/`; discovery showed snapshots for additional context, compaction, model-visible layout, pending input, realtime conversation, and token budget.

Shared invariant:
- A durable harness separates model-visible provider payload assertions, runtime event assertions, persisted artifact assertions, terminal/TUI assertions, debug/trace replay assertions, and smoke tests that cross process boundaries.
- Golden evidence should be generated from structured runtime artifacts, not from invented prose: Claude hashes normalized API payloads into VCR fixtures; Codex captures provider JSON and formats it through `context_snapshot`, records rollout traces through typed payload refs, and compares final filesystem state for patch fixtures.
- Determinism belongs in explicit seams: dependency injection, isolated temp homes/cwds, mock provider servers, ordered SSE scripts, deterministic process ids, env-gated feature overrides, and snapshot workspace-root setup.
- Source-specific mechanisms must stay source-specific. Claude's provided `src` does not expose the same first-class test tree as Codex; Codex's multi-crate test harness must not be projected back onto Claude.

Claude-specific design:
- Claude-specific eval support is VCR-centered and hook-centered in the provided source: fixture hashing/replay, a test-only permission tool, query dependency injection, test-mode analytics suppression, and eval feature-flag overrides.
- Claude-specific fixture safety treats message normalization changes as potentially test-breaking; comments explicitly mention VCR fixture hashes and SDK harness tests.
- Claude-specific SWE-bench support is analytics metadata, not a full local SWE-bench runner in the provided `src`.
- Claude-specific `CLAUDE_INTERNAL_FC_OVERRIDES` is ant-only and deterministic because env overrides precede local config, remote eval, and disk cache.

Codex-specific design:
- Codex-specific harness architecture spans crates and surfaces: Rust core integration tests, app-server stdio JSON-RPC tests, executable smoke clients, rollout-trace replay tests, TUI VT100 tests, TypeScript SDK tests, and portable apply-patch filesystem fixtures.
- Codex-specific provider mocking is structured around `ResponseMock`, `ResponsesRequest`, ordered `mount_sse_sequence`, and gated `StreamingSseServer`.
- Codex-specific golden payloads are often normalized snapshots of provider `input` items, not raw request dumps.
- Codex-specific trajectory traces are first-class local bundles with raw event logs, raw payload refs, and replay/reducer tests.

Gaps or unknowns:
- Claude tests outside the provided `src` tree were not examined because the reference source for Claude was restricted to `claude-code-source-code/src`.
- Claude runtime traces were not generated live; evidence is source-derived from VCR/test/eval hooks and comments.
- Codex suites are large; this card cites representative harness entrypoints and proof surfaces, not every individual test file.
- Mechanism 29 product-name/runtime hygiene and mechanism 30 final skill packaging are intentionally not covered here.

Distilled harness rule:
- Build eval coverage as layered, source-grounded proof: provider-payload fixtures/snapshots, runtime event assertions, persisted artifact comparisons, terminal/TUI output tests, trace replay/reduction, package/API contract tests, and process-level smoke tests. For each layer, record the exact source harness, how inputs are scripted, what artifact is captured, what comparison is made, and which surface it proves.

Anti-invention rule:
- Do not invent a test strategy or call it "best practice" unless the reference source implements it. If using Claude evidence, cite VCR, `TestingPermissionTool`, `QueryDeps`, GrowthBook overrides, SWE-bench metadata, and fixture-hash guards as Claude-specific. If using Codex evidence, cite the concrete test harness files, mock server helpers, snapshots, trace replay tests, TUI tests, SDK tests, smoke client, and apply-patch fixtures.

Verification pattern:
- For a Claude-style harness, verify: `NODE_ENV === 'test'` includes `TestingPermissionTool`; calling it requests permission and returns a `tool_result`; `QueryDeps` can inject model/compact/uuid fakes; VCR fixture paths are derived from normalized/dehydrated model input; CI fails on missing fixtures without `VCR_RECORD`; path normalization keeps fixture hashes stable; message/tool backfill does not mutate transcripts in ways comments prohibit; `CLAUDE_INTERNAL_FC_OVERRIDES` wins over config/cache/remote eval; SWE-bench env vars enter analytics metadata.
- For a Codex-style harness, verify: core tests use isolated temp homes/cwds and mock provider base URLs; ordered/gated SSE scripts produce expected `EventMsg` sequences; provider request capture asserts instructions/input/tool/result fields; `context_snapshot` snapshots model-visible layout; `prompt_debug` builds prompt input without inference; app-server tests complete JSON-RPC initialize/request/notification loops against a child process; smoke tests validate analytics artifacts across process boundaries; rollout-trace writer tests replay bundles and raw payload refs; TUI tests assert VT100 output; SDK tests assert public streamed events and captured provider payloads; apply-patch fixtures compare exact final filesystem state.

Skill text candidate:
- When extracting eval design from a coding-agent harness, start by identifying which surface each test proves: model-visible provider payload, runtime event stream, persisted state/artifact, terminal/TUI output, debug/trace replay, package API, or process-level smoke. Then cite the exact test harness file, mock/server/fixture builder, scripted input, captured artifact, comparison method, and determinism gate. Keep Claude's source-limited VCR/test-hook design separate from Codex's first-class multi-crate harness unless both sources demonstrate the same invariant.
