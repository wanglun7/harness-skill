---
name: coding-agent-harness-distillation
description: Source-faithful coding-agent harness research and reuse workflow. Use when comparing Claude Code and Codex harness architecture, extracting mechanisms from their source code, building mechanism cards, auditing source citations, or packaging reusable harness rules without inventing target-specific implementation details.
---

# Coding-Agent Harness Distillation

Use this skill to understand, compare, and reuse coding-agent harness mechanisms from Claude Code and Codex with strict source grounding.

## Core Rule

Do not start from best practices, diagrams, or assumptions. Start from source files, functions, classes, types, tests, runtime trace paths, and existing mechanism cards. Every reusable rule must name the exact source evidence that justifies it.

Keep these surfaces separate in every note:
- Model-visible behavior
- Runtime behavior
- Persisted state
- Terminal/TUI display
- Debug/trace behavior
- Eval/test behavior

If only Claude or only Codex implements a mechanism, mark it source-specific. Do not generalize it.

## Inputs

Require or infer these inputs before doing work:
- `claude_source_root`: Claude Code source root.
- `codex_source_root`: Codex repository root.
- `ledger_path`: durable ledger file, usually `ledger.md`.
- `cards_dir`: mechanism-card directory, usually `cards/`.
- `mechanism_order`: the ordered mechanism list below.

Do not depend on any target implementation. Use a target codebase only if the user explicitly asks to apply the distilled rules later.

## Work Cycle

For each cycle:

1. Read the ledger before doing anything else.
2. Select the next incomplete mechanism in strict order.
3. Add a `Before` entry to the ledger and mark the mechanism `in_progress`.
4. Locate exact source files/functions/types/tests for that mechanism.
5. Read only relevant source deeply.
6. Fill one mechanism card with the required card fields.
7. Update the ledger with card path, key citations, `After` notes, and next incomplete mechanism.
8. Do not proceed to a later mechanism until the current card is complete.

Treat chat history as non-authoritative. Re-read the ledger and relevant source each cycle.

## Mechanism Order

1. Repository/source map and harness boundary definition
2. Model/provider abstraction
3. Main turn loop and stop conditions
4. Streaming protocol and assistant text/tool interleaving
5. Message normalization and provider payload construction
6. System prompt and instruction layering
7. Dynamic context assembly
8. Token budgeting and context limits
9. Compaction: manual, auto, reactive, post-compact rehydration
10. Tool registry and model-visible schema projection
11. Tool dispatch lifecycle and scheduling/concurrency
12. Tool result projection: raw output, model output, display output, persisted output
13. File read/search tools
14. File edit/write/patch/diff tools
15. Shell/exec/background process/interrupt tools
16. Web/network tools
17. Task/todo/plan tools and task state
18. Subagent/fork/delegation model
19. Worktree/isolation/sandbox behavior
20. Agent messaging/inbox/notifications
21. Memory: session, project, long-term, extraction lifecycle
22. Skills/plugins/custom instructions
23. Session persistence, transcript storage, resume, repair
24. Permissions, approvals, policy enforcement
25. Error taxonomy and recovery/retry behavior
26. Debug logs, trace bundles, model payload capture
27. TUI rendering and interaction model
28. Eval harness, smoke tests, golden payloads, trajectory traces
29. Runtime hygiene and product-name leakage prevention
30. Final skill packaging and usage workflow

## Card Template

Each card must contain exactly these sections:

```markdown
Mechanism:
Status: not_started | in_progress | complete
Claude source:
Codex source:
Runtime trace evidence:
What the model sees:
What the runtime does:
What gets persisted:
What the user/TUI sees:
What debug/eval surfaces exist:
Shared invariant:
Claude-specific design:
Codex-specific design:
Gaps or unknowns:
Distilled harness rule:
Anti-invention rule:
Verification pattern:
Skill text candidate:
```

Use source-relative citations such as `src/query.ts:181-239` or `codex-rs/core/src/session/turn.rs:140-245`. Use absolute paths only when the user needs a local clickable file link.

## Source Search Pattern

For each mechanism, search narrowly and then read deeply:

1. Search names from the mechanism title plus likely runtime nouns.
2. Search both references independently.
3. Prefer exact owners over callers:
   - entrypoint
   - type definition
   - builder/loader
   - runtime executor
   - persistence writer
   - UI renderer
   - debug/eval test
4. Read the smallest source ranges that prove behavior.
5. If behavior appears only in tests, label it eval-only.
6. If behavior appears only in comments, label it comment evidence unless implementation confirms it.

Use `rg` first. Prefer structured parsers or source-level types over ad hoc string interpretation when a source has them.

## Distillation Rules

Use these rules when turning source evidence into reusable guidance:

- Build a source map before architecture claims.
- Trace one concrete runtime path before naming a mechanism.
- Separate model payload, runtime event, persisted artifact, display item, and debug/eval artifact.
- Preserve source-specific shapes, names, gates, and constraints.
- Treat prompts, schemas, and tests as evidence for their own surface only.
- Do not claim runtime enforcement from prompt text unless runtime code enforces it.
- Do not claim model visibility from UI rendering code.
- Do not claim persistence from transient runtime state.
- Do not claim shared invariants until both references prove the same invariant.
- Mark gaps explicitly instead of filling them with plausible design.

## Mechanism Navigation

Use the existing card files as progressively disclosed references:

- For source-map and provider flow, read cards 01-05.
- For prompt/context/window management, read cards 06-09.
- For tools and tool results, read cards 10-16.
- For tasking, delegation, isolation, and messaging, read cards 17-20.
- For memory, skills, sessions, and permissions, read cards 21-24.
- For errors, observability, TUI, eval, and hygiene, read cards 25-29.
- For packaging and workflow, read card 30 and this `SKILL.md`.

When a user asks about a specific mechanism, read only the relevant card first, then inspect cited source files if current verification is needed.

## Reuse Workflow

When applying distilled knowledge to a new harness or design:

1. Name the mechanism and source cards used.
2. Restate the source-backed invariant.
3. List Claude-specific and Codex-specific details that must not be copied blindly.
4. Inspect the target only if the user asked for target application.
5. Map only proven equivalents.
6. Propose or implement only the smallest change that follows from source evidence.
7. Verify against the same surface matrix: model, runtime, persistence, TUI, debug/eval.

## Final Audit

Before claiming the distillation is complete, verify:

- Ledger has one cycle entry per mechanism, with `Before` and `After`.
- Mechanism status table marks all 30 mechanisms `complete`.
- `cards_dir` contains one card for each mechanism.
- Every card has all required sections.
- Every card cites Claude and/or Codex source, with source-specific gaps marked.
- `SKILL.md` exists with only `name` and `description` frontmatter fields.
- The final skill body explains strict order, source grounding, card format, progressive disclosure, reuse workflow, and completion audit.
- No runtime source files in Claude or Codex were edited.

If any item fails, keep the goal active and fix it before reporting completion.
