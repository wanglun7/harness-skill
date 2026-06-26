# Mechanism 16: Web/Network Tools

Mechanism: Web/network tools
Status: complete

Claude source:
- `src/tools.ts:193-217` registers `WebFetchTool` and `WebSearchTool` as base tools and has only a feature-gated optional `WebBrowserTool` import path; no `src/tools/WebBrowserTool` implementation exists in the cited source tree.
- `src/tools/WebFetchTool/WebFetchTool.ts:24-45` defines the model-visible `WebFetch` input schema (`url`, `prompt`) and runtime output shape (`bytes`, `code`, `codeText`, `result`, `durationMs`, `url`).
- `src/tools/WebFetchTool/WebFetchTool.ts:50-64` derives permission rule content from the URL hostname as `domain:{hostname}`; `src/components/permissions/WebFetchPermissionRequest/WebFetchPermissionRequest.tsx:12-28` mirrors that hostname-to-rule mapping in the permission request UI.
- `src/tools/WebFetchTool/WebFetchTool.ts:66-103` builds the strict `WebFetch` tool with search hint, `maxResultLength` 100k, user-facing name, read-only/concurrency-safe flags, defer behavior, hostname-based description, and auto-classifier input.
- `src/tools/WebFetchTool/WebFetchTool.ts:104-180` checks preapproved domains, deny/ask/allow permission rules, default ask behavior, and suggested allow-rule updates for hostname-scoped fetches.
- `src/tools/WebFetchTool/prompt.ts:1-21` tells the model that `WebFetch` fetches a URL, converts HTML to markdown, processes it with a prompt using a small model, is read-only, may return summarized results, has a 15-minute cache, requires a new call for cross-host redirects, and should prefer MCP/`gh` for certain sources.
- `src/tools/WebFetchTool/prompt.ts:23-45` defines the secondary model prompt used by `applyPromptToMarkdown`, with different quote/excerpt rules for preapproved versus non-preapproved domains.
- `src/tools/WebFetchTool/utils.ts:99-128` defines WebFetch limits: URL length 2000, HTTP content length 10 MB, fetch timeout 60 s, domain-check timeout 10 s, max redirects 10, and markdown length 100k.
- `src/tools/WebFetchTool/utils.ts:130-169` validates URLs, rejects username/password credentials, requires a multi-part hostname, and later upgrades `http` to `https`.
- `src/tools/WebFetchTool/utils.ts:176-203` calls Anthropic domain-info API for blocklist checks and caches allowed results; `src/tools/WebFetchTool/utils.ts:20-48` defines `DomainBlockedError`, `DomainCheckFailedError`, and `EgressBlockedError`.
- `src/tools/WebFetchTool/utils.ts:205-329` manually follows only permitted redirects, allowing same protocol/port and same host with optional `www` add/remove, stopping cross-host redirects as a model-visible redirect result, and treating proxy allowlist 403s as egress blocks.
- `src/tools/WebFetchTool/utils.ts:337-481` implements `getURLMarkdownContent`: cache lookup, URL upgrade, domain preflight, axios GET, binary content persistence, HTML-to-markdown conversion, raw text decoding, and 15-minute/50 MB LRU cache storage.
- `src/tools/WebFetchTool/utils.ts:484-530` implements `applyPromptToMarkdown`: truncates markdown, builds the secondary prompt, calls `queryHaiku` with `querySource: 'web_fetch_apply'`, no tools/MCP, and returns the first text block.
- `src/tools/WebFetchTool/WebFetchTool.ts:208-306` runs the WebFetch call, handles cross-host redirect response text, returns direct markdown for small preapproved markdown content, otherwise runs the secondary model prompt, appends persisted binary-path notes, and projects only `result` into the model tool-result block.
- `src/tools/WebFetchTool/UI.tsx:9-71` renders URL/prompt use messages, `Fetching...` progress, result bytes/status, optional verbose result, and truncated summary.
- `src/tools/WebFetchTool/preapproved.ts:1-12` states that preapproved hosts apply only to WebFetch GET requests and do not grant sandbox network access.
- `src/commands/clear/caches.ts:124-132` clears the WebFetch URL cache as part of session cache clearing.
- `src/tools/WebSearchTool/WebSearchTool.ts:25-37` defines the model-visible `WebSearch` input schema: `query`, optional `allowed_domains`, and optional `blocked_domains`.
- `src/tools/WebSearchTool/WebSearchTool.ts:76-84` creates an Anthropic server tool schema named `web_search` with type `web_search_20250305`, domain filters, and `max_uses: 8`.
- `src/tools/WebSearchTool/WebSearchTool.ts:86-150` parses Anthropic `server_tool_use` and `web_search_tool_result` content blocks into search summaries and result URLs.
- `src/tools/WebSearchTool/WebSearchTool.ts:152-222` builds `WebSearch` as a read-only/concurrency-safe tool, gates availability by provider/model support, and uses passthrough permission with a local-settings allow-rule suggestion.
- `src/tools/WebSearchTool/prompt.ts:1-33` tells the model to search current web information, cite sources with markdown links, optionally filter domains, and note that the tool is US-only.
- `src/tools/WebSearchTool/WebSearchTool.ts:235-253` validates non-empty query and forbids both allowed and blocked domains.
- `src/tools/WebSearchTool/WebSearchTool.ts:254-400` executes `WebSearch` by making a nested `queryModelWithStreaming` call with no normal tools/MCP and `extraToolSchemas: [web_search]`, consuming streamed server-tool events, emitting query/result progress, and returning extracted output with duration.
- `src/tools/WebSearchTool/WebSearchTool.ts:401-434` maps search output into model-visible tool-result text with summaries, JSON links, and a reminder to include markdown hyperlinks as sources.
- `src/tools/WebSearchTool/UI.tsx:8-100` summarizes search/result counts, renders query/domain filters, progress messages, final duration, and truncated summary.
- `src/services/api/claude.ts:2948-3011` accumulates `server_tool_use.web_search_requests` and `server_tool_use.web_fetch_requests` in usage.
- `src/utils/contextAnalysis.ts:165-182` counts `server_tool_use`, `web_search_tool_result`, and `web_fetch_tool_result` blocks as token-analysis "other" blocks.
- `src/utils/messages.ts:3030-3042` treats streaming `server_tool_use`, `web_search_tool_result`, and `web_fetch_tool_result` content blocks as tool-input stream mode.

Codex source:
- `codex-rs/protocol/src/config_types.rs:304-348` defines `WebSearchMode` (`disabled`, `cached`, `live`) plus `WebSearchToolConfig` fields for context size, allowed domains, and location.
- `codex-rs/core/src/config/mod.rs:2369-2392` resolves web-search mode from explicit config or feature flags and maps `[tools].web_search` config into `WebSearchConfig`; `codex-rs/core/src/config/mod.rs:2534-2568` resolves the per-turn web-search mode against the permission profile, preferring `live` under disabled permissions when allowed.
- `codex-rs/tui/src/cli.rs:64-66` documents `--search` as enabling live web search with no per-call approval; `codex-rs/tui/src/lib.rs:863-867` maps that legacy flag to `web_search="live"`.
- `codex-rs/protocol/src/openai_models.rs:283-377` defines `WebSearchToolType` (`text`, `text_and_image`) on model metadata.
- `codex-rs/core/src/tools/spec_plan.rs:259-285` emits hosted model web-search specs only when Responses Lite is not in use, standalone `web.run` is not available, and the provider advertises web-search capability.
- `codex-rs/core/src/tools/spec_plan.rs:576-585` defines standalone web-search enablement through namespace tools plus Responses Lite or `StandaloneWebSearch` feature; `codex-rs/core/src/tools/spec_plan.rs:935-950` filters extension executor `web.run` out when standalone web search is disabled or web-search mode is off.
- `codex-rs/core/src/tools/hosted_spec.rs:1-43` creates the hosted `ToolSpec::WebSearch`, mapping cached/live to `external_web_access: false/true`, carrying filters, user location, context size, and optional `["text","image"]` content types.
- `codex-rs/core/src/tools/hosted_spec_tests.rs:15-60` verifies hosted web-search option preservation and absence when disabled.
- `codex-rs/app-server/src/extensions.rs:60-80` installs `codex_web_search_extension` into the app-server extension registry.
- `codex-rs/ext/web-search/src/extension.rs:26-84` builds standalone web-search extension config: available only for OpenAI provider and non-disabled mode; settings include location, context size, allowed domains, `allowed_callers: Direct`, and `external_web_access` true only for live mode.
- `codex-rs/ext/web-search/src/extension.rs:109-126` contributes the standalone `WebSearchTool` when available; `codex-rs/ext/web-search/src/extension.rs:151-190` tests that enabling the extension contributes namespaced `web.run` with parallel support.
- `codex-rs/ext/web-search/src/tool.rs:36-70` defines standalone `web.run` as a namespaced direct tool using a schema generated from `SearchCommands`, the `web_run_description.md` prompt text, non-strict parameters, and parallel tool-call support.
- `codex-rs/ext/web-search/web_run_description.md:1-43` describes model-visible operations: search/image query, open/click/find/screenshot, finance/weather/sports/time, response length, usage hints, and browsing decision boundaries.
- `codex-rs/codex-api/src/search.rs:7-66` defines `SearchRequest`, `SearchInput`, and `SearchCommands`; `codex-rs/codex-api/src/search.rs:68-151` defines search/open/click/find/screenshot/finance/weather/sports/time command payloads and response length.
- `codex-rs/ext/web-search/src/history.rs:14-65` builds the standalone search conversation tail from visible user and assistant text only, keeping the previous user text, up to 1k assistant tokens, and current user text.
- `codex-rs/ext/web-search/src/tool.rs:81-123` handles standalone `web.run`: parse commands, derive a display action, construct `SearchClient`, build a request with session id/model/recent input/commands/settings/token budget, emit started/completed web-search turn items, call `alpha/search`, and return `SearchOutput`.
- `codex-rs/codex-api/src/endpoint/search.rs:16-38` posts typed search requests to `alpha/search`; `codex-rs/codex-api/src/endpoint/search.rs:180-233` verifies request JSON and response parsing.
- `codex-rs/ext/web-search/src/output.rs:7-33` marks standalone search output as external context, logs only `[standalone web search output]`, and projects plaintext output as a `FunctionCallOutput`.
- `codex-rs/ext/web-search/src/tool.rs:126-183` derives user-visible `WebSearchAction` from search/image query, literal open URL, and find-in-page commands; `codex-rs/ext/web-search/src/tool.rs:193-246` tests query/open/find action mapping.
- `codex-rs/protocol/src/models.rs:1058-1080` defines Responses API `web_search_call` response items with status and optional action; `codex-rs/protocol/src/models.rs:1498-1534` defines `WebSearchAction` variants: search, open page, find in page, and other.
- `codex-rs/core/src/event_mapping.rs:170-193` converts `ResponseItem::WebSearchCall` into `TurnItem::WebSearch`; `codex-rs/protocol/src/protocol.rs:1738-1750` maps started web-search turn items to legacy `WebSearchBegin`.
- `codex-rs/protocol/src/protocol.rs:2378-2389` defines `WebSearchBeginEvent` and `WebSearchEndEvent`; `codex-rs/app-server-protocol/src/protocol/thread_history.rs:633-650` upserts web-search thread items on begin/end.
- `codex-rs/tui/src/chatwidget/tool_lifecycle.rs:70-107` renders web-search begin/end into active/completed history cells; `codex-rs/tui/src/history_cell/search.rs:5-150` displays "Searching the web" / "Searched the web" and action detail.
- `codex-rs/core/src/context_manager/history.rs:350-372`, `:450-465`, and `:680-695` keep `WebSearchCall` in transcript/history, count it as model-generated, and avoid truncating it like function-output payloads.
- `codex-rs/core/src/stream_events_utils.rs:240-260` and `:520-535` treat `WebSearchCall` as external-context pollution; `codex-rs/core/src/stream_events_utils_tests.rs:45-60` verifies web search and tool search are external-context-polluting items.
- `codex-rs/app-server/src/request_processors/config_processor.rs:154-168` exposes provider capability `web_search` to clients through `ModelProviderCapabilitiesReadResponse`.

Runtime trace evidence:
- Claude WebFetch trace: model calls `WebFetch(url,prompt)` -> hostname permission rule is checked -> `getURLMarkdownContent` validates/upgrades URL, preflights domain, fetches with bounded redirects, converts/persists content, and caches it -> small preapproved markdown may return directly, otherwise `applyPromptToMarkdown` makes a nested Haiku call -> model receives only the `result` string while UI receives bytes/status/result details (`src/tools/WebFetchTool/WebFetchTool.ts:24-45`, `src/tools/WebFetchTool/WebFetchTool.ts:104-180`, `src/tools/WebFetchTool/utils.ts:337-530`, `src/tools/WebFetchTool/WebFetchTool.ts:208-306`, `src/tools/WebFetchTool/UI.tsx:9-71`).
- Claude WebSearch trace: model calls `WebSearch(query,filters?)` -> tool validates filters -> runtime calls `queryModelWithStreaming` with Anthropic hosted `web_search_20250305` as an extra server tool -> streamed server-tool blocks are parsed into progress and results -> model receives summarized search text plus JSON links and citation reminder (`src/tools/WebSearchTool/WebSearchTool.ts:76-84`, `src/tools/WebSearchTool/WebSearchTool.ts:235-400`, `src/tools/WebSearchTool/WebSearchTool.ts:401-434`).
- Codex hosted trace: config and model/provider capability allow web search -> `hosted_model_tool_specs` emits `ToolSpec::WebSearch` unless standalone `web.run` is active -> Responses API may return `web_search_call` items -> event mapping converts them into `TurnItem::WebSearch` for history/TUI, not into a local function-output item (`codex-rs/core/src/tools/spec_plan.rs:259-285`, `codex-rs/core/src/tools/hosted_spec.rs:20-43`, `codex-rs/protocol/src/models.rs:1058-1080`, `codex-rs/core/src/event_mapping.rs:170-193`).
- Codex standalone trace: extension contributes direct `web.run` -> model sends `SearchCommands` JSON -> runtime builds an `alpha/search` request with recent visible conversation input and web settings -> `SearchClient` posts to `alpha/search` -> plaintext output returns as a function-call output and a web-search turn item is emitted for display (`codex-rs/ext/web-search/src/tool.rs:36-70`, `codex-rs/ext/web-search/src/tool.rs:81-123`, `codex-rs/ext/web-search/src/history.rs:14-65`, `codex-rs/codex-api/src/endpoint/search.rs:16-38`, `codex-rs/ext/web-search/src/output.rs:7-33`).

What the model sees:
- Claude: `WebFetch` appears as a local tool taking URL and prompt; its prompt says fetched content may be summarized and cross-host redirects require a second explicit fetch (`src/tools/WebFetchTool/WebFetchTool.ts:24-45`, `src/tools/WebFetchTool/prompt.ts:1-21`).
- Claude: `WebSearch` appears as a local tool taking query plus optional allow/block domain filters, but internally delegates actual web search to an Anthropic server tool; model-visible result text explicitly reminds the assistant to cite markdown links (`src/tools/WebSearchTool/WebSearchTool.ts:25-37`, `src/tools/WebSearchTool/WebSearchTool.ts:401-434`).
- Codex hosted: the model sees a Responses hosted web-search tool spec when enabled; cached/live affects `external_web_access`, and text-and-image capable models can receive content type projection (`codex-rs/core/src/tools/hosted_spec.rs:20-43`).
- Codex standalone: the model sees namespaced `web.run` with commands for search, image search, opening/clicking/finding pages, PDF screenshots, finance/weather/sports/time, and response length (`codex-rs/ext/web-search/src/tool.rs:36-70`, `codex-rs/ext/web-search/web_run_description.md:1-43`, `codex-rs/codex-api/src/search.rs:27-66`).

What the runtime does:
- Claude: WebFetch owns HTTP fetching, redirect policy, HTML-to-markdown conversion, binary persistence, URL/domain cache, and a secondary model call for prompt-specific extraction (`src/tools/WebFetchTool/utils.ts:205-530`).
- Claude: WebSearch does not directly call a search HTTP endpoint from the tool body; it creates a nested model call with the provider's server-search tool schema and parses the streamed server-tool blocks (`src/tools/WebSearchTool/WebSearchTool.ts:254-400`).
- Codex: hosted web search is a provider-hosted tool spec in the main Responses request, while standalone `web.run` is an extension runtime that calls `alpha/search` through `SearchClient`; the tool planner prevents both from being exposed at once when standalone is available (`codex-rs/core/src/tools/spec_plan.rs:259-285`, `codex-rs/ext/web-search/src/tool.rs:81-123`).
- Codex: standalone search sends only a bounded visible conversation tail to the search endpoint: previous/current user text and truncated assistant text, not arbitrary system/developer/tool history (`codex-rs/ext/web-search/src/history.rs:14-65`).

What gets persisted:
- Claude: WebFetch caches URL content in an in-memory 15-minute/50 MB LRU cache and can persist binary content with a generated `webfetch-...` id; cache clearing imports `clearWebFetchCache` during session cache reset (`src/tools/WebFetchTool/utils.ts:50-83`, `src/tools/WebFetchTool/utils.ts:417-481`, `src/commands/clear/caches.ts:124-132`).
- Claude: WebSearch result text is a normal tool result; its `extractSearchText` returns empty so hidden result content does not create phantom transcript search matches (`src/tools/WebSearchTool/WebSearchTool.ts:223-234`).
- Codex: hosted `WebSearchCall` items are kept as response/history items and treated as model-generated transcript items (`codex-rs/core/src/context_manager/history.rs:350-372`, `:450-465`, `:680-695`).
- Codex: standalone search returns plaintext as a `FunctionCallOutput` and marks the output as containing external context; separate `WebSearch` turn items are emitted for display/history (`codex-rs/ext/web-search/src/output.rs:7-33`, `codex-rs/ext/web-search/src/tool.rs:112-123`, `codex-rs/app-server-protocol/src/protocol/thread_history.rs:633-650`).

What the user/TUI sees:
- Claude: WebFetch shows URL/prompt use, "Fetching..." progress, bytes and HTTP status, optional verbose result, and URL summary; WebSearch shows query/domain filters, "Searching" or "Found N results" progress, final search count and duration (`src/tools/WebFetchTool/UI.tsx:9-71`, `src/tools/WebSearchTool/UI.tsx:8-100`).
- Claude: WebFetch permission UI asks whether to allow fetching hostname content and can add a hostname allow rule (`src/components/permissions/WebFetchPermissionRequest/WebFetchPermissionRequest.tsx:29-220`).
- Codex: web search begin/end becomes active/completed history cells labeled "Searching the web" / "Searched the web" with query/open/find detail (`codex-rs/tui/src/chatwidget/tool_lifecycle.rs:70-107`, `codex-rs/tui/src/history_cell/search.rs:5-150`).
- Codex: app-server thread history upserts a `ThreadItem::WebSearch` on begin and end so clients see one evolving item instead of raw provider blocks (`codex-rs/app-server-protocol/src/protocol/thread_history.rs:633-650`, `codex-rs/app-server-protocol/src/protocol/v2/item.rs:352-365`, `:766-850`).

What debug/eval surfaces exist:
- Claude: usage accumulation tracks `server_tool_use.web_search_requests` and `server_tool_use.web_fetch_requests`; WebFetch logs domain-check failures, egress blocks, and URL-cache/domain-cache behavior through source-specific error/log paths (`src/services/api/claude.ts:2948-3011`, `src/tools/WebFetchTool/utils.ts:20-48`, `src/tools/WebFetchTool/utils.ts:176-203`).
- Claude: token/context analysis treats web-search and web-fetch result blocks as "other"; streaming message handling marks server/web search/fetch blocks as tool-input mode (`src/utils/contextAnalysis.ts:165-182`, `src/utils/messages.ts:3030-3042`).
- Codex: hosted web-search spec tests verify option preservation and disabled absence; standalone extension tests verify contribution of `web.run`; standalone output/action tests verify function-call output and action detail mapping; search endpoint tests verify posted JSON and response parsing (`codex-rs/core/src/tools/hosted_spec_tests.rs:15-60`, `codex-rs/ext/web-search/src/extension.rs:151-190`, `codex-rs/ext/web-search/src/output.rs:50-77`, `codex-rs/ext/web-search/src/tool.rs:193-246`, `codex-rs/codex-api/src/endpoint/search.rs:65-233`).
- Codex: web search is explicitly classified as external-context pollution, with tests covering that classification; provider capability `web_search` is exposed through app-server config capability reads (`codex-rs/core/src/stream_events_utils.rs:240-260`, `codex-rs/core/src/stream_events_utils_tests.rs:45-60`, `codex-rs/app-server/src/request_processors/config_processor.rs:154-168`).

Shared invariant:
- Web/network tools must be split by authority and surface: model-visible schema, permission/config gate, actual network executor or hosted provider tool, content/result transformation, model result projection, user display, persistence/cache behavior, and trace/eval markers. Do not collapse provider-hosted web search, local fetch, standalone search endpoint calls, and browser-like navigation into one generic "internet access" mechanism.

Claude-specific design:
- Claude has a local `WebFetch` tool with hostname-scoped permissions, domain preflight, strict redirect policy, HTML-to-markdown conversion, URL/content cache, binary persistence, and a secondary Haiku extraction call (`src/tools/WebFetchTool/WebFetchTool.ts:104-306`, `src/tools/WebFetchTool/utils.ts:130-530`).
- Claude has a local `WebSearch` wrapper that delegates to Anthropic's `web_search_20250305` server tool through a nested streaming model call rather than exposing a standalone search endpoint implementation (`src/tools/WebSearchTool/WebSearchTool.ts:76-84`, `src/tools/WebSearchTool/WebSearchTool.ts:254-400`).
- Claude's preapproved WebFetch host list is explicitly not a sandbox/network allowlist; it applies only to WebFetch GET behavior (`src/tools/WebFetchTool/preapproved.ts:1-12`).
- Claude's feature-gated `WebBrowserTool` reference is only a registration/import hook in this source snapshot; without an implementation file, it must remain an unknown/optional placeholder (`src/tools.ts:117-118`, `src/tools.ts:217`).

Codex-specific design:
- Codex has two mutually managed web-search paths: hosted Responses `ToolSpec::WebSearch` and standalone extension `web.run`; tool planning suppresses hosted web search when standalone web search is available (`codex-rs/core/src/tools/spec_plan.rs:259-285`).
- Codex standalone `web.run` is a namespaced direct extension tool, not a core shell/browser tool; it calls `alpha/search` with typed `SearchCommands`, settings, recent visible input, and token budget (`codex-rs/ext/web-search/src/tool.rs:36-123`, `codex-rs/codex-api/src/search.rs:7-66`).
- Codex `web_search` mode has `disabled/cached/live`; `cached` and `live` map to `external_web_access: false/true` for hosted and standalone paths (`codex-rs/protocol/src/config_types.rs:304-348`, `codex-rs/core/src/tools/hosted_spec.rs:20-43`, `codex-rs/ext/web-search/src/extension.rs:50-83`).
- Codex records web search as `WebSearchCall`/`TurnItem::WebSearch` and TUI history cells, while standalone result content is returned separately as function-call output (`codex-rs/protocol/src/models.rs:1058-1080`, `codex-rs/core/src/event_mapping.rs:170-193`, `codex-rs/ext/web-search/src/output.rs:7-33`).

Gaps or unknowns:
- Claude's optional `WebBrowserTool` is referenced but not implemented in the source files discovered for this mechanism, so no browser automation semantics should be inferred.
- General shell network sandboxing, managed network proxy policy, and network approval overlays are not covered here except where directly tied to web tools; those belong to mechanisms 19 and 24.
- Codex hosted web-search provider execution is represented by `ToolSpec::WebSearch` and provider response items; the provider-side implementation is not in the local Codex source.
- Browser/app-link flows in Codex TUI are separate app/client flows and are not treated as model-visible web/network tools in this card.

Distilled harness rule:
- Treat web/network access as multiple explicit surfaces, not a capability flag. For each surface, prove: the model-visible contract, who actually performs network access, what permissions/config gate it, how content is transformed, what is returned to the model, what appears in the UI/history, what is cached or persisted, and what tests/trace markers exist.

Anti-invention rule:
- Do not invent a general web browser tool unless a cited implementation exists. Do not infer that a fetch allowlist grants shell/network access. Do not merge Claude WebFetch, Claude WebSearch, Codex hosted web search, and Codex standalone `web.run`; if only one source has a mechanism, label it source-specific.

Verification pattern:
- For Claude WebFetch claims, verify schema/prompt, hostname permission rule, URL validation/preflight, redirect policy, fetch/markdown conversion, secondary model call, result projection, UI, cache clearing, and usage counters.
- For Claude WebSearch claims, verify schema/prompt, provider/model gating, hosted server-tool schema, nested streaming call, server-tool block parsing, model-result formatting, UI, and usage counters.
- For Codex hosted claims, verify config/mode resolution, provider capability/model metadata, `spec_plan` selection, `hosted_spec` mapping, `ResponseItem::WebSearchCall`, event mapping, history/TUI projection, and hosted-spec tests.
- For Codex standalone claims, verify extension installation/config, contributed namespaced tool spec, `SearchCommands` schema, recent-input construction, `SearchClient` `alpha/search` request, output projection, turn-item emission, app-server/TUI projection, and tests.

Skill text candidate:
- When studying web/network tools, first classify the surface: local fetch, hosted provider web search, standalone search endpoint, or browser/app flow.
- A web tool card must cite both the model-visible schema/prompt and the runtime network executor or hosted-tool spec. Schema alone does not prove runtime behavior.
- Keep permissions separate from network execution. A hostname rule, cached/live mode, provider capability, or feature flag is not interchangeable with shell sandbox network access.
- Separate outputs: raw fetched/search content, transformed markdown or search summaries, model-visible tool result, UI progress/history item, cached/persisted content, and debug/eval markers.
- If a provider performs the actual search, say so and cite the local hosted-tool spec plus returned response item shape; do not invent provider internals.
- If two tools both look like "web search", check the tool planner for mutual exclusion and source-specific feature gates before generalizing.
