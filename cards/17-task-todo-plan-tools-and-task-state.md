# Mechanism 17: Task/todo/plan tools and task state

Mechanism: Task/todo/plan tools and task state
Status: complete

Claude source:
- V1 todo tool: `src/tools/TodoWriteTool/TodoWriteTool.ts:13-115`; `src/utils/todo/types.ts:1-16`; `src/tools/TodoWriteTool/prompt.ts:1-170`.
- V2 task state and storage: `src/utils/tasks.ts:61-224`, `src/utils/tasks.ts:249-410`, `src/utils/tasks.ts:412-744`.
- V2 task tools: `src/tools/TaskCreateTool/TaskCreateTool.ts:18-125`; `src/tools/TaskListTool/TaskListTool.ts:13-112`; `src/tools/TaskUpdateTool/TaskUpdateTool.ts:33-398`; `src/tools/TaskGetTool/TaskGetTool.ts:1-112`.
- V2 task display/watch: `src/hooks/useTasksV2.ts:1-205`; `src/hooks/useTaskListWatcher.ts:1-220`.
- Plan mode tools and plan files: `src/tools/EnterPlanModeTool/EnterPlanModeTool.ts:21-126`; `src/tools/ExitPlanModeTool/ExitPlanModeV2Tool.ts:119-190`, `src/tools/ExitPlanModeTool/ExitPlanModeV2Tool.ts:220-345`, `src/tools/ExitPlanModeTool/ExitPlanModeV2Tool.ts:423-493`; `src/utils/plans.ts:60-144`.
- Registration: `src/tools.ts:193-250`.

Codex source:
- `update_plan` tool schema/handler/types: `codex-rs/core/src/tools/handlers/plan_spec.rs:7-58`; `codex-rs/core/src/tools/handlers/plan.rs:18-105`; `codex-rs/protocol/src/plan_tool.rs:6-29`.
- `update_plan` registration and prompt guidance: `codex-rs/core/src/tools/spec_plan.rs:644-665`; `codex-rs/protocol/src/prompts/base_instructions/default.md:54-70`, `codex-rs/protocol/src/prompts/base_instructions/default.md:267-275`.
- `update_plan` app-server/TUI projection: `codex-rs/app-server/src/bespoke_event_handling.rs:1260-1318`; `codex-rs/app-server-protocol/src/protocol/v2/turn.rs:405-450`; `codex-rs/tui/src/chatwidget/protocol.rs:79-106`; `codex-rs/tui/src/chatwidget/turn_runtime.rs:460-474`; `codex-rs/tui/src/history_cell/plans.rs:47-240`.
- Plan-mode proposed-plan stream: `codex-rs/core/src/session/turn.rs:1294-1430`, `codex-rs/core/src/session/turn.rs:1544-1608`; `codex-rs/utils/stream-parser/src/proposed_plan.rs:1-85`.
- Persisted thread goals: `codex-rs/ext/goal/src/spec.rs:1-94`; `codex-rs/ext/goal/src/tool.rs:134-285`, `codex-rs/ext/goal/src/tool.rs:300-430`; `codex-rs/ext/goal/src/extension.rs:415-477`; `codex-rs/state/src/model/thread_goal.rs:14-75`; `codex-rs/state/src/runtime/goals.rs:41-210`, `codex-rs/state/src/runtime/goals.rs:384-455`.

Runtime trace evidence:
- Claude V1 todo calls write `appState.todos[todoKey]`, where `todoKey` is `context.agentId ?? getSessionId()`, and clear persisted app-state todos when all submitted todos are completed (`TodoWriteTool.ts:65-94`).
- Claude V2 task calls write JSON task files under `getClaudeConfigHomeDir()/tasks/{taskListId}` after path sanitization and high-water mark locking (`tasks.ts:91-224`, `tasks.ts:249-310`).
- Claude plan entry mutates `toolPermissionContext` into `plan`; exit reads or writes the plan file, asks non-teammate permission, may send teammate leader-approval mailbox messages, then restores permission mode and returns the approved plan to the model (`EnterPlanModeTool.ts:77-124`; `ExitPlanModeV2Tool.ts:220-345`, `ExitPlanModeV2Tool.ts:423-493`).
- Codex `update_plan` parses function arguments into `UpdatePlanArgs`, rejects calls when `collaboration_mode.mode == ModeKind::Plan`, emits `EventMsg::PlanUpdate`, and returns model-visible function output `Plan updated` (`plan.rs:33-95`).
- Codex app-server maps `EventMsg::PlanUpdate` into `TurnPlanUpdatedNotification`; TUI records `saw_plan_update_this_turn`, computes `(completed,total)`, refreshes status surfaces, and appends a `PlanUpdateCell` (`bespoke_event_handling.rs:1260-1318`; `turn_runtime.rs:460-474`).
- Codex proposed plan streaming is separate: `<proposed_plan>` segments become `PlanDelta` and `TurnItem::Plan`, while normal text remains assistant message delta (`turn.rs:1294-1430`, `turn.rs:1544-1608`; `proposed_plan.rs:1-85`).
- Codex goal tools are extension tools that read/write `thread_goals` state DB rows and emit thread-goal updated events through the extension event sink (`spec.rs:1-94`; `tool.rs:134-285`; `extension.rs:415-477`; `state/src/runtime/goals.rs:41-210`).

What the model sees:
- Claude V1 sees `TodoWrite` with a strict input schema containing `todos`, each todo following the shared todo schema; its prompt defines status usage and the "exactly one in_progress" convention (`TodoWriteTool.ts:13-17`; `prompt.ts:1-170`).
- Claude V2 sees `TaskCreate`, `TaskGet`, `TaskUpdate`, and `TaskList` only when `isTodoV2Enabled()` is true; schemas expose task subject/description/status/owner/dependency fields and list output lines like `#id [status] subject (owner) [blocked by #...]` (`TaskCreateTool.ts:18-70`; `TaskUpdateTool.ts:33-112`; `TaskListTool.ts:91-112`).
- Claude plan mode exposes `EnterPlanMode` and `ExitPlanMode`; `EnterPlanMode` returns read-only planning instructions, while `ExitPlanMode` returns approved-plan content, saved path, and todo/team hints when applicable (`EnterPlanModeTool.ts:103-124`; `ExitPlanModeV2Tool.ts:423-493`).
- Codex sees `update_plan` with `explanation?: string` and `plan: [{step,status}]`, statuses `pending|in_progress|completed`, and a schema description limiting the plan to at most one `in_progress` step (`plan_spec.rs:7-58`).
- Codex base prompt tells the model to use `update_plan` for non-trivial, multi-step, ambiguous, or user-requested TODO work; it explicitly says not to repeat the full plan after the tool call because the harness displays it (`default.md:54-70`, `default.md:267-275`).
- Codex goal tools are model-visible only through the goal extension and include `get_goal`, `create_goal`, and `update_goal`; `create_goal` is restricted to explicit user/system/developer requests, and `update_goal` can only mark `complete` or `blocked` (`spec.rs:13-94`; `extension.rs:415-447`).

What the runtime does:
- Claude TodoWrite is app-state-only per session/agent and has no permission checks (`TodoWriteTool.ts:58-94`).
- Claude V2 tasks are file-backed, locked, and list-scoped; task list IDs come from env/team/session priority, each task JSON has id/subject/description/activeForm/owner/status/blocks/blockedBy/metadata, and update/delete/block operations mutate files and notify watchers (`tasks.ts:69-224`, `tasks.ts:249-410`).
- Claude task tools auto-expand the tasks UI on create/update, run task-created/task-completed hooks, delete rejected created tasks, auto-assign owners in swarm mode, and send mailbox assignment messages on owner changes (`TaskCreateTool.ts:80-119`; `TaskUpdateTool.ts:139-324`).
- Claude plan mode is a permission-mode state machine: entering plan mode applies a permission update to `plan`; exiting reads plan text from disk or permission-updated input, optionally writes the edited plan back to disk, asks non-teammates for approval, and restores execution mode (`EnterPlanModeTool.ts:77-94`; `ExitPlanModeV2Tool.ts:220-345`).
- Codex `update_plan` does not maintain a durable task object. It emits a turn event and returns a tool output; display state is accumulated by app-server/TUI from the event (`plan.rs:84-95`; `bespoke_event_handling.rs:1260-1318`; `turn_runtime.rs:460-474`).
- Codex proposed-plan stream state is ephemeral per response; the code comment says it is intentionally not persisted or stored in session/state, and the final plan text is extracted from the completed assistant message (`turn.rs:1294-1302`).
- Codex thread goals are durable state objects with status, token budget, tokens used, elapsed time, and timestamps. Goal lifecycle contributors count token/tool/turn progress and inject budget-limit steering when needed (`state/src/model/thread_goal.rs:14-75`; `tool.rs:236-285`, `tool.rs:300-430`; `extension.rs:471-476`).

What gets persisted:
- Claude V1 todo state persists in the session app state under `appState.todos`, unless all items are complete, in which case the stored list is cleared (`TodoWriteTool.ts:65-94`).
- Claude V2 task state persists as JSON files plus `.highwatermark` in the task directory; completed V2 lists can be reset after the display hide timer fires (`tasks.ts:91-188`; `useTasksV2.ts:123-170`).
- Claude plan mode persists plan Markdown under `plansDirectory` or config-home `plans`; main conversations use `{planSlug}.md`, subagents use `{planSlug}-agent-{agentId}.md`, and `getPlan()` reads that file (`plans.ts:79-144`).
- Codex `update_plan` is not a durable goal/task store in the cited core path; it is a turn event and TUI/history display cell (`plan.rs:90-95`; `turn_runtime.rs:460-474`; `history_cell/plans.rs:160-240`).
- Codex proposed-plan finalized history owns raw Markdown for terminal resize reflow, while the live stream cell is explicitly a transient display fragment (`history_cell/plans.rs:52-95`, `history_cell/plans.rs:100-158`).
- Codex thread goals persist in `thread_goals` SQLite-backed state rows, with insert/replace/update/delete/accounting methods in `state/src/runtime/goals.rs:41-210` and `state/src/runtime/goals.rs:384-455`.

What the user/TUI sees:
- Claude V1 TodoWrite has no tool-use message; V2 task tools return no direct tool-use message, but the task view is auto-expanded and `useTasksV2` watches, polls, hides, and resets the visible task list (`TodoWriteTool.ts:62-64`; `TaskCreateTool.ts:115-119`; `TaskUpdateTool.ts:139-143`; `useTasksV2.ts:1-205`).
- Claude plan entry/exit tools have explicit UI renderers and the model-visible exit result mirrors user approval, rejection, saved path, or teammate leader-approval state (`EnterPlanModeTool.ts:74-76`; `ExitPlanModeV2Tool.ts:240-242`, `ExitPlanModeV2Tool.ts:423-493`).
- Codex TUI renders `update_plan` as an "Updated Plan" history cell with optional italic note and checkbox-like statuses: completed crossed/dim, in-progress cyan/bold, pending dim (`history_cell/plans.rs:160-240`).
- Codex TUI records latest plan progress for status/title surfaces (`turn_runtime.rs:460-474`).
- Codex proposed plan renders as "Proposed Plan" history cells, distinct from `update_plan` checklist cells (`history_cell/plans.rs:52-158`).
- Codex goal status display formats `/goal` usage, status labels, elapsed time, and token budget summaries (`tui/src/goal_display.rs:1-65`).

What debug/eval surfaces exist:
- Claude V2 task mutations call `notifyTasksUpdated()` and tolerate listener failures, giving a debuggable update signal without failing the task mutation (`tasks.ts:61-67`).
- Claude task create/update hook failures are surfaced as blocking tool errors or non-success task update data (`TaskCreateTool.ts:92-113`; `TaskUpdateTool.ts:231-264`).
- Claude TodoWrite and TaskUpdate include source-specific verification nudges when feature flags are enabled and a 3+ item list is closed without a verification-named task (`TodoWriteTool.ts:72-88`, `TodoWriteTool.ts:104-113`; `TaskUpdateTool.ts:326-398`).
- Codex `PlanToolOutput` reports `Plan updated` as both logging preview and response item, with `success=true` (`plan.rs:22-45`).
- Codex TUI has snapshot tests for plan-update rendering and wrapping, and a plan-mode test proving `update_plan` does not create the plan implementation popup without a proposed plan (`tui/src/history_cell/tests.rs:2118-2190`; `tui/src/chatwidget/tests/plan_mode.rs:930-955`).
- Codex goal tools emit analytics/metrics events on creation, status changes, usage accounting, and terminal status changes (`tool.rs:207-218`, `tool.rs:273-285`, `tool.rs:300-360`).

Shared invariant:
- A harness should separate lightweight model planning/checklist state from permission-gated plan-mode state and from durable, resumable goal/task state. Claude and Codex both expose model-visible progress tools, but they do not give those tools the same runtime semantics.

Claude-specific design:
- Claude has two todo/task generations: V1 `TodoWrite` writes per-session/per-agent app-state todos and clears them when complete; V2 `Task*` tools write lock-protected, file-backed JSON task lists with ownership, dependency, hooks, mailbox assignment, task watchers, and UI auto-expansion.
- Claude plan mode is an explicit tool-driven permission-mode transition, not just a checklist. `EnterPlanMode` changes `toolPermissionContext.mode` to `plan`; `ExitPlanMode` requires approval, reads/writes a plan file, and returns approved plan content to the model.

Codex-specific design:
- Codex `update_plan` is a turn-local TODO/checklist display tool. The source explicitly says its arguments are for the `update_plan` todo/checklist tool, "not plan mode" (`plan_tool.rs:22-28`), and the handler rejects it inside collaboration Plan mode (`plan.rs:84-88`).
- Codex plan-mode proposed plans are parsed from `<proposed_plan>` tags and streamed as `PlanDelta`/`TurnItem::Plan`; they are distinct from `update_plan`.
- Codex persisted task-like objective state is modeled as thread goals, exposed by extension tools and backed by the `thread_goals` state table with token/time usage accounting and budget/terminal statuses.

Gaps or unknowns:
- Claude source shows V2 task files and plan files, but this card does not assert how every task/planning field is serialized into long-term transcript history; transcript persistence is mechanism 23.
- Codex `update_plan` persistence beyond TUI/history cells is not asserted here; session transcript storage and rollout replay are mechanism 23 and eval traces are mechanism 28.
- Subagent/fork/delegation effects on task ownership, plan approval, and leader mailbox are intentionally deferred to mechanisms 18 and 20.
- Permission-policy taxonomy for plan mode and tool approvals is deferred to mechanism 24.

Distilled harness rule:
- Implement progress planning as three explicit layers, only if the source supports each layer: model-visible checklist updates, runtime task/plan state, and durable objective/goal state. Name each layer after its actual source mechanism and keep its lifecycle, persistence, and display semantics separate.

Anti-invention rule:
- Do not infer that a checklist tool is a plan-mode system, or that a plan-mode proposal is a durable task store. In source-derived harness guidance, call Codex `update_plan` a TODO/checklist event tool, Claude `EnterPlanMode`/`ExitPlanMode` a permission-mode plan workflow, Claude `Task*` tools file-backed V2 tasks, and Codex goals persisted thread goals.

Verification pattern:
- For a todo/checklist mechanism, verify the tool schema, handler output, event/display projection, and whether a durable store exists.
- For a task-state mechanism, verify the data type/schema, storage path/table, create/update/delete operations, status transitions, ownership/dependency behavior, watcher/display behavior, and hook/error behavior.
- For plan mode, verify entry state mutation, allowed writes, plan file path/read/write behavior, approval path, model-visible exit result, and whether proposed-plan display is separate from checklist display.
- For goal state, verify model tool exposure, state DB object fields, create/update restrictions, accounting lifecycle hooks, budget/terminal behavior, and user/TUI status surfaces.

Skill text candidate:
- When studying task/todo/plan harness behavior, first classify the mechanism: checklist progress, explicit runtime task list, permission-gated plan mode, proposed-plan display stream, or durable thread goal. Then cite the exact source path for schema, runtime mutation, persistence, user display, and debug/eval surface. Never merge two categories because their names sound similar.
