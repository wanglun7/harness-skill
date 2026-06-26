# Mechanism 30: Final skill packaging and usage workflow

Mechanism: Final skill packaging and usage workflow
Status: complete

Claude source:
- Skill discovery path and metadata-only token estimate: `src/skills/loadSkillsDir.ts:67-105`.
- Frontmatter parsing fields for description, when-to-use, tool/model/context controls, and user invocability: `src/skills/loadSkillsDir.ts:180-265`.
- Skill command creation and full-content prompt expansion with base directory, arguments, skill dir/session substitutions, and MCP shell-command guard: `src/skills/loadSkillsDir.ts:267-401`.
- Directory-format skill loading from `skill-name/SKILL.md`: `src/skills/loadSkillsDir.ts:403-470`.
- Skill listing budget, truncation, and discovery prompt: `src/tools/SkillTool/prompt.ts:20-50`, `src/tools/SkillTool/prompt.ts:70-195`.
- Skill execution validation, permission checks, inline/forked invocation, and telemetry redaction: `src/tools/SkillTool/SkillTool.ts:118-288`, `src/tools/SkillTool/SkillTool.ts:331-430`, `src/tools/SkillTool/SkillTool.ts:432-578`, `src/tools/SkillTool/SkillTool.ts:580-700`.

Codex source:
- Skill metadata model: `codex-rs/core-skills/src/model.rs:14-68`.
- Skill root discovery, scope ordering, repo/user/system/admin roots, and skill file parsing: `codex-rs/core-skills/src/loader.rs:155-233`, `codex-rs/core-skills/src/loader.rs:283-405`, `codex-rs/core-skills/src/loader.rs:639-725`.
- Available-skills prompt text, progressive-disclosure usage instructions, metadata budget, truncation, warnings, and telemetry side effects: `codex-rs/core-skills/src/render.rs:17-60`, `codex-rs/core-skills/src/render.rs:62-84`, `codex-rs/core-skills/src/render.rs:143-200`, `codex-rs/core-skills/src/render.rs:202-320`, `codex-rs/core-skills/src/render.rs:324-445`.
- Explicit skill mention detection and full `SKILL.md` injection: `codex-rs/core-skills/src/injection.rs:58-111`, `codex-rs/core-skills/src/injection.rs:133-200`, `codex-rs/core-skills/src/injection.rs:230-347`.
- Skills extension lifecycle, catalog contribution, turn-input injection, read-main-prompt path, and tool contribution: `codex-rs/ext/skills/src/extension.rs:53-142`, `codex-rs/ext/skills/src/extension.rs:144-168`, `codex-rs/ext/skills/src/extension.rs:170-292`, `codex-rs/ext/skills/src/extension.rs:294-379`.
- Orchestrator skill list/read tools and namespaced tool schema: `codex-rs/ext/skills/src/tools/mod.rs:33-160`.

Runtime trace evidence:
- Claude proves a skill is packaged as directory metadata plus full markdown body. Discovery uses frontmatter/name/description/when-to-use for listing and token estimation, but full `markdownContent` is only expanded when `getPromptForCommand()` is invoked (`loadSkillsDir.ts:96-105`, `loadSkillsDir.ts:180-265`, `loadSkillsDir.ts:344-399`).
- Claude proves progressive disclosure in the model-facing prompt: skill listings are budgeted and descriptions are capped; the Skill tool prompt instructs the model to invoke a matching skill before answering and says the list appears in system reminders (`SkillTool/prompt.ts:20-50`, `SkillTool/prompt.ts:173-195`).
- Claude proves invocation has runtime gates: validation checks existence, disabled model invocation, and prompt type; permissions can deny, allow, auto-allow safe skills, or ask the user; execution can be inline or forked (`SkillTool.ts:354-430`, `SkillTool.ts:432-578`, `SkillTool.ts:580-700`).
- Codex proves a skill catalog is model-visible metadata, while full content is loaded only for selected/mentioned skills. `render_available_skills_body()` emits roots, available skill lines, and how-to-use instructions; `build_available_skills()` applies a metadata budget and warnings; turn-input contribution reads selected main prompts and injects `SkillInstructions` fragments (`render.rs:17-84`, `render.rs:160-320`, `extension.rs:170-292`).
- Codex proves explicit mention resolution is ordered and path-aware: structured skill selections and `$skill` text mentions are collected, ambiguous plain names are avoided, `skill://` and `SKILL.md` paths are treated as skill mentions, and selected prompts are read through `read_skill()` (`injection.rs:133-200`, `injection.rs:230-347`, `extension.rs:319-339`).
- The final local artifact `SKILL.md` follows the same invariant: keep the trigger metadata short, keep the body procedural, and route detailed mechanism evidence to `cards/` and `ledger.md` instead of duplicating all source evidence inline.

What the model sees:
- Claude model sees a compact skill listing and a Skill tool contract; after invocation, it sees the processed skill body as prompt text (`SkillTool/prompt.ts:70-195`, `loadSkillsDir.ts:344-399`).
- Codex model sees a rendered available-skills block, how-to-use instructions, and selected skill body fragments for explicitly mentioned skills (`render.rs:17-84`, `extension.rs:202-248`).
- Future users of this final skill should see `SKILL.md` metadata and procedural body first; detailed mechanism cards are loaded only as needed.

What the runtime does:
- Claude runtime loads `skill-name/SKILL.md`, parses frontmatter, creates a prompt command, validates invocation, applies permissions, expands arguments/placeholders, and executes inline or in a forked agent (`loadSkillsDir.ts:403-470`, `SkillTool.ts:331-700`).
- Codex runtime discovers skills across scoped roots, parses `SKILL.md`, renders a budgeted catalog, detects explicit mentions, reads selected main prompts, injects them into turn context, and exposes list/read tools for orchestrator skills (`loader.rs:155-233`, `loader.rs:283-405`, `loader.rs:639-725`, `extension.rs:53-379`, `tools/mod.rs:33-160`).
- The final workflow runtime is file-based: read `ledger.md`, read selected card(s), inspect cited source, write/update one card and ledger entry, then package the durable skill.

What gets persisted:
- Claude persists no new state merely by listing a skill; skill usage can be recorded for ranking/telemetry, and skill files persist as `skill-name/SKILL.md` plus optional resources (`SkillTool.ts:618-620`, `loadSkillsDir.ts:403-470`).
- Codex persists skill metadata in runtime state for the thread/turn, emits warnings/events, and may track invocations; skill files persist on disk or through provider resources (`extension.rs:57-72`, `extension.rs:279-287`, `injection.rs:82-108`).
- This distillation persists `ledger.md`, `cards/01...30.md`, and root `SKILL.md`.

What the user/TUI sees:
- Claude users can see skill availability through listings and may see permission prompts such as `Execute skill: <name>` (`SkillTool/prompt.ts:173-195`, `SkillTool.ts:569-577`).
- Codex users can see warnings when skill metadata or selected prompt content is truncated or fails to load (`render.rs:213-249`, `extension.rs:228-258`, `extension.rs:341-345`).
- The final user-facing artifact is a reusable skill file plus card/ledger references, not a long essay.

What debug/eval surfaces exist:
- Claude logs skill invocation telemetry with redacted/general and privileged fields, plus skill execution context (`SkillTool.ts:152-203`, `SkillTool.ts:675-700`).
- Codex records skill render side effects/metrics, emits warnings, and has bounded handle validation for orchestrator skill tools (`render.rs:253-320`, `extension.rs:341-345`, `tools/mod.rs:142-160`).
- The local validation surface is structural: verify all cards exist, all statuses are complete, all required sections appear, final `SKILL.md` has valid frontmatter, and no reference source files were edited.

Shared invariant:
- Both references implement skills through progressive disclosure: compact metadata is visible first; full skill content is loaded only when selected or invoked; warnings/permissions/telemetry/debug are separate runtime surfaces.
- A reusable harness skill should therefore keep `SKILL.md` concise, route detailed evidence to references/cards, and provide exact rules for when to open those references.

Claude-specific design:
- Claude's skill system is tool-invoked through `SkillTool`, supports inline/forked prompt commands, frontmatter controls such as allowed tools/model/context, and directory-format loading under `skill-name/SKILL.md`.

Codex-specific design:
- Codex's skill system is extension-driven, renders available-skills instructions into context, supports scoped roots and orchestrator resources, detects explicit `$skill`/path mentions, and injects full selected skill content into turn input.

Gaps or unknowns:
- The final local skill does not create `agents/openai.yaml`; the minimum required artifact is root `SKILL.md`. UI metadata can be added later if installation into a live skill registry requires it.
- No subagent forward-test was run; local validation instead checks structure and current artifacts.

Distilled harness rule:
- Package source-grounded harness knowledge as a progressive-disclosure skill: concise trigger metadata, procedural body, strict ledger workflow, card references for detailed evidence, and a completion audit that proves every source-grounded deliverable exists before claiming completion.

Anti-invention rule:
- Do not add mechanisms to the final `SKILL.md` that are not already proven by cards or cited source. Do not convert local card summaries into uncited best practices. Do not make the skill depend on a target implementation.

Verification pattern:
- Verify `SKILL.md` has only `name` and `description` frontmatter fields, includes strict-order workflow, names all 30 mechanisms, defines the card template, points to cards by range, includes reuse and completion-audit rules, and avoids target-implementation dependencies.
- Verify ledger row 30 points to this card and final `SKILL.md`, all rows are complete, all 30 card files exist, and reference source worktrees have no edits.

Skill text candidate:
- Use a compact `SKILL.md` as the entrypoint and keep detailed mechanism evidence in card files. Future agents should read the ledger first, select only the relevant mechanism card, verify cited source when needed, and preserve strict separation between model-visible, runtime, persisted, TUI, debug/trace, and eval surfaces. The skill must forbid invention, target dependence, and source-specific generalization while giving a repeatable cycle for card creation, reuse, and completion audit.
