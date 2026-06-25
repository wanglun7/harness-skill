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

## Mechanism Status

| # | Mechanism | Status | Card | Key citations |
|---:|---|---|---|---|
| 1 | Repository/source map and harness boundary definition | complete | `cards/01-repository-source-map-and-harness-boundary.md` | Claude: `src/entrypoints/cli.tsx:28-33`, `src/QueryEngine.ts:175-184`, `src/QueryEngine.ts:1179-1186`, `src/query.ts:181-239`, `src/Tool.ts:697-701`; Codex: `codex-rs/cli/src/main.rs:90-120`, `codex-rs/core/src/lib.rs:1-18`, `codex-rs/core/src/session/session.rs:23-47`, `codex-rs/core/src/session/turn.rs:140-245` |
| 2 | Model/provider abstraction | in_progress | | |
| 3 | Main turn loop and stop conditions | not_started | | |
| 4 | Streaming protocol and assistant text/tool interleaving | not_started | | |
| 5 | Message normalization and provider payload construction | not_started | | |
| 6 | System prompt and instruction layering | not_started | | |
| 7 | Dynamic context assembly | not_started | | |
| 8 | Token budgeting and context limits | not_started | | |
| 9 | Compaction: manual, auto, reactive, post-compact rehydration | not_started | | |
| 10 | Tool registry and model-visible schema projection | not_started | | |
| 11 | Tool dispatch lifecycle and scheduling/concurrency | not_started | | |
| 12 | Tool result projection: raw output, model output, display output, persisted output | not_started | | |
| 13 | File read/search tools | not_started | | |
| 14 | File edit/write/patch/diff tools | not_started | | |
| 15 | Shell/exec/background process/interrupt tools | not_started | | |
| 16 | Web/network tools | not_started | | |
| 17 | Task/todo/plan tools and task state | not_started | | |
| 18 | Subagent/fork/delegation model | not_started | | |
| 19 | Worktree/isolation/sandbox behavior | not_started | | |
| 20 | Agent messaging/inbox/notifications | not_started | | |
| 21 | Memory: session, project, long-term, extraction lifecycle | not_started | | |
| 22 | Skills/plugins/custom instructions | not_started | | |
| 23 | Session persistence, transcript storage, resume, repair | not_started | | |
| 24 | Permissions, approvals, policy enforcement | not_started | | |
| 25 | Error taxonomy and recovery/retry behavior | not_started | | |
| 26 | Debug logs, trace bundles, model payload capture | not_started | | |
| 27 | TUI rendering and interaction model | not_started | | |
| 28 | Eval harness, smoke tests, golden payloads, trajectory traces | not_started | | |
| 29 | Runtime hygiene and product-name leakage prevention | not_started | | |
| 30 | Final skill packaging and usage workflow | not_started | | |
