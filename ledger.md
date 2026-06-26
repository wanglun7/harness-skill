# Coding-Agent Harness Distillation Ledger

Goal: distill source-grounded harness mechanisms from the two reference codebases into reusable skill material, in strict mechanism order.

Reference sources:
- Claude Code: `/Users/lun/Desktop/manifex/collection-claude-code-source-code/claude-code-source-code/src`
- Codex: `/Users/lun/Desktop/manifex/collection-claude-code-source-code/openai-codex`

Rules snapshot:
- Do not implement runtime changes.
- Do not generalize source-specific mechanisms.
- Re-read this ledger and relevant source before each work cycle.
- Do not proceed to the next mechanism until the current mechanism card is complete.
- Keep model-visible, runtime, persisted, TUI/display, debug/trace, and eval surfaces separate.

## Work Cycle Log

### 2026-06-25 Cycle 1

Before:
- Ledger read result: no existing ledger or card files were present in this workspace.
- Selected mechanism: 1. Repository/source map and harness boundary definition.

After:
- Completed `cards/01-repository-source-map-and-harness-boundary.md`.
- Mechanism 1 source citations recorded below.
- Next incomplete mechanism: 2. Model/provider abstraction.

### 2026-06-25 Cycle 2

Before:
- Ledger read result: mechanism 1 complete; mechanism 2 is the next incomplete mechanism.
- Selected mechanism: 2. Model/provider abstraction.
- Assumption: "provider abstraction" means source-grounded mechanisms for choosing/configuring model providers, carrying model/provider identity into sampling, and streaming responses through provider/client boundaries. It does not include full payload construction details, which belong to mechanism 5.

After:
- Completed `cards/02-model-provider-abstraction.md`.
- Mechanism 2 source citations recorded below.
- Next incomplete mechanism: 3. Main turn loop and stop conditions.

### 2026-06-25 Cycle 3

Before:
- Ledger read result: mechanisms 1 and 2 complete; mechanism 3 is the next incomplete mechanism.
- Selected mechanism: 3. Main turn loop and stop conditions.
- Assumption: this mechanism covers loop state, continuation decisions, terminal reasons, abort/interruption handling, and user-visible completion/failure surfaces. It does not cover detailed streaming text/tool interleaving, which belongs to mechanism 4.

After:
- Completed `cards/03-main-turn-loop-and-stop-conditions.md`.
- Mechanism 3 source citations recorded below.
- Next incomplete mechanism: 4. Streaming protocol and assistant text/tool interleaving.

### 2026-06-25 Cycle 4

Before:
- Ledger read result: mechanisms 1, 2, and 3 complete; mechanism 4 is the next incomplete mechanism.
- Selected mechanism: 4. Streaming protocol and assistant text/tool interleaving.
- Assumption: this mechanism covers provider/runtime streaming events, assistant text deltas, reasoning/text block completion, tool-call start/input streaming, and the boundary between streamed display events and completed prompt/history items. It does not cover provider payload construction, which belongs to mechanism 5, or tool dispatch scheduling, which belongs to mechanism 11.

After:
- Completed `cards/04-streaming-protocol-and-assistant-text-tool-interleaving.md`.
- Mechanism 4 source citations recorded below.
- Next incomplete mechanism: 5. Message normalization and provider payload construction.

### 2026-06-25 Cycle 5

Before:
- Ledger read result: mechanisms 1, 2, 3, and 4 complete; mechanism 5 is the next incomplete mechanism.
- Selected mechanism: 5. Message normalization and provider payload construction.
- Assumption: this mechanism covers conversion from internal transcript/history messages into provider request payloads, provider-specific payload fields, and API request capture/logging boundaries. It does not cover system prompt content/layering beyond where it is passed into the provider payload; that belongs to mechanism 6.

After:
- Completed `cards/05-message-normalization-and-provider-payload-construction.md`.
- Mechanism 5 source citations recorded below.
- Next incomplete mechanism: 6. System prompt and instruction layering.

### 2026-06-25 Cycle 6

Before:
- Ledger read result: mechanisms 1, 2, 3, 4, and 5 complete; mechanism 6 is the next incomplete mechanism.
- Selected mechanism: 6. System prompt and instruction layering.
- Assumption: this mechanism covers ordered instruction layers and how they enter provider `system`/`instructions` fields. It does not cover dynamic context discovery/assembly, token budgeting, skills/plugins, or memory extraction beyond where those layers are passed into the prompt; those belong to later mechanisms.

After:
- Completed `cards/06-system-prompt-and-instruction-layering.md`.
- Mechanism 6 source citations recorded below.
- Next incomplete mechanism: 7. Dynamic context assembly.

### 2026-06-25 Cycle 7

Before:
- Ledger read result: mechanisms 1, 2, 3, 4, 5, and 6 complete; mechanism 7 is the next incomplete mechanism.
- Selected mechanism: 7. Dynamic context assembly.
- Assumption: this mechanism covers per-turn dynamic context collection/assembly and how that context becomes provider-visible prompt, system, user, or context items. It does not cover token budgeting/context limits, compaction, full tool registry projection, memory extraction lifecycle, or detailed skills/plugins/custom-instructions loading, which belong to later mechanisms.

After:
- Completed `cards/07-dynamic-context-assembly.md`.
- Mechanism 7 source citations recorded below.
- Next incomplete mechanism: 8. Token budgeting and context limits.

### 2026-06-25 Cycle 8

Before:
- Ledger read result: mechanisms 1, 2, 3, 4, 5, 6, and 7 complete; mechanism 8 is the next incomplete mechanism.
- Selected mechanism: 8. Token budgeting and context limits.
- Assumption: this mechanism covers context-window sizing, token estimation/counting, prompt-too-long/blocking checks, token-budget prompt/status surfaces, and telemetry/debug surfaces. It does not cover compaction lifecycle details beyond the limit checks that trigger or inform compaction; compaction belongs to mechanism 9.

After:
- Completed `cards/08-token-budgeting-and-context-limits.md`.
- Mechanism 8 source citations recorded below.
- Next incomplete mechanism: 9. Compaction: manual, auto, reactive, post-compact rehydration.

### 2026-06-25 Cycle 9

Before:
- Ledger read result: mechanisms 1 through 8 complete; mechanism 9 is the next incomplete mechanism.
- Selected mechanism: 9. Compaction: manual, auto, reactive, post-compact rehydration.
- Assumption: this mechanism covers manual compaction entry points, automatic and reactive compaction triggers, summary/replacement-history construction, post-compact context rehydration, persisted compact boundaries, and user/debug surfaces. It does not re-explain token-limit calculations, which belong to mechanism 8.

After:
- Completed `cards/09-compaction-manual-auto-reactive-post-compact-rehydration.md`.
- Mechanism 9 source citations recorded below.
- Next incomplete mechanism: 10. Tool registry and model-visible schema projection.

### 2026-06-25 Cycle 10

Before:
- Ledger read result: mechanisms 1 through 9 complete; mechanism 10 is the next incomplete mechanism.
- Selected mechanism: 10. Tool registry and model-visible schema projection.
- Assumption: this mechanism covers how tools are registered, filtered, named, described, schema-projected, and attached to provider/model requests. It does not cover tool execution scheduling/concurrency, tool result projection, or individual file/shell/web tool behavior, which belong to later mechanisms.

After:
- Completed `cards/10-tool-registry-and-model-visible-schema-projection.md`.
- Mechanism 10 source citations recorded below.
- Next incomplete mechanism: 11. Tool dispatch lifecycle and scheduling/concurrency.

### 2026-06-25 Cycle 11

Before:
- Ledger read result: mechanisms 1 through 10 complete; mechanism 11 is the next incomplete mechanism.
- Selected mechanism: 11. Tool dispatch lifecycle and scheduling/concurrency.
- Assumption: this mechanism covers tool-call recognition from model output, validation/permission hooks as part of dispatch, lifecycle notifications/progress, concurrency/scheduling, cancellation/interrupt handling, and return of tool execution items to the turn loop. It does not cover result-shape projection details, individual file/shell/web semantics, or permission-policy taxonomy beyond dispatch gates, which belong to later mechanisms.

After:
- Completed `cards/11-tool-dispatch-lifecycle-and-scheduling-concurrency.md`.
- Mechanism 11 source citations recorded below.
- Next incomplete mechanism: 12. Tool result projection: raw output, model output, display output, persisted output.

### 2026-06-25 Cycle 12

Before:
- Ledger read result: mechanisms 1 through 11 complete; mechanism 12 is the next incomplete mechanism.
- Selected mechanism: 12. Tool result projection: raw output, model output, display output, persisted output.
- Assumption: this mechanism covers how a completed tool's raw/runtime output is converted into model-visible payloads, user/TUI display items, transcript/history items, attachments, truncation or disk persistence, and debug/eval trace payloads. It does not cover dispatch scheduling, individual file/shell/web semantics, or permission-policy decisions, which belong to other mechanisms.

After:
- Completed `cards/12-tool-result-projection-raw-output-model-output-display-output-persisted-output.md`.
- Mechanism 12 source citations recorded below.
- Next incomplete mechanism: 13. File read/search tools.

### 2026-06-25 Cycle 13

Before:
- Ledger read result: mechanisms 1 through 12 complete; mechanism 13 is the next incomplete mechanism.
- Selected mechanism: 13. File read/search tools.
- Assumption: this mechanism covers model-visible read/list/search tool contracts, path validation, ignore/symlink/binary/size behavior, runtime file-system operations, output shaping, display summaries, and source-backed tests for read/search tools. It does not cover file write/edit/patch/diff behavior, shell-command search via general exec, sandbox policy taxonomy, or web/network search, which belong to later mechanisms.

After:
- Completed `cards/13-file-read-search-tools.md`.
- Mechanism 13 source citations recorded below.
- Next incomplete mechanism: 14. File edit/write/patch/diff tools.

### 2026-06-25 Cycle 14

Before:
- Ledger read result: mechanisms 1 through 13 complete; mechanism 14 is the next incomplete mechanism.
- Selected mechanism: 14. File edit/write/patch/diff tools.
- Assumption: this mechanism covers model-visible file creation/replacement/edit/patch contracts, diff generation and display, pre/post edit validation, permission/sandbox checks, persisted edit state, and debug/eval evidence for edit/write/patch/diff tools. It does not cover shell execution, worktree isolation, broad permission policy taxonomy, or session persistence beyond edit-related state and transcript outputs, which belong to later mechanisms.

After:
- Completed `cards/14-file-edit-write-patch-diff-tools.md`.
- Mechanism 14 source citations recorded below.
- Next incomplete mechanism: 15. Shell/exec/background process/interrupt tools.

### 2026-06-25 Cycle 15

Before:
- Ledger read result: mechanisms 1 through 14 complete; mechanism 15 is the next incomplete mechanism.
- Selected mechanism: 15. Shell/exec/background process/interrupt tools.
- Assumption: this mechanism covers model-visible shell/command execution contracts, command permission/sandbox gating where tied directly to execution, streaming stdout/stderr/progress, timeout and long-running/background behavior, explicit interrupt/cancel behavior, persisted process/output state, terminal/TUI display, and debug/eval surfaces. It does not cover general permission-policy taxonomy, worktree/isolation design, web/network tools, or task/todo state beyond shell execution, which belong to later mechanisms.

After:
- Completed `cards/15-shell-exec-background-process-interrupt-tools.md`.
- Mechanism 15 source citations recorded below.
- Next incomplete mechanism: 16. Web/network tools.

### 2026-06-25 Cycle 16

Before:
- Ledger read result: mechanisms 1 through 15 complete; mechanism 16 is the next incomplete mechanism.
- Selected mechanism: 16. Web/network tools.
- Assumption: this mechanism covers model-visible web fetch/search/browser/network-hosted tools, their runtime fetch/search/client paths, network permission surfaces directly tied to those tools, output projection, UI/debug/test surfaces, and source-specific hosted-provider web search behavior. It does not cover general shell network sandbox policy, broad permission taxonomy, MCP resource tools, or browser/TUI interaction beyond web tool surfaces, which belong to later mechanisms.

After:
- Completed `cards/16-web-network-tools.md`.
- Mechanism 16 source citations recorded below.
- Next incomplete mechanism: 17. Task/todo/plan tools and task state.

### 2026-06-25 Cycle 17

Before:
- Ledger read result: mechanisms 1 through 16 complete; mechanism 17 is the next incomplete mechanism.
- Selected mechanism: 17. Task/todo/plan tools and task state.
- Assumption: this mechanism covers model-visible todo/task/plan tools, explicit task-state data structures, status update lifecycle, persistence/display of task or plan state, and debug/eval evidence. It does not cover subagent/fork/delegation, background shell process state, memory extraction, or session transcript persistence beyond task/plan-specific state, which belong to other mechanisms.

After:
- Completed `cards/17-task-todo-plan-tools-and-task-state.md`.
- Mechanism 17 source citations recorded below.
- Next incomplete mechanism: 18. Subagent/fork/delegation model.

### 2026-06-25 Cycle 18

Before:
- Ledger read result: mechanisms 1 through 17 complete; mechanism 18 is the next incomplete mechanism.
- Selected mechanism: 18. Subagent/fork/delegation model.
- Assumption: this mechanism covers model-visible delegation/fork/subagent tools and runtime lifecycle for delegated agents or forked threads. It does not cover worktree/sandbox isolation, inbox/notifications, or full transcript resume/repair beyond delegation-specific handoff state, which belong to mechanisms 19, 20, and 23.

After:
- Completed `cards/18-subagent-fork-delegation-model.md`.
- Mechanism 18 source citations recorded below.
- Next incomplete mechanism: 19. Worktree/isolation/sandbox behavior.

### 2026-06-25 Cycle 19

Before:
- Ledger read result: mechanisms 1 through 18 complete; mechanism 19 is the next incomplete mechanism.
- Selected mechanism: 19. Worktree/isolation/sandbox behavior.
- Assumption: this mechanism covers runtime workspace/worktree isolation, cwd/workspace-root selection, filesystem sandbox configuration, sandbox execution policy, and model-visible/user-visible surfaces that expose or constrain isolation. It does not cover permission approval policy details beyond sandbox/isolation inputs, which belong to mechanism 24.

After:
- Completed `cards/19-worktree-isolation-sandbox-behavior.md`.
- Mechanism 19 source citations recorded below.
- Next incomplete mechanism: 20. Agent messaging/inbox/notifications.

### 2026-06-25 Cycle 20

Before:
- Ledger read result: mechanisms 1 through 19 complete; mechanism 20 is the next incomplete mechanism.
- Selected mechanism: 20. Agent messaging/inbox/notifications.
- Assumption: this mechanism covers agent-to-agent or parent-child message delivery, inbox/mailbox queues, notifications and completion delivery, user/TUI surfacing, and persisted message/event records. It does not re-explain subagent spawn/fork lifecycle from mechanism 18 or general session transcript persistence from mechanism 23 except where messaging evidence directly writes or reconstructs inbox/notification items.

After:
- Completed `cards/20-agent-messaging-inbox-notifications.md`.
- Mechanism 20 source citations recorded below.
- Next incomplete mechanism: 21. Memory: session, project, long-term, extraction lifecycle.

### 2026-06-25 Cycle 21

Before:
- Ledger read result: mechanisms 1 through 20 complete; mechanism 21 is the next incomplete mechanism.
- Selected mechanism: 21. Memory: session, project, long-term, extraction lifecycle.
- Assumption: this mechanism covers explicit memory surfaces across session/project/long-term storage, model-visible memory/context injection, memory creation or extraction flows, persistence/replay, user/TUI controls, debug/eval surfaces, and source-specific absence where one reference lacks a comparable lifecycle. It does not cover general transcript persistence/resume except where memory records directly participate; that belongs to mechanism 23.

After:
- Completed `cards/21-memory-session-project-long-term-extraction-lifecycle.md`.
- Mechanism 21 source citations recorded below.
- Next incomplete mechanism: 22. Skills/plugins/custom instructions.

### 2026-06-25 Cycle 22

Before:
- Ledger read result: mechanisms 1 through 21 complete; mechanism 22 is the next incomplete mechanism.
- Selected mechanism: 22. Skills/plugins/custom instructions.
- Assumption: this mechanism covers skills/plugins/custom-instruction discovery, gating, prompt injection, tool/context contribution, model-visible projection, persisted configuration/cache state, user/TUI controls, and debug/eval surfaces. It does not cover general session persistence/resume except where skill/plugin/custom-instruction state directly participates; that belongs to mechanism 23.

After:
- Completed `cards/22-skills-plugins-custom-instructions.md`.
- Mechanism 22 source citations recorded below.
- Next incomplete mechanism: 23. Session persistence, transcript storage, resume, repair.

### 2026-06-25 Cycle 23

Before:
- Ledger read result: mechanisms 1 through 22 complete; mechanism 23 is the next incomplete mechanism.
- Selected mechanism: 23. Session persistence, transcript storage, resume, repair.
- Assumption: this mechanism covers transcript/session identifiers, message/event persistence, conversation reconstruction, resume/continue entrypoints, repair/backfill/migration behavior, persisted sidechain/subagent transcript state, user/TUI session surfaces, and debug/eval surfaces. It does not cover compaction itself except where compacted transcripts are persisted or used during resume; compaction mechanics were covered in mechanism 9.

After:
- Completed `cards/23-session-persistence-transcript-storage-resume-repair.md`.
- Mechanism 23 source citations recorded below.
- Next incomplete mechanism: 24. Permissions, approvals, policy enforcement.

### 2026-06-25 Cycle 24

Before:
- Ledger read result: mechanisms 1 through 23 complete; mechanism 24 is the next incomplete mechanism.
- Selected mechanism: 24. Permissions, approvals, policy enforcement.
- Assumption: this mechanism covers runtime permission modes/profiles, approval prompts and decisions, policy/config gates, persisted allow/deny state, model-visible permission guidance, user/TUI approval surfaces, and debug/eval evidence. It does not re-explain sandbox/worktree isolation except where permission policy directly selects or enforces sandbox behavior; isolation mechanics were covered in mechanism 19.

After:
- Completed `cards/24-permissions-approvals-policy-enforcement.md`.
- Mechanism 24 source citations recorded below.
- Next incomplete mechanism: 25. Error taxonomy and recovery/retry behavior.

### 2026-06-25 Cycle 25

Before:
- Ledger read result: mechanisms 1 through 24 complete; mechanism 25 is the next incomplete mechanism.
- Selected mechanism: 25. Error taxonomy and recovery/retry behavior.
- Assumption: this mechanism covers typed error categories, provider/API error parsing, retry/backoff/fallback behavior, user abort/interruption handling, tool/runtime error projection, conversation/session recovery after errors, user/TUI error display, and debug/eval evidence. It does not re-explain session transcript repair from mechanism 23 except where an error path directly triggers recovery or retry.

After:
- Completed `cards/25-error-taxonomy-and-recovery-retry-behavior.md`.
- Mechanism 25 source citations recorded below.
- Next incomplete mechanism: 26. Debug logs, trace bundles, model payload capture.

### 2026-06-25 Cycle 26

Before:
- Ledger read result: mechanisms 1 through 25 complete; mechanism 26 is the next incomplete mechanism.
- Selected mechanism: 26. Debug logs, trace bundles, model payload capture.
- Assumption: this mechanism covers debug logging APIs, structured telemetry/log events, trace bundle/file generation, provider/model payload capture or redaction, request/response tracing, tool/turn trace surfaces, user-accessible debug exports, and eval-visible trace artifacts. It does not cover general error taxonomy from mechanism 25 or eval harness design from mechanism 28 except where trace artifacts directly feed debugging or payload capture.

After:
- Completed `cards/26-debug-logs-trace-bundles-model-payload-capture.md`.
- Mechanism 26 source citations recorded below.
- Next incomplete mechanism: 27. TUI rendering and interaction model.

### 2026-06-25 Cycle 27

Before:
- Ledger read result: mechanisms 1 through 26 complete; mechanism 27 is the next incomplete mechanism.
- Selected mechanism: 27. TUI rendering and interaction model.
- Assumption: this mechanism covers terminal/TUI application entrypoints, event-to-render pipelines, message/history cell rendering, input composer/keybinding/interrupt behavior, layout/status/footer surfaces, streaming display updates, approval/permission UI as display interaction, and debug/eval surfaces for rendering. It does not re-explain tool runtime semantics, persistence, permissions policy, or eval harness internals except where those mechanisms directly feed TUI rendering.

After:
- Completed `cards/27-tui-rendering-and-interaction-model.md`.
- Mechanism 27 source citations recorded below.
- Next incomplete mechanism: 28. Eval harness, smoke tests, golden payloads, trajectory traces.

### 2026-06-25 Cycle 28

Before:
- Ledger read result: mechanisms 1 through 27 complete; mechanism 28 is the next incomplete mechanism.
- Selected mechanism: 28. Eval harness, smoke tests, golden payloads, trajectory traces.
- Assumption: this mechanism covers source-defined test/eval harnesses, smoke or replay entrypoints, golden/snapshot expectations, fixtures/payloads, trajectory or rollout trace-based validation, and how eval artifacts relate to model-visible/runtime/persisted/TUI/debug surfaces. It does not cover product-name hygiene or final skill packaging, which belong to mechanisms 29 and 30.

After:
- Completed `cards/28-eval-harness-smoke-tests-golden-payloads-trajectory-traces.md`.
- Mechanism 28 source citations recorded below.
- Next incomplete mechanism: 29. Runtime hygiene and product-name leakage prevention.

### 2026-06-25 Cycle 29

Before:
- Ledger read result: mechanisms 1 through 28 complete; mechanism 29 is the next incomplete mechanism.
- Selected mechanism: 29. Runtime hygiene and product-name leakage prevention.
- Assumption: this mechanism covers source-grounded controls that keep runtime surfaces clean, deterministic, non-leaky, or product/brand-safe across model-visible prompts, telemetry/debug payloads, persisted files, user/TUI text, generated titles/summaries, update/version surfaces, and internal-only/provider-specific naming. It does not cover final reusable skill packaging, which belongs to mechanism 30.

After:
- Completed `cards/29-runtime-hygiene-and-product-name-leakage-prevention.md`.
- Mechanism 29 source citations recorded below.
- Next incomplete mechanism: 30. Final skill packaging and usage workflow.

### 2026-06-25 Cycle 30

Before:
- Ledger read result: mechanisms 1 through 29 complete; mechanism 30 is the next incomplete mechanism.
- Selected mechanism: 30. Final skill packaging and usage workflow.
- Assumption: this mechanism covers packaging the completed source-grounded mechanism cards into a reusable `SKILL.md` with progressive-disclosure instructions, strict-order workflow, source-citation discipline, validation/audit steps, and usage rules. It does not add runtime behavior to either reference source and does not introduce target-implementation dependencies.

After:
- Completed `cards/30-final-skill-packaging-and-usage-workflow.md`.
- Completed final root `SKILL.md`.
- Mechanism 30 source citations recorded below.
- Structural validation passed for 30 mechanism cards, required card sections, final `SKILL.md` frontmatter, final `SKILL.md` workflow sections, and mechanism order.
- `quick_validate.py` could not run because both system and bundled Python lacked the `yaml` module; manual validation covered the same frontmatter/structure constraints needed for this artifact.
- Next incomplete mechanism: none.

## Mechanism Status

| # | Mechanism | Status | Card | Key citations |
|---:|---|---|---|---|
| 1 | Repository/source map and harness boundary definition | complete | `cards/01-repository-source-map-and-harness-boundary.md` | Claude: `src/entrypoints/cli.tsx:28-33`, `src/QueryEngine.ts:175-184`, `src/QueryEngine.ts:1179-1186`, `src/query.ts:181-239`, `src/Tool.ts:697-701`; Codex: `codex-rs/cli/src/main.rs:90-120`, `codex-rs/core/src/lib.rs:1-18`, `codex-rs/core/src/session/session.rs:23-47`, `codex-rs/core/src/session/turn.rs:140-245` |
| 2 | Model/provider abstraction | complete | `cards/02-model-provider-abstraction.md` | Claude: `src/utils/model/providers.ts:4-14`, `src/utils/model/modelStrings.ts:17-31`, `src/services/api/client.ts:88-315`, `src/query.ts:570-705`; Codex: `codex-rs/model-provider-info/src/lib.rs:82-137`, `codex-rs/model-provider/src/provider.rs:94-180`, `codex-rs/core/src/client.rs:208-245`, `codex-rs/core/src/client.rs:1610-1668` |
| 3 | Main turn loop and stop conditions | complete | `cards/03-main-turn-loop-and-stop-conditions.md` | Claude: `src/query.ts:181-238`, `src/query.ts:241-321`, `src/query.ts:554-705`, `src/query.ts:1011-1357`, `src/query.ts:1366-1728`, `src/query/stopHooks.ts:60-80`, `src/query/stopHooks.ts:257-455`, `src/QueryEngine.ts:675-875`, `src/QueryEngine.ts:943-1155`; Codex: `codex-rs/core/src/session/turn.rs:140-414`, `codex-rs/core/src/session/turn.rs:1283-1287`, `codex-rs/core/src/session/turn.rs:1849-2336`, `codex-rs/core/src/tasks/mod.rs:210-232`, `codex-rs/core/src/tasks/mod.rs:308-425`, `codex-rs/core/src/tasks/mod.rs:486-519`, `codex-rs/core/src/tasks/mod.rs:731-868`, `codex-rs/protocol/src/protocol.rs:1933-1948`, `codex-rs/protocol/src/protocol.rs:3864-3884` |
| 4 | Streaming protocol and assistant text/tool interleaving | complete | `cards/04-streaming-protocol-and-assistant-text-tool-interleaving.md` | Claude: `src/services/api/claude.ts:1761-1770`, `src/services/api/claude.ts:1979-2304`, `src/query.ts:742-862`, `src/query.ts:1366-1408`, `src/services/tools/StreamingToolExecutor.ts:19-40`, `src/services/tools/StreamingToolExecutor.ts:73-151`, `src/services/tools/StreamingToolExecutor.ts:153-230`, `src/services/tools/StreamingToolExecutor.ts:320-490`, `src/QueryEngine.ts:720-828`, `src/utils/queryHelpers.ts:102-222`; Codex: `codex-rs/codex-api/src/common.rs:72-105`, `codex-rs/codex-api/src/sse/responses.rs:298-418`, `codex-rs/core/src/session/turn.rs:1544-1642`, `codex-rs/core/src/session/turn.rs:1956-2288`, `codex-rs/core/src/stream_events_utils.rs:190-244`, `codex-rs/core/src/stream_events_utils.rs:303-315`, `codex-rs/core/src/stream_events_utils.rs:404-586`, `codex-rs/core/src/tools/registry.rs:140-157`, `codex-rs/core/src/tools/handlers/apply_patch.rs:71-132`, `codex-rs/core/src/session/mod.rs:1831-1880` |
| 5 | Message normalization and provider payload construction | complete | `cards/05-message-normalization-and-provider-payload-construction.md` | Claude: `src/query.ts:449-451`, `src/query.ts:659-708`, `src/utils/messages.ts:1989-2369`, `src/utils/messages.ts:2372-2515`, `src/services/api/claude.ts:1248-1345`, `src/services/api/claude.ts:1538-1836`, `src/utils/api.ts:119-265`, `src/utils/api.ts:437-474`; Codex: `codex-rs/protocol/src/models.rs:804-857`, `codex-rs/protocol/src/models.rs:1420-1470`, `codex-rs/protocol/src/models.rs:1539-1595`, `codex-rs/core/src/session/mod.rs:2642-2670`, `codex-rs/core/src/context_manager/history.rs:90-114`, `codex-rs/core/src/context_manager/history.rs:323-336`, `codex-rs/core/src/context_manager/normalize.rs:14-192`, `codex-rs/core/src/context_manager/normalize.rs:291-340`, `codex-rs/core/src/session/turn.rs:221-245`, `codex-rs/core/src/session/turn.rs:1016-1034`, `codex-rs/core/src/client_common.rs:17-106`, `codex-rs/core/src/client.rs:780-830`, `codex-rs/core/src/client.rs:1302-1320`, `codex-rs/core/src/client.rs:1407-1486`, `codex-rs/core/src/client.rs:1610-1668`, `codex-rs/codex-api/src/common.rs:182-253`, `codex-rs/codex-api/src/endpoint/responses.rs:118-172`, `codex-rs/rollout-trace/src/inference.rs:169-190` |
| 6 | System prompt and instruction layering | complete | `cards/06-system-prompt-and-instruction-layering.md` | Claude: `src/utils/systemPrompt.ts:28-122`, `src/constants/prompts.ts:105-115`, `src/constants/prompts.ts:444-576`, `src/constants/systemPromptSections.ts:16-67`, `src/screens/REPL.tsx:2534-2544`, `src/screens/REPL.tsx:2770-2788`, `src/QueryEngine.ts:284-325`, `src/query.ts:449-451`, `src/utils/api.ts:296-447`, `src/constants/system.ts:10-95`, `src/services/api/claude.ts:1347-1380`, `src/services/api/claude.ts:3213-3237`, `src/entrypoints/cli.tsx:50-69`, `src/utils/analyzeContext.ts:936-955`; Codex: `codex-rs/protocol/src/models.rs:1196-1211`, `codex-rs/protocol/src/openai_models.rs:350-530`, `codex-rs/core/src/config/mod.rs:669-673`, `codex-rs/core/src/config/mod.rs:3331-3345`, `codex-rs/core/src/config/mod.rs:3551-3559`, `codex-rs/core/src/session/session.rs:50-69`, `codex-rs/core/src/session/session.rs:528-546`, `codex-rs/protocol/src/protocol.rs:2863-2898`, `codex-rs/core/src/session/turn_context.rs:107-140`, `codex-rs/core/src/session/turn_context.rs:397-420`, `codex-rs/core/src/session/turn_context.rs:582-615`, `codex-rs/core/src/session/mod.rs:1196-1201`, `codex-rs/core/src/session/mod.rs:2845-3101`, `codex-rs/core/src/session/mod.rs:3180-3216`, `codex-rs/core/src/context_manager/updates.rs:183-208`, `codex-rs/core/src/session/turn.rs:1016-1034`, `codex-rs/core/src/client.rs:780-830`, `codex-rs/core/src/session/config_lock.rs:107-114`, `codex-rs/core/src/prompt_debug.rs:74-101` |
| 7 | Dynamic context assembly | complete | `cards/07-dynamic-context-assembly.md` | Claude: `src/context.ts:36-188`, `src/utils/claudemd.ts:1-26`, `src/utils/queryContext.ts:30-74`, `src/screens/REPL.tsx:2533-2544`, `src/screens/REPL.tsx:2767-2800`, `src/screens/REPL.tsx:3797-3810`, `src/QueryEngine.ts:284-325`, `src/query.ts:449-451`, `src/query.ts:659-661`, `src/utils/api.ts:437-474`, `src/utils/api.ts:479-499`, `src/utils/analyzeContext.ts:937-955`; Codex: `codex-rs/core/src/session/turn_context.rs:105-150`, `codex-rs/core/src/session/turn_context.rs:397-420`, `codex-rs/core/src/session/turn_context.rs:575-615`, `codex-rs/core/src/session/mod.rs:2845-3101`, `codex-rs/core/src/session/mod.rs:3172-3216`, `codex-rs/core/src/context_manager/updates.rs:21-40`, `codex-rs/core/src/context_manager/updates.rs:183-244`, `codex-rs/core/src/context/environment_context.rs:19-27`, `codex-rs/core/src/context/environment_context.rs:422-437`, `codex-rs/core/src/context/environment_context.rs:520-580`, `codex-rs/core/src/session/turn.rs:150-166`, `codex-rs/core/src/session/turn.rs:221-245`, `codex-rs/core/src/session/mod.rs:2658-2670`, `codex-rs/protocol/src/protocol.rs:481-535`, `codex-rs/protocol/src/protocol.rs:2983-3026`, `codex-rs/core/src/state/additional_context.rs:15-36`, `codex-rs/context-fragments/src/fragment.rs:37-88`, `codex-rs/context-fragments/src/additional_context.rs:21-92`, `codex-rs/core/src/hook_runtime.rs:595-615`, `codex-rs/core/src/prompt_debug.rs:74-101` |
| 8 | Token budgeting and context limits | complete | `cards/08-token-budgeting-and-context-limits.md` | Claude: `src/utils/context.ts:8-25`, `src/utils/context.ts:31-98`, `src/utils/context.ts:118-144`, `src/utils/tokens.ts:39-136`, `src/utils/tokens.ts:201-260`, `src/services/compact/autoCompact.ts:28-49`, `src/services/compact/autoCompact.ts:62-145`, `src/query.ts:592-647`, `src/services/api/errors.ts:62-77`, `src/services/api/claude.ts:470-501`, `src/services/api/claude.ts:1571-1575`, `src/services/api/claude.ts:2279-2292`, `src/utils/tokenBudget.ts:1-28`, `src/utils/tokenBudget.ts:66-73`, `src/bootstrap/state.ts:724-743`, `src/query/tokenBudget.ts:13-93`, `src/query.ts:1308-1332`, `src/screens/REPL.tsx:2893-2896`, `src/screens/REPL.tsx:2953-2968`, `src/components/Spinner.tsx:261-272`, `src/utils/attachments.ts:3828-3844`; Codex: `codex-rs/protocol/src/openai_models.rs:342-398`, `codex-rs/protocol/src/openai_models.rs:428-443`, `codex-rs/core/src/session/turn_context.rs:203-210`, `codex-rs/codex-api/src/sse/responses.rs:110-134`, `codex-rs/core/src/client.rs:1823-1840`, `codex-rs/core/src/session/turn.rs:2162-2177`, `codex-rs/core/src/context_manager/history.rs:130-150`, `codex-rs/core/src/context_manager/history.rs:249-314`, `codex-rs/core/src/session/mod.rs:1155-1194`, `codex-rs/core/src/session/turn.rs:230-322`, `codex-rs/core/src/session/turn.rs:736-815`, `codex-rs/core/src/session/turn.rs:1096-1108`, `codex-rs/protocol/src/protocol.rs:1672-1720`, `codex-rs/protocol/src/protocol.rs:1994-2072`, `codex-rs/protocol/src/protocol.rs:2146-2188`, `codex-rs/core/src/session/mod.rs:3218-3348`, `codex-rs/core/src/context/token_budget_context.rs:4-81`, `codex-rs/core/src/session/token_budget.rs:6-43`, `codex-rs/core/src/session/mod.rs:3033-3045`, `codex-rs/tui/src/chatwidget.rs:1109-1140`, `codex-rs/tui/src/bottom_pane/footer.rs:1008-1020` |
| 9 | Compaction: manual, auto, reactive, post-compact rehydration | complete | `cards/09-compaction-manual-auto-reactive-post-compact-rehydration.md` | Claude: `src/commands/compact/compact.ts:55-124`, `src/commands/compact/compact.ts:143-220`, `src/services/compact/autoCompact.ts:160-351`, `src/services/compact/compact.ts:299-367`, `src/services/compact/compact.ts:387-460`, `src/services/compact/compact.ts:587-624`, `src/services/compact/postCompactCleanup.ts:31-77`, `src/query.ts:453-535`, `src/query.ts:1080-1160`, `src/utils/messages.ts:4530-4555`, `src/utils/messages.ts:4608-4656`, `src/utils/sessionStorage.ts:1025-1048`, `src/utils/sessionStorage.ts:1839-1932`, `src/QueryEngine.ts:916-940`; Codex: `codex-rs/core/src/tasks/compact.rs:24-64`, `codex-rs/core/src/session/turn.rs:240-334`, `codex-rs/core/src/session/turn.rs:798-960`, `codex-rs/core/src/compact.rs:73-190`, `codex-rs/core/src/compact.rs:197-330`, `codex-rs/core/src/compact.rs:453-605`, `codex-rs/core/src/compact_remote.rs:49-89`, `codex-rs/core/src/compact_remote.rs:245-352`, `codex-rs/core/src/compact_remote_v2.rs:56-99`, `codex-rs/core/src/compact_remote_v2.rs:285-318`, `codex-rs/protocol/src/protocol.rs:621-624`, `codex-rs/protocol/src/protocol.rs:2942-2961`, `codex-rs/core/src/session/mod.rs:2756-2784`, `codex-rs/core/src/session/mod.rs:3129-3164`, `codex-rs/core/src/session/rollout_reconstruction.rs:96-142`, `codex-rs/core/src/session/rollout_reconstruction.rs:261-330` |
| 10 | Tool registry and model-visible schema projection | complete | `cards/10-tool-registry-and-model-visible-schema-projection.md` | Claude: `src/Tool.ts:158-179`, `src/Tool.ts:362-405`, `src/Tool.ts:430-472`, `src/Tool.ts:518-524`, `src/Tool.ts:697-792`, `src/tools.ts:185-250`, `src/tools.ts:253-327`, `src/tools.ts:329-388`, `src/query.ts:659-708`, `src/utils/api.ts:119-260`, `src/utils/toolSearch.ts:155-198`, `src/utils/toolSearch.ts:323-456`, `src/services/api/claude.ts:1118-1288`, `src/services/api/claude.ts:1458-1470`; Codex: `codex-rs/tools/src/tool_definition.rs:4-25`, `codex-rs/tools/src/responses_api.rs:25-136`, `codex-rs/tools/src/tool_spec.rs:13-89`, `codex-rs/tools/src/tool_executor.rs:13-68`, `codex-rs/tools/src/tool_search.rs:21-65`, `codex-rs/core/src/mcp_tool_exposure.rs:14-54`, `codex-rs/core/src/tools/registry.rs:43-144`, `codex-rs/core/src/tools/registry.rs:237-340`, `codex-rs/core/src/tools/router.rs:35-78`, `codex-rs/core/src/tools/spec_plan.rs:155-257`, `codex-rs/core/src/tools/spec_plan.rs:259-293`, `codex-rs/core/src/tools/spec_plan.rs:432-537`, `codex-rs/core/src/tools/spec_plan.rs:563-893`, `codex-rs/core/src/tools/spec_plan.rs:940-1007`, `codex-rs/core/src/tools/handlers/tool_search_spec.rs:7-61`, `codex-rs/core/src/session/turn.rs:1016-1034`, `codex-rs/core/src/session/turn.rs:1258-1278`, `codex-rs/core/src/client.rs:780-830` |
| 11 | Tool dispatch lifecycle and scheduling/concurrency | complete | `cards/11-tool-dispatch-lifecycle-and-scheduling-concurrency.md` | Claude: `src/query.ts:826-862`, `src/query.ts:1366-1408`, `src/services/tools/toolOrchestration.ts:19-187`, `src/services/tools/StreamingToolExecutor.ts:19-151`, `src/services/tools/StreamingToolExecutor.ts:210-240`, `src/services/tools/StreamingToolExecutor.ts:265-490`, `src/services/tools/toolExecution.ts:337-490`, `src/services/tools/toolExecution.ts:599-733`, `src/services/tools/toolExecution.ts:800-862`, `src/services/tools/toolExecution.ts:916-1222`, `src/services/tools/toolExecution.ts:1397-1473`, `src/Tool.ts:379-416`, `src/Tool.ts:483-503`; Codex: `codex-rs/core/src/session/turn.rs:1016-1027`, `codex-rs/core/src/session/turn.rs:1056-1070`, `codex-rs/core/src/session/turn.rs:1822-1847`, `codex-rs/core/src/session/turn.rs:1896-2043`, `codex-rs/core/src/session/turn.rs:2218-2234`, `codex-rs/core/src/session/turn.rs:2305-2311`, `codex-rs/core/src/stream_events_utils.rs:303-314`, `codex-rs/core/src/stream_events_utils.rs:405-507`, `codex-rs/core/src/tools/router.rs:100-239`, `codex-rs/core/src/tools/parallel.rs:30-234`, `codex-rs/core/src/tools/registry.rs:380-708`, `codex-rs/core/src/tools/lifecycle.rs:12-85`, `codex-rs/core/src/tools/tool_dispatch_trace.rs:24-115`, `codex-rs/core/src/tools/registry_tests.rs:373-452`, `codex-rs/core/src/tools/parallel.rs:404-544`, `codex-rs/core/src/tools/tool_dispatch_trace_tests.rs:61-123` |
| 12 | Tool result projection: raw output, model output, display output, persisted output | complete | `cards/12-tool-result-projection-raw-output-model-output-display-output-persisted-output.md` | Claude: `src/Tool.ts:321-335`, `src/Tool.ts:557-599`, `src/services/tools/toolExecution.ts:1271-1473`, `src/services/tools/toolExecution.ts:1540-1737`, `src/utils/toolResultStorage.ts:137-241`, `src/utils/toolResultStorage.ts:272-333`, `src/utils/messages.ts:460-523`, `src/utils/messages.ts:2522-2647`, `src/utils/messages.ts:5118-5458`, `src/utils/sessionStorage.ts:1025-1040`, `src/components/messages/UserToolResultMessage/UserToolSuccessMessage.tsx:52-102`; Codex: `codex-rs/tools/src/tool_output.rs:15-53`, `codex-rs/core/src/tools/context.rs:65-140`, `codex-rs/core/src/tools/context.rs:307-463`, `codex-rs/core/src/tools/registry.rs:159-205`, `codex-rs/core/src/tools/registry.rs:543-708`, `codex-rs/protocol/src/models.rs:804-840`, `codex-rs/protocol/src/models.rs:1420-1468`, `codex-rs/protocol/src/models.rs:1703-1950`, `codex-rs/core/src/session/turn.rs:1822-1847`, `codex-rs/core/src/session/mod.rs:2658-2670`, `codex-rs/core/src/context_manager/history.rs:347-439`, `codex-rs/core/src/tools/tool_dispatch_trace.rs:24-115`, `codex-rs/protocol/src/protocol.rs:1293-1328`, `codex-rs/protocol/src/protocol.rs:2313-2361`, `codex-rs/protocol/src/protocol.rs:3227-3293`, `codex-rs/core/src/tools/context_tests.rs:48-388`, `codex-rs/core/src/tools/registry_tests.rs:321-371`, `codex-rs/core/src/tools/tool_dispatch_trace_tests.rs:103-143` |
| 13 | File read/search tools | complete | `cards/13-file-read-search-tools.md` | Claude: `src/tools.ts:193-204`, `src/tools/FileReadTool/prompt.ts:27-48`, `src/tools/FileReadTool/FileReadTool.ts:227-417`, `src/tools/FileReadTool/FileReadTool.ts:652-718`, `src/tools/FileReadTool/FileReadTool.ts:1019-1070`, `src/utils/readFileInRange.ts:73-194`, `src/tools/GrepTool/GrepTool.ts:33-198`, `src/tools/GrepTool/GrepTool.ts:201-309`, `src/tools/GrepTool/GrepTool.ts:310-575`, `src/tools/GlobTool/GlobTool.ts:26-198`, `src/utils/glob.ts:66-130`, `src/utils/permissions/filesystem.ts:837-851`, `src/utils/permissions/filesystem.ts:1030-1095`; Codex: `codex-rs/core/src/tools/spec_plan.rs:155-237`, `codex-rs/core/src/tools/spec_plan.rs:563-693`, `codex-rs/core/src/tools/handlers/view_image_spec.rs:15-49`, `codex-rs/core/src/tools/handlers/view_image.rs:70-269`, `codex-rs/app-server-protocol/src/protocol/v2/fs.rs:7-110`, `codex-rs/app-server/src/request_processors/fs_processor.rs:64-140`, `codex-rs/file-system/src/lib.rs:187-220`, `codex-rs/app-server-protocol/src/protocol/common.rs:1153-1177`, `codex-rs/app-server-protocol/src/protocol/common.rs:1491-1568`, `codex-rs/app-server/src/request_processors/search.rs:39-133`, `codex-rs/file-search/src/lib.rs:90-307`, `codex-rs/app-server/tests/suite/fuzzy_file_search.rs:54-160` |
| 14 | File edit/write/patch/diff tools | complete | `cards/14-file-edit-write-patch-diff-tools.md` | Claude: `src/tools.ts:193-206`, `src/tools/FileEditTool/types.ts:5-79`, `src/tools/FileEditTool/prompt.ts:20-27`, `src/tools/FileEditTool/FileEditTool.ts:137-361`, `src/tools/FileEditTool/FileEditTool.ts:425-594`, `src/tools/FileWriteTool/prompt.ts:11-17`, `src/tools/FileWriteTool/FileWriteTool.ts:56-221`, `src/tools/FileWriteTool/FileWriteTool.ts:223-433`, `src/tools/NotebookEditTool/NotebookEditTool.ts:30-85`, `src/tools/NotebookEditTool/NotebookEditTool.ts:176-237`, `src/tools/NotebookEditTool/NotebookEditTool.ts:319-488`, `src/utils/permissions/filesystem.ts:1205-1411`, `src/utils/fileHistory.ts:63-193`; Codex: `codex-rs/core/src/tools/spec_plan.rs:689-693`, `codex-rs/core/src/tools/handlers/apply_patch_spec.rs:5-27`, `codex-rs/core/src/tools/handlers/apply_patch.lark:1-19`, `codex-rs/apply-patch/src/parser.rs:1-25`, `codex-rs/core/src/tools/handlers/apply_patch.rs:57-255`, `codex-rs/core/src/tools/handlers/apply_patch.rs:351-522`, `codex-rs/core/src/tools/runtimes/apply_patch.rs:46-267`, `codex-rs/apply-patch/src/lib.rs:92-246`, `codex-rs/apply-patch/src/lib.rs:276-550`, `codex-rs/apply-patch/src/lib.rs:619-872`, `codex-rs/core/src/tools/events.rs:122-164`, `codex-rs/core/src/tools/events.rs:352-430`, `codex-rs/core/src/tools/events.rs:566-618`, `codex-rs/core/src/turn_diff_tracker.rs:48-183`, `codex-rs/protocol/src/protocol.rs:3368-3420`, `codex-rs/protocol/src/protocol.rs:3842-3853` |
| 15 | Shell/exec/background process/interrupt tools | complete | `cards/15-shell-exec-background-process-interrupt-tools.md` | Claude: `src/tools/BashTool/BashTool.tsx:227-294`, `src/tools/BashTool/BashTool.tsx:624-820`, `src/tools/BashTool/BashTool.tsx:826-1142`, `src/utils/Shell.ts:181-345`, `src/utils/Shell.ts:385-423`, `src/utils/ShellCommand.ts:13-47`, `src/utils/ShellCommand.ts:135-193`, `src/utils/ShellCommand.ts:291-366`, `src/tasks/LocalShellTask/LocalShellTask.tsx:39-171`, `src/tasks/LocalShellTask/LocalShellTask.tsx:180-252`, `src/tasks/LocalShellTask/LocalShellTask.tsx:259-510`, `src/tasks/LocalShellTask/killShellTasks.ts:16-75`, `src/utils/task/TaskOutput.ts:21-164`, `src/tools/BashTool/UI.tsx:31-153`, `src/tools/BashTool/BashToolResultMessage.tsx:66-190`, `src/tools/PowerShellTool/PowerShellTool.tsx:232-995`; Codex: `codex-rs/core/src/tools/spec_plan.rs:585-623`, `codex-rs/core/src/tools/handlers/shell_spec.rs:21-151`, `codex-rs/core/src/tools/handlers/shell_spec.rs:154-220`, `codex-rs/core/src/tools/handlers/shell_spec.rs:261-292`, `codex-rs/core/src/tools/handlers/unified_exec.rs:27-95`, `codex-rs/core/src/tools/handlers/unified_exec/exec_command.rs:122-327`, `codex-rs/core/src/tools/handlers/unified_exec/write_stdin.rs:69-125`, `codex-rs/core/src/tools/handlers/shell.rs:60-238`, `codex-rs/core/src/tools/runtimes/shell.rs:54-326`, `codex-rs/core/src/unified_exec/mod.rs:1-179`, `codex-rs/core/src/unified_exec/process_manager.rs:346-614`, `codex-rs/core/src/unified_exec/process_manager.rs:616-888`, `codex-rs/core/src/unified_exec/process_manager.rs:1241-1357`, `codex-rs/core/src/unified_exec/process.rs:67-245`, `codex-rs/core/src/unified_exec/async_watcher.rs:37-157`, `codex-rs/core/src/tools/context.rs:307-435`, `codex-rs/protocol/src/protocol.rs:3206-3330` |
| 16 | Web/network tools | complete | `cards/16-web-network-tools.md` | Claude: `src/tools/WebFetchTool/WebFetchTool.ts:24-45`, `src/tools/WebFetchTool/WebFetchTool.ts:104-306`, `src/tools/WebFetchTool/utils.ts:130-530`, `src/tools/WebFetchTool/prompt.ts:1-45`, `src/tools/WebFetchTool/UI.tsx:9-71`, `src/tools/WebSearchTool/WebSearchTool.ts:25-37`, `src/tools/WebSearchTool/WebSearchTool.ts:76-84`, `src/tools/WebSearchTool/WebSearchTool.ts:254-434`, `src/tools/WebSearchTool/UI.tsx:8-100`, `src/services/api/claude.ts:2948-3011`; Codex: `codex-rs/core/src/tools/spec_plan.rs:259-285`, `codex-rs/core/src/tools/hosted_spec.rs:20-43`, `codex-rs/ext/web-search/src/extension.rs:26-126`, `codex-rs/ext/web-search/src/tool.rs:36-123`, `codex-rs/ext/web-search/src/history.rs:14-65`, `codex-rs/ext/web-search/src/output.rs:7-33`, `codex-rs/codex-api/src/search.rs:7-151`, `codex-rs/codex-api/src/endpoint/search.rs:16-38`, `codex-rs/protocol/src/models.rs:1058-1080`, `codex-rs/tui/src/history_cell/search.rs:5-150` |
| 17 | Task/todo/plan tools and task state | complete | `cards/17-task-todo-plan-tools-and-task-state.md` | Claude: `src/tools/TodoWriteTool/TodoWriteTool.ts:13-115`, `src/utils/tasks.ts:61-224`, `src/tools/TaskCreateTool/TaskCreateTool.ts:18-125`, `src/tools/TaskUpdateTool/TaskUpdateTool.ts:33-398`, `src/hooks/useTasksV2.ts:1-205`, `src/tools/EnterPlanModeTool/EnterPlanModeTool.ts:21-126`, `src/tools/ExitPlanModeTool/ExitPlanModeV2Tool.ts:220-493`, `src/utils/plans.ts:79-144`; Codex: `codex-rs/core/src/tools/handlers/plan_spec.rs:7-58`, `codex-rs/core/src/tools/handlers/plan.rs:18-105`, `codex-rs/protocol/src/plan_tool.rs:6-29`, `codex-rs/app-server/src/bespoke_event_handling.rs:1260-1318`, `codex-rs/tui/src/history_cell/plans.rs:47-240`, `codex-rs/core/src/session/turn.rs:1294-1430`, `codex-rs/ext/goal/src/spec.rs:1-94`, `codex-rs/ext/goal/src/tool.rs:134-285`, `codex-rs/state/src/model/thread_goal.rs:14-75` |
| 18 | Subagent/fork/delegation model | complete | `cards/18-subagent-fork-delegation-model.md` | Claude: `src/tools/AgentTool/AgentTool.tsx:82-154`, `src/tools/AgentTool/AgentTool.tsx:244-333`, `src/tools/AgentTool/AgentTool.tsx:483-640`, `src/tools/AgentTool/AgentTool.tsx:688-763`, `src/tools/AgentTool/forkSubagent.ts:18-198`, `src/tools/AgentTool/runAgent.ts:248-860`, `src/utils/forkedAgent.ts:46-120`, `src/utils/forkedAgent.ts:306-689`, `src/tools/shared/spawnMultiAgent.ts:305-535`, `src/tools/shared/spawnMultiAgent.ts:760-1031`; Codex: `codex-rs/core/src/tools/spec_plan.rs:311-323`, `codex-rs/core/src/tools/spec_plan.rs:718-792`, `codex-rs/core/src/tools/handlers/multi_agents_spec.rs:48-328`, `codex-rs/core/src/tools/handlers/multi_agents_spec.rs:555-745`, `codex-rs/core/src/tools/handlers/multi_agents_v2/spawn.rs:39-263`, `codex-rs/core/src/agent/control.rs:57-103`, `codex-rs/core/src/agent/control.rs:124-521`, `codex-rs/core/src/agent/control/spawn.rs:194-521`, `codex-rs/protocol/src/protocol.rs:677-784`, `codex-rs/protocol/src/protocol.rs:1641-1661`, `codex-rs/core/src/session/mod.rs:1699-1807`, `codex-rs/core/src/session/mod.rs:2673-2691` |
| 19 | Worktree/isolation/sandbox behavior | complete | `cards/19-worktree-isolation-sandbox-behavior.md` | Claude: `src/utils/worktree.ts:48-87`, `src/utils/worktree.ts:702-778`, `src/setup.ts:191-285`, `src/tools/EnterWorktreeTool/EnterWorktreeTool.ts:23-127`, `src/tools/ExitWorktreeTool/ExitWorktreeTool.ts:30-328`, `src/tools/AgentTool/AgentTool.tsx:90-101`, `src/tools/AgentTool/AgentTool.tsx:582-685`, `src/utils/sandbox/sandbox-adapter.ts:172-380`, `src/utils/sandbox/sandbox-adapter.ts:730-803`, `src/tools/BashTool/prompt.ts:172-272`; Codex: `codex-rs/protocol/src/protocol.rs:927-979`, `codex-rs/protocol/src/permissions.rs:568-660`, `codex-rs/protocol/src/permissions.rs:989-1103`, `codex-rs/core/src/config/mod.rs:2822-2865`, `codex-rs/core/src/config/mod.rs:2910-3078`, `codex-rs/core/src/exec.rs:331-438`, `codex-rs/core/src/landlock.rs:14-70`, `codex-rs/core/src/context/environment_context.rs:97-158`, `codex-rs/core/src/config/config_loader_tests.rs:2261-2392` |
| 20 | Agent messaging/inbox/notifications | complete | `cards/20-agent-messaging-inbox-notifications.md` | Claude: `src/utils/teammateMailbox.ts:43-192`, `src/hooks/useInboxPoller.ts:74-969`, `src/tools/SendMessageTool/SendMessageTool.ts:149-917`, `src/tasks/LocalAgentTask/LocalAgentTask.tsx:197-261`, `src/utils/attachments.ts:3532-3715`, `src/utils/messages.ts:3453-5511`; Codex: `codex-rs/protocol/src/protocol.rs:677-784`, `codex-rs/core/src/session/input_queue.rs:12-253`, `codex-rs/core/src/tools/handlers/multi_agents_v2/message_tool.rs:1-125`, `codex-rs/core/src/tools/handlers/multi_agents_v2/wait.rs:37-189`, `codex-rs/core/src/session/mod.rs:1699-1807`, `codex-rs/core/src/session/mod.rs:2673-2690` |
| 21 | Memory: session, project, long-term, extraction lifecycle | complete | `cards/21-memory-session-project-long-term-extraction-lifecycle.md` | Claude: `src/utils/claudemd.ts:1-26`, `src/memdir/paths.ts:21-90`, `src/memdir/memdir.ts:187-507`, `src/memdir/findRelevantMemories.ts:18-141`, `src/utils/attachments.ts:2200-2541`, `src/services/extractMemories/extractMemories.ts:1-603`, `src/services/SessionMemory/sessionMemory.ts:1-481`, `src/services/compact/sessionMemoryCompact.ts:400-575`, `src/tools/AgentTool/agentMemory.ts:12-177`; Codex: `codex-rs/features/src/lib.rs:126-130`, `codex-rs/config/src/types.rs:280-392`, `codex-rs/ext/memories/src/extension.rs:21-128`, `codex-rs/ext/memories/templates/memories/read_path.md:1-130`, `codex-rs/memories/write/src/start.rs:18-78`, `codex-rs/memories/write/src/phase1.rs:70-480`, `codex-rs/memories/write/src/phase2.rs:44-520`, `codex-rs/state/src/runtime/memories.rs:26-1045`, `codex-rs/app-server-protocol/src/protocol/common.rs:560-570` |
| 22 | Skills/plugins/custom instructions | complete | `cards/22-skills-plugins-custom-instructions.md` | Claude: `src/skills/loadSkillsDir.ts:67-105`, `src/skills/loadSkillsDir.ts:155-264`, `src/tools/SkillTool/prompt.ts:20-196`, `src/tools/SkillTool/SkillTool.ts:118-289`, `src/tools/SkillTool/SkillTool.ts:291-780`, `src/utils/plugins/pluginLoader.ts:1320-1610`, `src/utils/plugins/pluginLoader.ts:1880-2025`, `src/utils/plugins/pluginLoader.ts:3137-3225`, `src/utils/plugins/loadPluginCommands.ts:218-402`, `src/utils/plugins/loadPluginAgents.ts:65-229`, `src/utils/plugins/loadPluginHooks.ts:25-157`, `src/utils/claudemd.ts:1-26`, `src/utils/claudemd.ts:615-685`, `src/utils/claudemd.ts:790-940`, `src/utils/claudemd.ts:1399-1430`; Codex: `codex-rs/core-skills/src/model.rs:14-58`, `codex-rs/core-skills/src/loader.rs:155-233`, `codex-rs/core-skills/src/loader.rs:283-397`, `codex-rs/core-skills/src/loader.rs:639-920`, `codex-rs/core-skills/src/manager.rs:60-194`, `codex-rs/core-skills/src/render.rs:17-200`, `codex-rs/core-skills/src/injection.rs:19-200`, `codex-rs/ext/skills/src/extension.rs:53-379`, `codex-rs/ext/skills/src/tools/mod.rs:33-160`, `codex-rs/plugin/src/manifest.rs:3-57`, `codex-rs/plugin/src/load_outcome.rs:15-188`, `codex-rs/core/src/plugins/injection.rs:14-59`, `codex-rs/core/src/context/available_plugins_instructions.rs:1-58`, `codex-rs/core/src/agents_md.rs:1-17`, `codex-rs/core/src/agents_md.rs:168-450` |
| 23 | Session persistence, transcript storage, resume, repair | complete | `cards/23-session-persistence-transcript-storage-resume-repair.md` | Claude: `src/types/logs.ts:8-53`, `src/types/logs.ts:143-231`, `src/utils/sessionStorage.ts:128-303`, `src/utils/sessionStorage.ts:953-1668`, `src/utils/sessionStorage.ts:1770-2484`, `src/utils/sessionStorage.ts:3150-4461`, `src/commands/resume/resume.tsx:80-260`, `src/utils/sessionRestore.ts:64-520`, `src/utils/conversationRecovery.ts:148-520`, `src/tools/AgentTool/resumeAgent.ts:58-180`; Codex: `codex-rs/protocol/src/protocol.rs:2863-3045`, `codex-rs/protocol/src/protocol.rs:3591-3725`, `codex-rs/rollout/src/recorder.rs:66-980`, `codex-rs/rollout/src/metadata.rs:25-220`, `codex-rs/rollout/src/session_index.rs:17-260`, `codex-rs/core/src/session/rollout_reconstruction.rs:4-320`, `codex-rs/core/src/session/mod.rs:1224-3215`, `codex-rs/app-server/src/request_processors/thread_processor.rs:2500-3235`, `codex-rs/tui/src/session_resume.rs:1-180`, `codex-rs/exec/tests/suite/resume.rs:250-565` |
| 24 | Permissions, approvals, policy enforcement | complete | `cards/24-permissions-approvals-policy-enforcement.md` | Claude: `src/types/permissions.ts:16-80`, `src/types/permissions.ts:241-324`, `src/utils/permissions/permissions.ts:473-535`, `src/utils/permissions/permissions.ts:929-1320`, `src/utils/permissions/filesystem.ts:1028-1465`, `src/utils/permissions/PermissionUpdate.ts:55-389`, `src/hooks/useCanUseTool.tsx:27-182`, `src/hooks/toolPermission/handlers/interactiveHandler.ts:43-430`; Codex: `codex-rs/protocol/src/approvals.rs:17-390`, `codex-rs/protocol/src/request_permissions.rs:10-99`, `codex-rs/protocol/src/models.rs:264-485`, `codex-rs/protocol/src/protocol.rs:837-905`, `codex-rs/core/src/tools/sandboxing.rs:39-363`, `codex-rs/core/src/tools/orchestrator.rs:132-579`, `codex-rs/core/src/tools/runtimes/shell.rs:55-206`, `codex-rs/core/src/tools/runtimes/apply_patch.rs:40-213`, `codex-rs/app-server/src/bespoke_event_handling.rs:511-800`, `codex-rs/core/tests/suite/request_permissions_tool.rs:206-430` |
| 25 | Error taxonomy and recovery/retry behavior | complete | `cards/25-error-taxonomy-and-recovery-retry-behavior.md` | Claude: `src/services/api/withRetry.ts:57-516`, `src/services/api/errors.ts:425-1060`, `src/utils/errors.ts:1-238`, `src/services/tools/toolExecution.ts:599-1737`, `src/services/tools/StreamingToolExecutor.ts:210-490`, `src/query.ts:900-1305`, `src/QueryEngine.ts:1082-1160`; Codex: `codex-rs/protocol/src/error.rs:32-259`, `codex-rs/codex-api/src/api_bridge.rs:18-137`, `codex-rs/codex-api/src/sse/responses.rs:347-570`, `codex-rs/core/src/responses_retry.rs:1-105`, `codex-rs/core/src/session/turn.rs:140-409`, `codex-rs/core/src/session/turn.rs:1088-1140`, `codex-rs/core/src/client.rs:1263-1465`, `codex-rs/core/src/tools/parallel.rs:62-220`, `codex-rs/core/src/session/mod.rs:1645-1678` |
| 26 | Debug logs, trace bundles, model payload capture | complete | `cards/26-debug-logs-trace-bundles-model-payload-capture.md` | Claude: `src/utils/debug.ts:18-253`, `src/utils/diagLogs.ts:14-94`, `src/utils/log.ts:79-352`, `src/utils/errorLogSink.ts:24-234`, `src/services/api/logging.ts:171-788`, `src/services/api/dumpPrompts.ts:13-225`, `src/components/Feedback.tsx:194-225`; Codex: `codex-rs/rollout-trace/src/payload.rs:9-49`, `codex-rs/rollout-trace/src/writer.rs:31-265`, `codex-rs/rollout-trace/src/inference.rs:1-523`, `codex-rs/rollout-trace/src/protocol_event.rs:1-408`, `codex-rs/rollout-trace/src/thread.rs:39-524`, `codex-rs/core/src/client.rs:1744-1910`, `codex-rs/cli/src/main.rs:220-307`, `codex-rs/cli/src/main.rs:1886-1974` |
| 27 | TUI rendering and interaction model | complete | `cards/27-tui-rendering-and-interaction-model.md` | Claude: `src/screens/REPL.tsx:1458-1473`, `src/screens/REPL.tsx:2106-2163`, `src/screens/REPL.tsx:4380-4489`, `src/screens/REPL.tsx:4492-4907`, `src/components/Messages.tsx:341-820`, `src/components/PromptInput/PromptInput.tsx:984-1125`, `src/components/PromptInput/PromptInput.tsx:1637-1737`, `src/components/PromptInput/PromptInput.tsx:2172-2296`; Codex: `codex-rs/tui/src/tui.rs:510-1065`, `codex-rs/tui/src/app.rs:503-1350`, `codex-rs/tui/src/app/input.rs:1-254`, `codex-rs/tui/src/chatwidget.rs:1-760`, `codex-rs/tui/src/chatwidget/rendering.rs:5-149`, `codex-rs/tui/src/chatwidget/protocol.rs:4-335`, `codex-rs/tui/src/history_cell/mod.rs:180-280`, `codex-rs/tui/src/bottom_pane/mod.rs:1-241`, `codex-rs/tui/src/status_indicator_widget.rs:1-438` |
| 28 | Eval harness, smoke tests, golden payloads, trajectory traces | complete | `cards/28-eval-harness-smoke-tests-golden-payloads-trajectory-traces.md` | Claude: `src/services/vcr.ts:23-120`, `src/tools/testing/TestingPermissionTool.tsx:1-73`, `src/query/deps.ts:8-40`, `src/services/analytics/growthbook.ts:159-209`, `src/services/analytics/metadata.ts:469-496`; Codex: `codex-rs/core/tests/common/test_codex.rs:115-190`, `codex-rs/core/tests/common/responses.rs:38-304`, `codex-rs/core/tests/common/context_snapshot.rs:12-227`, `codex-rs/app-server/tests/common/test_app_server.rs:117-360`, `codex-rs/rollout-trace/src/writer.rs:157-265`, `sdk/typescript/tests/runStreamed.test.ts:14-200` |
| 29 | Runtime hygiene and product-name leakage prevention | complete | `cards/29-runtime-hygiene-and-product-name-leakage-prevention.md` | Claude: `src/services/analytics/metadata.ts:44-88`, `src/services/analytics/index.ts:11-58`, `src/services/mcp/auth.ts:96-125`, `src/services/mcp/client.ts:2593-2625`, `src/state/onChangeAppState.ts:55-82`; Codex: `codex-rs/login/src/auth/default_client.rs:57-90`, `codex-rs/otel/src/events/session_telemetry.rs:941-982`, `codex-rs/login/src/server.rs:626-706`, `codex-rs/mcp-server/src/message_processor.rs:216-250`, `codex-rs/rollout-trace/README.md:1-20` |
| 30 | Final skill packaging and usage workflow | complete | `cards/30-final-skill-packaging-and-usage-workflow.md`; `SKILL.md` | Claude: `src/skills/loadSkillsDir.ts:67-105`, `src/skills/loadSkillsDir.ts:180-265`, `src/skills/loadSkillsDir.ts:267-470`, `src/tools/SkillTool/prompt.ts:20-195`, `src/tools/SkillTool/SkillTool.ts:331-700`; Codex: `codex-rs/core-skills/src/model.rs:14-68`, `codex-rs/core-skills/src/loader.rs:155-233`, `codex-rs/core-skills/src/render.rs:17-84`, `codex-rs/core-skills/src/injection.rs:58-200`, `codex-rs/ext/skills/src/extension.rs:53-379` |
