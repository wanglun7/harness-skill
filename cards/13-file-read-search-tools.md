# Mechanism 13: File Read/Search Tools

Mechanism: File read/search tools
Status: complete

Claude source:
- `src/tools.ts:193-204` registers `GlobTool`, `GrepTool`, and `FileReadTool` in the base tool list, with `GlobTool`/`GrepTool` omitted only when embedded search tools are available.
- `src/tools.ts:271-288` simple mode keeps only Bash, Read, and Edit; `src/tools/REPLTool/primitiveTools.ts:28-35` keeps Read/Glob/Grep as REPL primitives even when the base list hides dedicated search tools.
- `src/tools/FileReadTool/prompt.ts:5-12` names the model-visible tool `Read`, defines the unchanged-file stub, and caps default reads at 2000 lines.
- `src/tools/FileReadTool/prompt.ts:27-48` tells the model that `file_path` must be absolute, results are line-numbered, images/PDFs/notebooks are supported, directories are not, and directory reads should use Bash `ls`.
- `src/tools/FileReadTool/FileReadTool.ts:227-242` defines the `Read` input schema: `file_path`, optional `offset`, optional `limit`, and optional PDF `pages`.
- `src/tools/FileReadTool/FileReadTool.ts:248-331` defines typed output variants for text, image, notebook, PDF, extracted PDF parts, and `file_unchanged`.
- `src/tools/FileReadTool/FileReadTool.ts:337-417` marks Read as strict, read-only, concurrency-safe, permission-checked, model-classifier-addressable by path, and not searchable in transcript UI because the UI summary is not the model serialization.
- `src/tools/FileReadTool/FileReadTool.ts:652-718` maps Read output to model-facing tool results: images become image blocks; notebooks map through notebook cells; PDFs/parts return metadata; unchanged reads return the stub; text returns numbered content plus optional security reminder or empty/short-file reminders.
- `src/tools/FileReadTool/FileReadTool.ts:1019-1070` reads text through `readFileInRange()`, validates token count, stores `readFileState`, notifies file-read listeners, logs file operation, and emits `tengu_session_file_read`.
- `src/utils/readFileInRange.ts:73-122` rejects directories, uses an in-memory fast path for regular files under 10MB, enforces `maxBytes` before full read unless truncation is requested, and streams larger files/devices.
- `src/utils/readFileInRange.ts:128-194` strips BOM, normalizes CRLF, selects only the requested line range, and returns line/byte counts and mtime.
- `src/tools/FileReadTool/UI.tsx:30-65` renders tool use as a path plus optional page or line range; `src/tools/FileReadTool/UI.tsx:77-164` renders summaries such as read image/PDF/line count/unchanged and compact error labels, not file contents.
- `src/tools/GrepTool/GrepTool.ts:33-90` defines model-visible Grep parameters for regex, path, glob, output mode, context lines, line numbers, case-insensitive search, type filter, pagination, and multiline search.
- `src/tools/GrepTool/GrepTool.ts:93-108` excludes VCS directories and defaults uncapped requests to `head_limit = 250`, with explicit `0` as the unlimited escape hatch.
- `src/tools/GrepTool/GrepTool.ts:160-198` marks Grep as strict, read-only, concurrency-safe, and a search command.
- `src/tools/GrepTool/GrepTool.ts:201-253` validates path existence, skips UNC path I/O during validation, checks read permissions, and exposes transcript search text as content or filenames.
- `src/tools/GrepTool/GrepTool.ts:254-309` maps Grep output to model-facing content, count summaries, or file lists with pagination markers.
- `src/tools/GrepTool/GrepTool.ts:310-441` builds ripgrep arguments for hidden files, VCS exclusions, max column width, multiline/case/mode/context/type/glob flags, read-deny ignore patterns, and plugin-cache exclusions, then calls `ripGrep()`.
- `src/tools/GrepTool/GrepTool.ts:443-575` shapes ripgrep results for content/count/files modes, applies `head_limit` and `offset`, relativizes paths, and sorts file matches by mtime with deterministic test ordering.
- `src/tools/GrepTool/UI.tsx:127-187` renders search use as pattern/path and renders results as content/count/file summaries.
- `src/tools/GlobTool/GlobTool.ts:26-52` defines Glob input/output: pattern, optional path, duration, filenames, file count, and truncation status.
- `src/tools/GlobTool/GlobTool.ts:57-153` marks Glob as read-only/concurrency-safe search, validates an optional directory path, checks read permissions, and exposes filenames for transcript search.
- `src/tools/GlobTool/GlobTool.ts:154-198` calls shared `glob()` with a default limit of 100, relativizes filenames, and maps empty/truncated results to model text.
- `src/tools/GlobTool/UI.tsx:11-53` renders Glob as `Search` and reuses Grep's result renderer.
- `src/utils/glob.ts:66-130` implements Glob with ripgrep `--files --glob --sort=modified`, optional `--no-ignore`/`--hidden`, read-deny ignore patterns, plugin-cache exclusions, absolute-path conversion, limit/offset, and truncation.
- `src/utils/permissions/filesystem.ts:837-851` extracts read-deny patterns for search hiding; `src/utils/permissions/filesystem.ts:1030-1095` checks original/resolved paths, blocks UNC and suspicious Windows patterns for approval, and applies read-deny rules before allows.
- `src/utils/ripgrep.ts:345-440` codesigns/tests ripgrep, treats exit code 1 as no matches, surfaces critical errors, retries EAGAIN with `-j 1`, and returns partial lines on timeout/buffer overflow when available.
- `src/utils/sessionFileAccessHooks.ts:1-5` documents analytics hooks for session memory/transcript access through Read/Grep/Glob; `src/utils/sessionFileAccessHooks.ts:75-112` parses Read/Grep/Glob inputs to detect session file access.
- `src/bridge/sessionRunner.ts:70-86` maps Read/FileReadTool to "Reading" and Glob/Grep to "Searching" activity labels.

Codex source:
- `codex-rs/core/src/tools/spec_plan.rs:155-237` builds the model-visible specs from planned tool runtimes and hosted specs.
- `codex-rs/core/src/tools/spec_plan.rs:563-623` adds shell tools as the environment-backed file exploration path, but this card does not treat shell file reads/searches as a dedicated file read/search mechanism because shell lifecycle is mechanism 15.
- `codex-rs/core/src/tools/spec_plan.rs:644-693` adds core utility tools and `apply_patch`; no generic model-visible text `Read`, `Grep`, or `Glob` runtime is registered in the cited core plan.
- `codex-rs/core/src/tools/handlers/view_image_spec.rs:15-49` defines the model-visible `view_image` tool for local image files, with `path`, optional `detail`, optional `environment_id`, and an output schema containing `image_url` and `detail`.
- `codex-rs/core/src/tools/handlers/view_image.rs:70-85` registers `view_image` as a parallel-capable runtime handler.
- `codex-rs/core/src/tools/handlers/view_image.rs:88-187` gates image reads on model image-input support, parses arguments, resolves the target environment, joins the path to the environment cwd, builds a filesystem sandbox context, validates metadata, and reads bytes through the environment filesystem.
- `codex-rs/core/src/tools/handlers/view_image.rs:189-227` chooses original vs resized detail, converts bytes to a prompt image data URL, emits image-view turn items, and returns typed output.
- `codex-rs/core/src/tools/handlers/view_image.rs:238-269` hides image bytes from log preview and maps output to a model-facing input-image `FunctionCallOutput`.
- `codex-rs/core/src/tools/spec_plan_tests.rs:620-658` tests that environment availability gates `view_image` and that multiple environments add `environment_id` to its visible spec.
- `codex-rs/app-server-protocol/src/protocol/v2/fs.rs:7-23` defines client/server `fs/readFile` as absolute-path input and base64 output.
- `codex-rs/app-server-protocol/src/protocol/v2/fs.rs:60-110` defines `fs/getMetadata` and `fs/readDirectory`, including file/directory/symlink metadata and direct child entries.
- `codex-rs/app-server/src/request_processors/fs_processor.rs:64-77` serves `fs/readFile` by reading bytes from the local executor filesystem and returning base64; `codex-rs/app-server/src/request_processors/fs_processor.rs:114-140` serves metadata and directory reads through the same filesystem abstraction.
- `codex-rs/file-system/src/lib.rs:187-220` defines `ExecutorFileSystem` read operations: `canonicalize`, `read_file`, chunked `read_file_stream`, and UTF-8 `read_file_text`.
- `codex-rs/app-server-protocol/src/protocol/common.rs:1153-1177` defines legacy and session-oriented fuzzy file search RPC methods.
- `codex-rs/app-server-protocol/src/protocol/common.rs:1491-1568` defines fuzzy search params/results/session update/completion payloads: query, roots, cancellation token, path, file name, match type, score, indices, and notifications.
- `codex-rs/app-server/src/request_processors/search.rs:39-79` handles one-shot fuzzy file search with cancellation tokens that cancel prior matching requests.
- `codex-rs/app-server/src/request_processors/search.rs:81-133` starts, updates, and stops long-lived fuzzy search sessions by session id.
- `codex-rs/file-search/src/lib.rs:90-127` defines fuzzy search snapshots and options: limit, exclusions, thread count, match-index computation, and gitignore handling.
- `codex-rs/file-search/src/lib.rs:158-210` creates a search session with override matcher, Nucleo matcher, cancellation flag, and separate matcher/walker worker threads.
- `codex-rs/file-search/src/lib.rs:219-287` implements the CLI/run path; a missing pattern lists the directory, while a pattern runs fuzzy search and reports truncation.
- `codex-rs/file-search/src/lib.rs:289-307` runs a one-shot search by creating a session, updating the query, waiting for completion, and returning matches/total count.
- `codex-rs/app-server/tests/suite/fuzzy_file_search.rs:54-99` waits for fuzzy search update notifications and validates session id/query/file expectations; `codex-rs/app-server/tests/suite/fuzzy_file_search.rs:101-160` validates completion and missing-session errors.

Runtime trace evidence:
- Claude Read trace: tool registry includes `FileReadTool`; the model sees the `Read` schema/prompt; runtime validates path and permissions, calls `readFileInRange()` for text, caches mtime/content in `readFileState`, emits file-read listeners and analytics, then `mapToolResultToToolResultBlockParam()` returns numbered text, image blocks, PDF/notebook metadata, or `FILE_UNCHANGED_STUB` (`src/tools.ts:193-204`, `src/tools/FileReadTool/FileReadTool.ts:337-417`, `src/tools/FileReadTool/FileReadTool.ts:1019-1070`, `src/tools/FileReadTool/FileReadTool.ts:652-718`).
- Claude Grep trace: model arguments flow through validation and read-permission checks, ripgrep arguments are built with search/output/pagination/deny-pattern options, `ripGrep()` returns lines, and the tool maps results into content/count/file-list model output and a separate UI summary (`src/tools/GrepTool/GrepTool.ts:201-253`, `src/tools/GrepTool/GrepTool.ts:310-441`, `src/tools/GrepTool/GrepTool.ts:443-575`, `src/tools/GrepTool/GrepTool.ts:254-309`).
- Claude Glob trace: model arguments flow through directory validation/read permission, shared `glob()` executes ripgrep `--files`, results are limited/relativized/truncation-marked, and the model receives either a file list or "No files found" (`src/tools/GlobTool/GlobTool.ts:94-198`, `src/utils/glob.ts:66-130`).
- Codex model-visible image trace: `view_image` appears only when an environment exists and model image input is supported; runtime reads bytes through the selected environment filesystem under sandbox context, emits image-view turn items, and returns an input-image function-call output (`codex-rs/core/src/tools/spec_plan_tests.rs:620-658`, `codex-rs/core/src/tools/handlers/view_image.rs:88-227`, `codex-rs/core/src/tools/handlers/view_image.rs:238-269`).
- Codex client/app file trace: app-server `fs/readFile`, `fs/getMetadata`, `fs/readDirectory`, and fuzzy search RPCs operate through protocol/request processors and file-search sessions; these are client/app-server surfaces, not generic model-visible text Read/Grep/Glob tools in the cited core registry (`codex-rs/app-server-protocol/src/protocol/v2/fs.rs:7-110`, `codex-rs/app-server/src/request_processors/fs_processor.rs:64-140`, `codex-rs/app-server/src/request_processors/search.rs:39-133`).

What the model sees:
- Claude: dedicated `Read`, `Grep`, and `Glob` schemas/prompts. Read requires an absolute `file_path` and optional range/page controls; Grep exposes ripgrep-like regex/search options; Glob exposes filename pattern/path. The model receives tool results shaped by each mapper, not raw filesystem objects (`src/tools/FileReadTool/prompt.ts:27-48`, `src/tools/FileReadTool/FileReadTool.ts:227-242`, `src/tools/GrepTool/GrepTool.ts:33-90`, `src/tools/GlobTool/GlobTool.ts:26-52`).
- Claude: Read text is line-numbered and may include system-reminder text; image reads are model image blocks; empty/short files and unchanged files become explicit reminders/stubs. Grep content/count/file-list and Glob file-list outputs include pagination/truncation markers when applicable (`src/tools/FileReadTool/FileReadTool.ts:652-718`, `src/tools/GrepTool/GrepTool.ts:254-309`, `src/tools/GlobTool/GlobTool.ts:177-198`).
- Codex: no generic model-visible text `Read`, `Grep`, or `Glob` source was found in the core tool plan cited for this card. The source-proven model-visible file reader here is `view_image`, which exposes a local image path and returns image content to the model (`codex-rs/core/src/tools/spec_plan.rs:563-693`, `codex-rs/core/src/tools/handlers/view_image_spec.rs:15-49`, `codex-rs/core/src/tools/handlers/view_image.rs:247-262`).
- Codex: generic filesystem read/list and fuzzy search payloads are app-server/client protocol objects, not model tool schemas in the cited core registry (`codex-rs/app-server-protocol/src/protocol/v2/fs.rs:7-110`, `codex-rs/app-server-protocol/src/protocol/common.rs:1491-1568`).

What the runtime does:
- Claude: separates text, image, PDF, notebook, and unchanged-file read paths behind one `Read` tool; text uses line-range reads and token validation, while images/PDFs/notebooks have their own output variants and model serialization (`src/tools/FileReadTool/FileReadTool.ts:248-331`, `src/tools/FileReadTool/FileReadTool.ts:652-718`, `src/tools/FileReadTool/FileReadTool.ts:1019-1070`).
- Claude: search tools run ripgrep with guardrails: VCS exclusions, max columns, deny-pattern glob exclusions, plugin-cache exclusions, pagination, hidden-file handling, and explicit error/partial-output behavior (`src/tools/GrepTool/GrepTool.ts:310-441`, `src/utils/glob.ts:91-130`, `src/utils/ripgrep.ts:345-440`).
- Claude: read/search permission enforcement is shared through `checkReadPermissionForTool()` and read-deny patterns, with explicit handling for symlink-resolved paths, UNC paths, and suspicious Windows path patterns (`src/utils/permissions/filesystem.ts:837-851`, `src/utils/permissions/filesystem.ts:1030-1095`).
- Codex: `view_image` uses the turn environment's filesystem abstraction and sandbox context to read the image; app-server `fs/*` methods use `ExecutorFileSystem`; fuzzy search uses a dedicated `file-search` crate with Nucleo matching and worker threads (`codex-rs/core/src/tools/handlers/view_image.rs:139-187`, `codex-rs/file-system/src/lib.rs:187-220`, `codex-rs/file-search/src/lib.rs:158-210`).

What gets persisted:
- Claude: Read stores last-read content/mtime/range in `readFileState` for unchanged-read deduplication and updates memory-related tracking when applicable; session/transcript persistence is handled by the general tool-result/message path covered in mechanism 12 (`src/tools/FileReadTool/prompt.ts:7-8`, `src/tools/FileReadTool/FileReadTool.ts:1019-1038`, `src/tools/FileReadTool/FileReadTool.ts:1056-1070`).
- Claude: Grep/Glob outputs are normal tool outputs with `maxResultSizeChars` thresholds (`GrepTool` 20,000 chars; `GlobTool` 100,000 chars) and are not cited here as having file-read-style content caches (`src/tools/GrepTool/GrepTool.ts:160-165`, `src/tools/GlobTool/GlobTool.ts:57-60`).
- Codex: `view_image` emits `ImageView` turn items and returns a function-call output; persistence follows the general conversation/history path from mechanism 12. Fuzzy search sessions are stored in an in-memory `fuzzy_search_sessions` map for the active app-server processor, and no durable fuzzy-search persistence is cited here (`codex-rs/core/src/tools/handlers/view_image.rs:217-227`, `codex-rs/app-server/src/request_processors/search.rs:23-35`, `codex-rs/app-server/src/request_processors/search.rs:81-133`).
- Codex: app-server `fs/readFile` returns base64 bytes directly and does not persist file contents in the cited processor path (`codex-rs/app-server/src/request_processors/fs_processor.rs:64-77`).

What the user/TUI sees:
- Claude: Read tool use displays a file path plus page/range metadata; results display summaries only, such as read image size, read PDF size, read cell count, read line count, or unchanged. The UI intentionally does not render full file contents (`src/tools/FileReadTool/UI.tsx:30-65`, `src/tools/FileReadTool/UI.tsx:77-164`).
- Claude: Grep/Glob tool use displays pattern/path; results render as search summaries with optional expanded content/file lists; Glob reuses Grep's result renderer (`src/tools/GrepTool/UI.tsx:127-187`, `src/tools/GlobTool/UI.tsx:11-53`).
- Claude: bridge/session activity labels classify Read as "Reading" and Glob/Grep as "Searching" (`src/bridge/sessionRunner.ts:70-86`).
- Codex: `view_image` emits `ImageViewItem` start/completion events carrying the call id and path; fuzzy file search emits app-server notifications for session updates/completion (`codex-rs/core/src/tools/handlers/view_image.rs:217-222`, `codex-rs/app-server-protocol/src/protocol/common.rs:1554-1568`).

What debug/eval surfaces exist:
- Claude: Read logs file operations and `tengu_session_file_read`; session file access hooks parse Read/Grep/Glob inputs for session-memory/transcript access analytics; ripgrep logs retry/error details and `tengu_ripgrep_eagain_retry` on EAGAIN retry (`src/tools/FileReadTool/FileReadTool.ts:1060-1070`, `src/utils/sessionFileAccessHooks.ts:1-5`, `src/utils/sessionFileAccessHooks.ts:75-112`, `src/utils/ripgrep.ts:390-440`).
- Claude: no dedicated `Read`/`Grep`/`Glob` test file was located during this cycle with a targeted `rg` over `src` test/spec filenames; this card cites implementation and telemetry/debug surfaces rather than claiming a test suite.
- Codex: `spec_plan_tests` verifies `view_image` model visibility and environment-id schema behavior; `view_image.rs` unit tests cover log-preview/code-mode output behavior; app-server fuzzy-search tests validate update/completion notifications and missing-session errors (`codex-rs/core/src/tools/spec_plan_tests.rs:620-658`, `codex-rs/core/src/tools/handlers/view_image.rs:272-320`, `codex-rs/app-server/tests/suite/fuzzy_file_search.rs:54-160`).

Shared invariant:
- Keep file exploration read-only, bounded, permission/sandbox-aware, and projection-separated. The model-visible contract, runtime filesystem operation, persisted/cache state, user display, and debug/eval output are separate surfaces and require separate source evidence.

Claude-specific design:
- Claude has first-class model-visible text/file search primitives: `Read`, `Grep`, and `Glob`. They are not merely shell aliases; each has schemas, permission checks, output shaping, UI renderers, and search/read classification (`src/tools/FileReadTool/FileReadTool.ts:337-417`, `src/tools/GrepTool/GrepTool.ts:160-198`, `src/tools/GlobTool/GlobTool.ts:57-153`).
- Claude's `Read` handles text/images/PDFs/notebooks under one tool and tells the model to use Bash `ls` for directories instead of inventing a directory read mode (`src/tools/FileReadTool/prompt.ts:40-48`, `src/tools/FileReadTool/FileReadTool.ts:248-331`).
- Claude's search path is ripgrep-centered, with explicit pagination, path relativization, deny-pattern projection, plugin-cache exclusions, and mtime sorting for file-list mode (`src/tools/GrepTool/GrepTool.ts:310-575`, `src/utils/glob.ts:66-130`).

Codex-specific design:
- Codex's source-proven model-visible file reader in this mechanism is `view_image`, not a generic text `Read` tool. Generic text file reads/searches are available through shell tools or external/MCP tools in other mechanisms, and through app-server/client filesystem/search RPCs for UI/client workflows (`codex-rs/core/src/tools/spec_plan.rs:563-693`, `codex-rs/core/src/tools/handlers/view_image_spec.rs:15-49`, `codex-rs/app-server-protocol/src/protocol/v2/fs.rs:7-110`, `codex-rs/app-server-protocol/src/protocol/common.rs:1491-1568`).
- Codex isolates reusable filesystem access behind `ExecutorFileSystem`, allowing local/remote environment filesystem operations to share read/metadata/directory/stream contracts (`codex-rs/file-system/src/lib.rs:187-220`).
- Codex has a dedicated app-server fuzzy file search subsystem with one-shot cancellation and long-lived sessions, backed by the `file-search` crate and Nucleo worker threads; this is a product/client search surface, not a core generic model-visible Grep/Glob tool in the cited registry (`codex-rs/app-server/src/request_processors/search.rs:39-133`, `codex-rs/file-search/src/lib.rs:158-210`).

Gaps or unknowns:
- No Codex core source was found in this cycle for a dedicated model-visible generic text `Read`, `Grep`, or `Glob` tool equivalent to Claude's primitives. The card therefore marks Claude's dedicated text read/search primitives as source-specific.
- Codex shell-based file exploration is intentionally not analyzed here because shell/exec/background/interrupt tools are mechanism 15.
- Codex app-server filesystem read/list APIs include write/remove/copy neighbors in the same protocol file; this card cites only read/metadata/directory/search surfaces and does not infer edit/write behavior.
- Claude test evidence for file read/search was not found with the targeted test/spec search used in this cycle; implementation, UI, telemetry, and helper-runtime paths are cited instead.

Distilled harness rule:
- If a harness exposes model-visible file read/search primitives, define them as read-only tools with explicit schemas, permission/sandbox checks, result bounds, output-mode controls, and separate model/display projections. If a reference exposes file exploration only through shell, image-only view tools, app-server APIs, or UI search services, do not generalize that into a generic `Read`/`Grep`/`Glob` mechanism.

Anti-invention rule:
- Do not invent a file tool because a harness can access files. Claude proves `Read`/`Grep`/`Glob`; Codex proves `view_image`, app-server `fs/*`, and fuzzy file search. A shell command such as `rg`, `cat`, or `ls` is not a dedicated file read/search tool unless the source registers and projects it as one.

Verification pattern:
- Re-read the ledger before starting this mechanism.
- For a model-visible file read/search claim, trace registry inclusion -> schema/prompt -> validation -> permission/sandbox check -> filesystem/search execution -> result shaping -> model result -> UI result -> persistence/cache/debug/test surface.
- For absence claims, cite the tool-spec planning path and list what is actually registered in the relevant source segment; then mark adjacent shell/MCP/app-server mechanisms as separate rather than filling the gap by inference.
- For search tools, verify ignore/deny behavior, default caps, pagination, path relativization, error handling, and truncation markers independently.
- For file reads, verify binary/media handling, directory behavior, size/token bounds, line numbering/range semantics, unchanged-read behavior, and whether user display shows content or only a summary.

Skill text candidate:
- When studying file exploration in a coding-agent harness, first classify the surface: model-visible text read/search tool, model-visible media viewer, shell-mediated file exploration, MCP/external file tool, app/client filesystem API, or UI-only fuzzy search.
- Only call a mechanism "general file read/search" if the source provides model-visible schemas and runtime handlers for text reads or searches. Otherwise, describe the exact source-specific surface.
- Keep read/search projection separate: model output may include line-numbered text or image blocks; UI may show only summaries; persistence may cache mtime/content or only record normal tool history.
- Require source citations for bounds and safety: file size/token limits, default line/result caps, ignore/deny patterns, sandbox contexts, symlink/UNC handling, and truncation/pagination notices.
- Defer shell `cat`/`sed`/`rg`/`ls` behavior to the shell mechanism unless the source wraps those behaviors as dedicated file tools.
