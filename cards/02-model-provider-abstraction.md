# Mechanism 2: Model/provider abstraction

Mechanism: Model/provider abstraction

Status: complete

Claude source:
- Provider identity is an environment-derived union, not a broad plugin interface. `APIProvider` is `'firstParty' | 'bedrock' | 'vertex' | 'foundry'`, and `getAPIProvider()` selects by `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX`, `CLAUDE_CODE_USE_FOUNDRY`, else `firstParty` (`src/utils/model/providers.ts:4-14`).
- First-party base URL detection is explicit: unset `ANTHROPIC_BASE_URL` or `api.anthropic.com` is first-party; ant users also allow `api-staging.anthropic.com` (`src/utils/model/providers.ts:20-39`).
- Model IDs are provider-specific strings. `ModelConfig` is a `Record<APIProvider, ModelName>` (`src/utils/model/configs.ts:1-5`), concrete model configs carry separate `firstParty`, `bedrock`, `vertex`, and `foundry` strings (`src/utils/model/configs.ts:9-84`), and `ALL_MODEL_CONFIGS` registers the model map (`src/utils/model/configs.ts:86-99`).
- `ModelStrings` maps internal model keys to provider-specific model strings (`src/utils/model/modelStrings.ts:17-31`). Bedrock may fetch inference profiles and match canonical first-party substrings against the profile list, falling back to built-in Bedrock IDs (`src/utils/model/modelStrings.ts:33-55`).
- User model overrides are layered over provider-derived strings; overrides are keyed by canonical first-party model ID and can map to arbitrary provider-specific strings such as Bedrock inference profile ARNs (`src/utils/model/modelStrings.ts:57-76`). Reverse resolution maps override values back to canonical IDs for display/canonicalization (`src/utils/model/modelStrings.ts:78-100`).
- Main loop model selection is separate from provider selection. `getUserSpecifiedModelSetting()` reads session override, startup override, `ANTHROPIC_MODEL`, then settings, and checks `availableModels` allowlist (`src/utils/model/model.ts:49-78`). `getMainLoopModel()` returns the parsed user model or the built-in default (`src/utils/model/model.ts:80-98`).
- Runtime model selection can modify the selected main-loop model by context, e.g. plan-mode aliases (`src/utils/model/model.ts:140-167`). User aliases are resolved by `parseUserSpecifiedModel()`; it preserves custom model casing for custom model names and only strips/preserves `[1m]` as documented (`src/utils/model/model.ts:433-506`).
- API request model strings are normalized by removing `[1m]`/`[2m]` suffixes (`src/utils/model/model.ts:616-618`).
- Client construction returns an Anthropic-compatible SDK client regardless of selected provider. `getAnthropicClient()` builds shared headers/options, then branches to Bedrock, Foundry, Vertex, or first-party `Anthropic` constructors based on provider env flags (`src/services/api/client.ts:88-153`, `src/services/api/client.ts:153-190`, `src/services/api/client.ts:191-220`, `src/services/api/client.ts:221-298`, `src/services/api/client.ts:300-315`).
- The main query loop computes `currentModel` with `getRuntimeMainLoopModel()` and passes it as `options.model` to `deps.callModel` with `fallbackModel` and request context (`src/query.ts:570-579`, `src/query.ts:657-705`).
- The streaming API path calls `getAnthropicClient({ model: options.model, source: options.querySource })`, builds params from retry context, captures the API request, and calls `anthropic.beta.messages.create({ ...params, stream: true })` (`src/services/api/claude.ts:1776-1845`).
- Non-streaming fallback also creates a provider client with the selected model and normalizes the request model string before `anthropic.beta.messages.create` (`src/services/api/claude.ts:818-873`).
- Fallback is model-specific, not provider-specific. `withRetry` carries `model` and optional `fallbackModel` in `RetryOptions` (`src/services/api/withRetry.ts:120-142`), throws `FallbackTriggeredError(originalModel, fallbackModel)` when the configured 529 conditions and fallback model apply (`src/services/api/withRetry.ts:160-168`, `src/services/api/withRetry.ts:326-350`), and `query.ts` catches it, switches `currentModel`, updates `toolUseContext.options.mainLoopModel`, and emits a user-visible warning system message (`src/query.ts:893-950`).

Codex source:
- Provider metadata is a serializable registry, not only a string. `model-provider-info` documents two sources: built-in defaults and user-defined `model_providers` in config (`codex-rs/model-provider-info/src/lib.rs:1-7`).
- `WireApi` currently supports the Responses API and rejects removed `chat` wire config (`codex-rs/model-provider-info/src/lib.rs:50-80`).
- `ModelProviderInfo` contains display name, base URL, env-key auth, command auth, AWS auth, wire API, query params, static/env HTTP headers, request and stream retry/timeout settings, OpenAI-auth requirement, and websocket support (`codex-rs/model-provider-info/src/lib.rs:82-137`).
- `ModelProviderInfo::validate()` rejects invalid auth combinations such as AWS with env key/bearer/command/openai auth, and command auth with env key/bearer/openai auth (`codex-rs/model-provider-info/src/lib.rs:149-207`).
- `to_api_provider()` adapts provider metadata plus auth mode into the `codex_api::Provider` used for requests, including default base URL selection, headers, retry config, and stream idle timeout (`codex-rs/model-provider-info/src/lib.rs:237-273`). Provider retry/timeouts are normalized by `request_max_retries()`, `stream_max_retries()`, `stream_idle_timeout()`, and `websocket_connect_timeout()` (`codex-rs/model-provider-info/src/lib.rs:296-323`).
- Built-in providers include OpenAI, Amazon Bedrock, Ollama, and LM Studio (`codex-rs/model-provider-info/src/lib.rs:414-441`). User-defined providers are merged into built-ins; built-in Bedrock allows only `aws.profile` and `aws.region` override (`codex-rs/model-provider-info/src/lib.rs:443-478`).
- Config keeps both the chosen provider key and resolved provider info: `model_provider_id` is the key into `model_providers`, and `model_provider` is the info needed to make API requests (`codex-rs/core/src/config/mod.rs:631-635`). The resolved config merges built-ins with configured providers, chooses explicit CLI/provider config or defaults to `openai`, then clones the selected `ModelProviderInfo` (`codex-rs/core/src/config/mod.rs:3150-3172`). The final `Config` stores both values (`codex-rs/core/src/config/mod.rs:3524-3533`).
- Runtime provider abstraction is the `ModelProvider` trait. It exposes configured metadata, provider capability upper bounds, provider-specific preferred models for approval/memory tasks, attestation support, provider-scoped auth manager/auth/account state, API-provider adaptation, runtime base URL, API auth, and provider-specific model manager construction (`codex-rs/model-provider/src/provider.rs:23-43`, `codex-rs/model-provider/src/provider.rs:94-180`).
- Runtime provider instances are shared trait objects. `create_model_provider()` returns Amazon Bedrock’s provider implementation when `provider_info.is_amazon_bedrock()`, else a configured provider (`codex-rs/model-provider/src/provider.rs:182-197`).
- `SessionConfiguration` receives `config.model_provider.clone()` when starting a session (`codex-rs/core/src/session/mod.rs:607-609`). `Session::new()` creates a session-scoped `ModelClient` with that provider info and stable session-level arguments (`codex-rs/core/src/session/session.rs:1018-1029`).
- `ModelClient` is explicitly session-scoped and holds stable provider/auth/transport fallback state; per-turn model selection, reasoning controls, telemetry, and metadata are passed to streaming/unary methods (`codex-rs/core/src/client.rs:1-13`, `codex-rs/core/src/client.rs:167-185`, `codex-rs/core/src/client.rs:208-219`). `ModelClient::new()` creates the runtime provider and stores it in `ModelClientState` (`codex-rs/core/src/client.rs:366-408`).
- `ModelClientSession` is explicitly turn-scoped, lazily opens/reuses Responses WebSocket within a turn, stores last request and sticky routing token, and must be fresh per turn to avoid replaying old turn state (`codex-rs/core/src/client.rs:225-245`).
- `TurnContext` stores both `model_info` and a `SharedModelProvider` (`codex-rs/core/src/session/turn_context.rs:105-119`). `TurnContext::new` creates a provider for the turn from `ModelProviderInfo` and auth manager, sets telemetry with the selected model and slug, and stores provider/model fields in the context (`codex-rs/core/src/session/turn_context.rs:516-542`, `codex-rs/core/src/session/turn_context.rs:582-594`).
- The turn runtime consults provider info for stream retry count (`codex-rs/core/src/session/turn.rs:1048-1073`), builds inference trace using provider display name (`codex-rs/core/src/session/turn.rs:1874-1880`), and calls `client_session.stream()` with the prompt, `ModelInfo`, telemetry, reasoning settings, service tier, metadata, and inference trace (`codex-rs/core/src/session/turn.rs:1881-1895`).
- `ModelClientSession::stream()` takes per-turn settings explicitly and chooses transport based on provider `wire_api` and websocket health/support; for `WireApi::Responses`, it prefers websocket when enabled, otherwise falls back to HTTP Responses API (`codex-rs/core/src/client.rs:1610-1668`). Its HTTP/websocket instrumentation records provider/wire API/transport fields (`codex-rs/core/src/client.rs:1165-1177`, `codex-rs/core/src/client.rs:1246-1263`).

Runtime trace evidence:
- Claude: `getAPIProvider()` selects provider from env -> `getModelStrings()` maps canonical model keys to provider-specific IDs and applies overrides -> `getMainLoopModel()` / `parseUserSpecifiedModel()` select model -> `query.ts` computes `currentModel` -> `deps.callModel` passes `options.model` and `fallbackModel` -> `claude.ts` creates a provider-specific Anthropic-compatible client and calls Anthropic SDK message APIs. Evidence: `src/utils/model/providers.ts:4-14`, `src/utils/model/modelStrings.ts:17-31`, `src/utils/model/model.ts:80-98`, `src/query.ts:570-705`, `src/services/api/client.ts:88-315`, `src/services/api/claude.ts:1776-1845`.
- Codex: config merges provider catalog and resolves `model_provider_id` -> session configuration receives `ModelProviderInfo` -> `ModelClient::new()` creates a `SharedModelProvider` for session-scoped state -> `TurnContext` carries model info plus provider -> `run_turn()` passes per-turn model settings to `ModelClientSession::stream()` -> stream chooses Responses websocket or HTTP based on provider wire API/websocket support. Evidence: `codex-rs/core/src/config/mod.rs:3150-3172`, `codex-rs/core/src/session/mod.rs:607-609`, `codex-rs/core/src/client.rs:366-408`, `codex-rs/core/src/session/turn_context.rs:105-119`, `codex-rs/core/src/session/turn.rs:1881-1895`, `codex-rs/core/src/client.rs:1610-1668`.

What the model sees:
- Claude: the model sees the resolved API `model` in the Anthropic Messages request; `[1m]`/`[2m]` suffixes are runtime/session notation and are stripped before API request normalization (`src/utils/model/model.ts:616-618`, `src/services/api/claude.ts:862-873`, `src/services/api/claude.ts:1822-1831`).
- Codex: the model request receives per-turn `ModelInfo` via `ModelClientSession::stream()` (`codex-rs/core/src/client.rs:1619-1629`). Provider metadata influences transport/auth/base URL/headers/timeouts, not as a model-visible instruction block in this mechanism (`codex-rs/model-provider-info/src/lib.rs:237-273`, `codex-rs/core/src/client.rs:1004-1036`).

What the runtime does:
- Claude: computes provider from environment, maps model names per provider, constructs one of the Anthropic SDK-compatible clients, and passes the selected model through retry/streaming paths. Model fallback mutates `currentModel` and retries the same request path.
- Codex: resolves a provider registry from built-ins plus config, validates provider metadata, stores provider info in session config, creates shared runtime provider handles, keeps the model client session-scoped, creates a per-turn streaming session, and branches transport by wire API/websocket availability.

What gets persisted:
- Claude: no provider catalog persistence is shown in this mechanism. User model overrides come from settings (`src/utils/model/modelStrings.ts:57-76`), and model usage/fallback events are logged, but transcript persistence is not provider-specific in the cited source.
- Codex: `Config` stores `model_provider_id` and `model_provider`; thread persistence metadata stores the config model provider id when creating/resuming threads (`codex-rs/core/src/config/mod.rs:631-635`, `codex-rs/core/src/session/session.rs:521-560` from mechanism 1). Dynamic provider definitions live under config `model_providers` by source comment (`codex-rs/model-provider-info/src/lib.rs:1-7`).

What the user/TUI sees:
- Claude: user-facing model selection can use aliases such as `opus`, `sonnet`, `haiku`, `best`, and `opusplan`, resolved by `parseUserSpecifiedModel()` (`src/utils/model/model.ts:433-506`). On model fallback, the runtime yields a warning system message stating the switch from original to fallback model (`src/query.ts:943-948`).
- Codex: provider display name is part of `ModelProviderInfo` (`codex-rs/model-provider-info/src/lib.rs:82-90`) and is used in traces/inference trace context (`codex-rs/core/src/session/turn.rs:1876-1880`). This card did not trace the exact TUI model picker rendering; that belongs more naturally to TUI rendering or model-selection UI details, not the provider abstraction itself.

What debug/eval surfaces exist:
- Claude: API client creation logs debug information about custom headers/auth header presence (`src/services/api/client.ts:118-133`), streaming checkpoints bracket client creation/request send/response headers (`src/services/api/claude.ts:1776-1835`), and fallback logs analytics with original model, fallback model, and provider (`src/services/api/withRetry.ts:337-344`, `src/query.ts:931-941`).
- Codex: `ModelClientSession` stream/websocket methods are instrumented with provider, wire API, transport, model, and metadata-header fields (`codex-rs/core/src/client.rs:1165-1177`, `codex-rs/core/src/client.rs:1246-1263`). Inference trace context includes turn id, model slug, and provider name (`codex-rs/core/src/session/turn.rs:1876-1880`).

Shared invariant:
- Model/provider abstraction is a two-part boundary in both references: first resolve model identity and provider identity from user/config/runtime state, then pass an explicit resolved model plus provider-aware client/transport metadata into the sampling call. Both keep the selected model visible at the turn/request call site rather than hiding it inside message construction.

Claude-specific design:
- Provider selection is environment-flag driven and limited to first-party Anthropic, Bedrock, Vertex, and Foundry in the cited source.
- Provider differences are normalized behind Anthropic SDK-compatible clients, even for Bedrock/Foundry/Vertex casts back to `Anthropic`.
- Model IDs are provider-specific strings generated from `ALL_MODEL_CONFIGS`, with settings-based overrides keyed by canonical first-party IDs.
- Model fallback is surfaced as a model switch inside the query loop after `FallbackTriggeredError`.

Codex-specific design:
- Provider definitions are serializable config objects and can be built-in or user-defined in `model_providers`.
- Runtime provider behavior is a trait object (`ModelProvider`) with auth, account, capability, model-manager, and API-provider adaptation methods.
- `ModelClient` is session-scoped, while `ModelClientSession` is turn-scoped and stores sticky routing/WebSocket state.
- Transport selection is provider metadata driven through `WireApi` and websocket support/health.

Gaps or unknowns:
- Claude has no source-grounded generic `ModelProvider` trait in the cited mechanism; do not describe it as if it does.
- Codex model catalog/model selection (`ModelInfo`, presets, model manager internals) was only traced enough to prove the provider boundary; deeper model catalog behavior should be handled when a later mechanism requires it.
- Exact provider-specific auth details for Codex Amazon Bedrock and Claude Vertex/Foundry are only cited at the abstraction edge here, not exhaustively documented.
- Request payload construction is intentionally deferred to mechanism 5.

Distilled harness rule:
- Keep provider resolution, model resolution, runtime client construction, and per-turn sampling invocation as separate cited boundaries. The harness may normalize providers behind a common client shape (Claude) or expose provider trait objects and metadata (Codex), but the selected model and provider-derived request behavior must be explicit at the sampling call site.

Anti-invention rule:
- Do not introduce a provider plugin system unless the reference source has one. Claude’s cited mechanism is env/provider-string/client-branch based; Codex’s cited mechanism is registry/config/trait-object based. Treat those as different source-specific designs, not as interchangeable examples of the same abstraction.

Verification pattern:
- Find the provider identity type or provider metadata struct.
- Find how provider is selected from env/config.
- Find how model aliases/defaults map to API model IDs.
- Find the runtime client/provider object constructor.
- Find the sampling call and verify model/provider data is passed there.
- Find fallback or transport behavior only where it is directly tied to provider/model selection.
- Record whether the mechanism is general across both references or source-specific.

Skill text candidate:
- For model/provider abstraction, first cite the source-level provider identity, then cite model resolution, then cite runtime client construction, then cite the sampling call. Separate "model visible to the provider API" from "provider metadata used for auth/base URL/transport." If one reference uses a trait/registry and the other uses environment-selected SDK branches, preserve that difference instead of inventing a common provider framework.

