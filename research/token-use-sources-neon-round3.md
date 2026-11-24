# Cutting token use in agent threads: third round, primary sources

Raw research result, saved as returned on 2026-11-24. Researcher: gemini-4-neon. Not edited. This round read T3 Code's source and the documentation to close open questions from the first two rounds.

---

# Third-Round Research Report: Cutting Token & Subscription Usage in Claude Code + T3 Code

All T3 Code source links below are pinned to `pingdotgg/t3code` commit **`3380585e1bfcc99634ee78edf02624d396886fd2`** (2026-11-24). Every URL cited was opened directly in this session.

---

## 1. Part 1: Open Questions Closed

### 1.1 T3 Code and Delegated Child Threads

#### 1.1(a) Options and environment variables passed to the Claude Agent SDK (normal thread vs. delegated child thread)
**Label:** `source code`
- **URLs:**
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L1828-L1834`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L2523-L2739`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L2852-L2880`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L13701-L13750`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/provider/Drivers/ClaudeDriver.ts#L124-L165`
- **What T3 passes to `query({ prompt, options })` for BOTH normal and delegated child threads:**
  - **`model`**: `compiledSelection.apiModelId` (`ClaudeAdapterV2.ts#L2647`).
  - **`title`**: `turnInput.appThread.title` — skips Claude's own title-generation request (`// T3 names threads; a supplied title skips Claude's redundant naming request.`, `#L2648-L2649`).
  - **`tools`**: defaults to `{ type: "preset", preset: "claude_code" }` (`CLAUDE_CODE_PRESET_TOOLS`, `#L724-L727`, `#L2592`, `#L2650`, `#L13729`).
  - **`allowedTools` / `disallowedTools`**: `allowedTools` adds `mcp__t3-code__*` (or `CLAUDE_READ_ONLY_T3_MCP_ALLOWED_TOOLS` in read-only sandboxes) (`#L2864-L2868`); `disallowedTools` is forwarded if set (`#L2666`).
  - **`systemPrompt`**: `{ type: "preset", preset: "claude_code", append: ... }` combining `buildRuntimeInstructions(...)` + (when MCP is attached) `T3_CODE_ORCHESTRATION_INSTRUCTIONS` + `CLAUDE_BACKGROUND_WAIT_INSTRUCTIONS` + `CLAUDE_SEND_TO_USER_INSTRUCTIONS` + (when device capability is active) `T3_CODE_DEVICE_TOOL_INSTRUCTIONS_WITH_TOOL_SEARCH` + any `--append-system-prompt-file` / `--append-system-prompt` from instance launch args (`#L2695-L2714`).
  - **`settingSources`**: **NOT passed to `query()` at all** (`settingSources` only appears at `#L6057` and `#L6090` inside T3's own pre-flight `resolveClaudeSettings(...)` calls). Thus the Claude Code CLI uses its normal default setting sources from the instance's `CLAUDE_CONFIG_DIR` (or `~/.claude`) plus project/local settings.
  - **`plugins`**: populated only from any `--plugin-dir` entries parsed out of instance `launchArgs` (`#L2585-L2588`, `#L2725-L2727`). Installed user/project plugins still load via Claude Code's normal settings.
  - **`mcpServers`**: attaches `{ "t3-code": { type: "http", url: session.endpoint, headers: { Authorization: ... }, timeout: ... } }` (`#L2869-L2878`, `#L2694`).
  - **`effort` & `thinking`**: passes `effort: compiledSelection.effort` (`#L2658-L2662`); unless `alwaysThinkingEnabled === false` or `--thinking-display omitted`, passes `thinking: { type: "adaptive", display: "summarized" }`, sets `extraArgs["thinking-display"] = "summarized"`, and injects `showThinkingSummaries: true` into `settings` (`#L2579-L2584`, `#L2671-L2681`).
  - **`settings` & `settings.env`**: merges `sdkSettings` (from `--settings` launch arg), `compiledSelection.settings`, `autoCompactWindow: Number(autoCompactWindow)`, and `settings.env` containing `CLAUDE_CODE_AUTO_COMPACT_WINDOW` (when set in provider settings), `ENABLE_TOOL_SEARCH: "true"` (when `claudeToolSearchOverride` detects `auto` or `auto:1..99` so T3's MCP tools stay deferred, `#L2409-L2418`, `#L2639-L2641`), and any pinned alias override (`ANTHROPIC_DEFAULT_<ALIAS>_MODEL`, `#L2616-L2622`).
  - **`hooks`**: registers `SubagentStart` (to inject device instructions into native subagents when `deviceToolsAvailable`) and `Stop` (to stop goal loops while T3 background wakes are pending) (`#L2715-L2724`).
  - **`env`**: instance process environment (including `CLAUDE_CONFIG_DIR` from `homePath` and per-instance custom env vars merged in `ClaudeDriver.ts#L124-L162`), passed through `claudeQueryEnvironment(...)` (`#L13721-L13728`). On resumed sessions, sets `MCP_CONNECTION_NONBLOCKING: "0"` unless already set (`#L2396`, `#L2403`).
- **Exact differences for a delegated child thread vs. a normal thread:**
  Only **two** SDK options differ in `ClaudeAdapterV2.ts`:
  1. `promptSuggestions` is forced to `false` (`#L13715-L13717`):
     ```ts
     // Delegated threads have no one typing, so they never ask for suggestions.
     promptSuggestions:
       !isDelegatedAppThread(turnInput.appThread) && (yield* promptSuggestionsEnabled),
     ```
  2. `env.CLAUDE_CODE_PROMPT_CACHE_TTL` may be set from `subagentPromptCacheTtl` (`#L13721-L13728`, detailed in 1.1(b)).
  Everything else — `systemPrompt`, `tools`, `mcpServers` (`t3-code`), `plugins`, `autoCompactWindow`, `thinking`, and inherited Claude settings/skills/hooks — is **identical** to a normal thread on the same provider instance.

#### 1.1(b) Cache lifetime for delegated child threads & whether T3 copies `subagentPromptCacheTtl` into `CLAUDE_CODE_PROMPT_CACHE_TTL`
**Label:** `source code`
- **URLs:**
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L2380-L2405`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L6078-L6102`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L13721-L13728`
- **Exact quote (`ClaudeAdapterV2.ts#L2380-L2405`):**
  ```ts
  // A delegated child is its own SDK session, so Claude Code gives it the main-conversation cache
  // TTL. When the user sets Claude's subagentPromptCacheTtl, the child follows it like a native
  // subagent would. A resumed session's first request otherwise goes out before its MCP servers
  // connect, and Claude Code tells the agent they disconnected. Explicit instance values still win.
  export function claudeQueryEnvironment(
    environment: NodeJS.ProcessEnv,
    appThread: Pick<OrchestrationV2AppThread, "lineage">,
    subagentPromptCacheTtl: "5m" | "1h" | undefined,
    resume = false,
  ): NodeJS.ProcessEnv {
    const promptCacheTtl =
      appThread.lineage.relationshipToParent === "subagent" &&
      subagentPromptCacheTtl !== undefined &&
      environment.CLAUDE_CODE_PROMPT_CACHE_TTL === undefined
        ? subagentPromptCacheTtl
        : undefined;
  ```
- **Critical finding (corrects a common assumption):**
  - Because each T3 `delegate_task` child is its own Agent SDK session, Claude Code treats it as a **main conversation** — which on a Claude subscription defaults to a **1-hour cache TTL (2x cache-write cost)**, NOT 5 minutes.
  - T3 **does** copy `subagentPromptCacheTtl` (or `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`, resolved in `resolveSubagentPromptCacheTtl`, `#L6080-L6102`) into `CLAUDE_CODE_PROMPT_CACHE_TTL` for delegated children (`lineage.relationshipToParent === "subagent"`), **only when the user explicitly sets it** (and only if the instance env doesn't already define `CLAUDE_CODE_PROMPT_CACHE_TTL`).
  - If `subagentPromptCacheTtl` is left **unset**, `resolveSubagentPromptCacheTtl` returns `undefined`, T3 leaves `CLAUDE_CODE_PROMPT_CACHE_TTL` unset, and every delegated child silently pays the **1-hour (2x)** cache write rate instead of **5m (1.25x)**. Explicitly setting `"subagentPromptCacheTtl": "5m"` in `~/.claude/settings.json` fixes this for all T3 delegated children.

#### 1.1(c) Whether "Auto-compact after" and "Compact idle threads" apply to delegated child threads, and when "Before the cache expires" fires
**Label:** `source code` + `documented`
- **URLs:**
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L2603-L2638`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/IdleCompaction.ts#L9-L115`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/IdleCompactionService.ts#L22-L124`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/packages/contracts/src/orchestrationV2.ts#L202-L205`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/docs/user/providers-claude.md#L51-L82`
- **Answers:**
  1. **"Auto-compact after" (`autoCompactWindow`)**: **YES, applies to delegated child threads.** `makeClaudeQueryOptions` sets both `settings.autoCompactWindow` and `settings.env.CLAUDE_CODE_AUTO_COMPACT_WINDOW` unconditionally for all threads on that Claude instance (`ClaudeAdapterV2.ts#L2603-L2638`).
  2. **"Compact idle threads" (`Before the cache expires (beta)`)**: **NO, explicitly skipped for delegated child threads** (and side chats and workflow step threads). `IdleCompaction.ts#L53` checks:
     ```ts
     if (!isListedThreadLineage(thread.lineage)) return null;
     ```
     And `packages/contracts/src/orchestrationV2.ts#L202-L205` defines:
     ```ts
     export const isListedThreadLineage = (lineage: OrchestrationV2AppThreadLineage): boolean =>
       lineage.relationshipToParent !== "subagent" &&
       lineage.sideChat !== true &&
       lineage.workflowStep !== true;
     ```
     *(Note: `onUserDialog` for Claude Code's `resume_return` dialog in `ClaudeAdapterV2.ts#L13290-L13307` still reads `resolveIdleCompaction` on any resumed session, returning `"continue"` if set to `"off"`.)*
  3. **Exact firing time of "Before the cache expires (beta)"**:
     - `IdleCompaction.ts#L9-L12`:
       ```ts
       const MIN_CONTEXT_TOKENS = COMPACTION_SUGGESTION_MIN_TOKENS; // 200_000
       const LEAD_MS = 2 * 60_000;
       ```
     - `IdleCompaction.ts#L102-L115`: requires `usedTokens >= 200_000` and a finite `promptCacheExpiresAt` (`endMs`), then schedules `/compact` at **`startMs = endMs - LEAD_MS` — exactly 2 minutes before `promptCacheExpiresAt`** (i.e., ~58 minutes idle on a 1h subscription cache, ~3 minutes idle on a 5m cache). Timers are in-memory only (`IdleCompactionService.ts#L25-L26`); if missed or after server restart it falls back to **When you return**.

#### 1.1(d) Running delegated children with a leaner configuration (second Claude instance, `CLAUDE_CONFIG_DIR`, env vars, launch args, `delegate_task` targeting)
**Label:** `source code` + `documented`
- **URLs:**
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/docs/user/providers-claude.md#L7-L45`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/packages/contracts/src/settings.ts#L803-L831`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/mcp/OrchestratorMcpService.ts#L1678-L1835`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/provider/Drivers/ClaudeDriver.ts#L124-L162`
- **Answer: YES, completely supported.**
  - `docs/user/providers-claude.md#L7-L10`: *"Use a separate Claude config directory for each account. This also works for named presets that need different Claude settings or a router connection."*
  - Each Claude instance in **Settings > Providers** has its own:
    - `CLAUDE_CONFIG_DIR path` (`homePath` in `settings.ts#L816-L824` — *"Keeps .claude.json and .claude separate"*, leaving `HOME` and Keychain intact so you can sign in with the same subscription account via `CLAUDE_CONFIG_DIR=~/.claude_lean claude auth login`).
    - **Environment variables** (e.g., `CLAUDE_CODE_PROMPT_CACHE_TTL=5m`, `ENABLE_CLAUDEAI_MCP_SERVERS=false`, `CLAUDE_CODE_DISABLE_AGENT_PROGRESS_SUMMARIES=1`).
    - **Launch arguments** (`launchArgs`, e.g., `--settings ~/.claude_lean/settings.json`, `--strict-mcp-config`, `--disallowedTools ...`, `--disable-slash-commands`).
  - `delegate_task` directly targets that instance via `target.providerInstanceId` (`OrchestratorMcpService.ts#L1714-L1835`):
    ```ts
    const requestedInstanceId = input.target?.providerInstanceId;
    ...
    const provider = input.providers.find((candidate) => candidate.instanceId === instanceId);
    ...
    modelSelection: { instanceId, model, ...(options === undefined ? {} : { options }) },
    ```
  - *(Note from `docs/user/providers-claude.md#L35-L38`: existing threads can only switch between Claude instances with the same config directory, which doesn't affect fresh delegated child threads.)*

#### 1.1(e) What T3 adds to the system prompt and tool list, how large it is, and whether any of it can be turned off
**Label:** `source code`
- **URLs:**
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L2497-L2512`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L2695-L2714`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/packages/provider-core/src/server/orchestrationInstructions.ts#L1-L146`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/packages/provider-core/src/server/runtimeInstructions.ts#L1-L107`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/mcp/McpHttpServer.ts#L293-L304` and `#L1391-L1575`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/mcp/McpToolExposure.ts#L7-L12`
- **Appended system prompt instructions (`ClaudeAdapterV2.ts#L2698-L2714`):**
  - `buildRuntimeInstructions(...)` (`runtimeInstructions.ts`, 6,732 chars total file; outputs ~400–1,200 chars depending on workspace/feedback).
  - `T3_CODE_ORCHESTRATION_INSTRUCTIONS` (`orchestrationInstructions.ts#L4-L30`): **4,063 chars (~1,000–1,100 tokens)**.
  - `CLAUDE_BACKGROUND_WAIT_INSTRUCTIONS` (`ClaudeAdapterV2.ts#L2499-L2504`): **115 chars (~30 tokens)**.
  - `CLAUDE_SEND_TO_USER_INSTRUCTIONS` (`ClaudeAdapterV2.ts#L2508-L2512`): **280 chars (~70 tokens)**.
  - `T3_CODE_DEVICE_TOOL_INSTRUCTIONS_WITH_TOOL_SEARCH` (`orchestrationInstructions.ts`): conditionally appended only when `deviceToolsAvailable === true`.
  - **Total appended system prompt:** roughly **1,200–1,500 tokens** (plus MCP server instructions).
- **T3's own MCP tool list (`apps/server/src/mcp/toolkits/*/tools.ts`):**
  - **92 total tools** across 15 populated toolkits: `orchestrator` (20), `pullRequests` (18), `preview` (12), `issues` (8), `thread` (8), `project` (7), `device` (4), `attachment` (3), `worktree` (3), `environment` (2), `previewControls` (2), `theme` (2), `feedback` (1), `html` (1 + `html_render`), `workflow` (1).
  - **How Claude loads them:** Claude Code defers MCP tools behind `ToolSearch` (`ClaudeAdapterV2.ts#L2497`, `#L2409-L2418` even overrides `ENABLE_TOOL_SEARCH=auto` to `"true"` so deferred loading stays active). So Claude gets only the **deferred tool names** in its initial prompt (a big chunk of the ~7,300 deferred-tool tokens in your profile, alongside any other MCP servers), not all 92 full JSON schemas. *(Non-Claude clients that send `MCP_TOOL_EXPOSURE_HEADER: deferred` use T3's server-side `McpToolExposure.ts#L7-L12`, which exposes only 4 tools initially: `orchestrator_capabilities`, `delegate_task`, `task_status`, `tool_search`.)*
- **Can any of it be turned off?**
  - **Settings-gated toolkits (`McpHttpServer.ts#L294-L304`):** `workflowsEnabled` (Labs → Workflows, off by default), `linkedEnvironmentsEnabled` (Labs → Linked environments), `agentFeedback.mode` (off by default), plus forge/issue/device availability (`forgeOfThread`, `deviceToolsAvailable`).
  - **Turning off T3's MCP tools or appended instructions entirely in T3 settings:** **No toggle exists in `ClaudeSettings`** — `t3-code` MCP and `T3_CODE_ORCHESTRATION_INSTRUCTIONS` are always attached when `mcpSession` exists (`ClaudeAdapterV2.ts#L2704-L2711`, `#L2860-L2879`). However, on a **lean delegated-child Claude instance**, passing `--disallowedTools "mcp__t3-code__*"` (or selective wildcards like `"mcp__t3-code__preview_*"`, `"mcp__t3-code__pull_request_*"`, `"mcp__t3-code__t3_project_*"`) in **Launch arguments** strips those tool names from Claude's context (verified in Claude Code `permissions.md#L204`).

#### 1.1(f) T3 Code docs, changelog, or issues on token use, usage limits, auto-resume, or usage view
**Label:** `documented` + `source code`
- **URLs:**
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/docs/user/usage.md#L1-L210`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/docs/user/thread-sidebar.md#L266-L288`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/docs/user/providers-claude.md#L84-L91`
  - `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/apps/server/src/orchestration-v2/UsageLimitRecoveryWorker.ts#L15-L155`
- **Findings:**
  - **Usage view (`docs/user/usage.md`):** Open via `mod+u`. Aggregates tokens, cache savings, cache hit rate, model breakdowns, and estimated API-equivalent cost across Claude, Codex, Grok, Antigravity, Cursor, OpenCode, Pi, and Muse (92-day default retention via `usageHistoryRetentionDays`; Claude deletes its own transcripts after 30 days). Text generation (titles, commits, PR text, review guides) is excluded on Claude Code 2.0.63+.
  - **Usage → Limits & `/usage-limits` (`docs/user/usage.md#L118-L159`):** Pools subscription accounts across connected environments and CLIProxyAPI hubs; `/usage-limits` inspects the current snapshot without running the agent; thread details card can display live **Usage limits** and **Context window** (with an auto-compact tick mark). Shows banked reset credits and **Use reset** *(except macOS Claude servers using Keychain, `usage.md#L131-L132`)*.
  - **Auto-resume & Snooze on usage limits (`docs/user/thread-sidebar.md#L275-L287` & `UsageLimitRecoveryWorker.ts#L17-L79`):** Built-in **Settings → Threads → Resume after usage limits** (`autoResumeLimitedThreads`) and **Snooze until limits reset** (`snoozeLimitedThreads`). When a thread fails with `lastErrorClass === "usage_limit"` and a future `usageLimitResetAt`, `UsageLimitRecoveryWorker` arms recovery and dispatches `"Continue where you left off."` once the reset passes (up to `MAX_OVERDUE_ARM_MS = 24h`). Note: on Claude Code mid-turn holds (`providers-claude.md#L86-L90`), Claude Code itself may hold the turn open until reset.

---

### 1.2 Trimming the 70,000-Token Start (Claude Code Documentation)

#### 1.2(a) `skillOverrides`
**Label:** `documented`
- **URLs:**
  - `https://code.claude.com/docs/en/settings-reference.md` (lines 3845–3871)
  - `https://code.claude.com/docs/en/skills.md` (lines 944–976)
- **Allowed values (exact quotes from `settings-reference.md`):**
  - `"on"`: *"Claude sees the skill and you can type `/name`"* (default when unset)
  - `"name-only"`: *"Claude sees the skill by name without its description"*
  - `"user-invocable-only"`: *"Claude doesn't see the skill, but you can still type `/name`"* (labeled `user-only` in `/skills`)
  - `"off"`: *"Claude doesn't see the skill and `/name` is hidden from autocomplete"* (also hides from Remote Control & Agent SDK command lists; invoking by full name returns an error)
- **Scope:** Any settings file (user, project, local, managed, `--settings`). The `/skills` UI writes to `.claude/settings.local.json`.
- **Does it cover plugin skills? NO.** Exact quote (`settings-reference.md` L3868 & `skills.md` L976):
  > *"Overrides don't apply to plugin skills, which you manage through `/plugin`."*
- **How to hide one plugin's skills in one project only:** You cannot hide individual skills of an enabled plugin via `skillOverrides`; instead, disable the whole plugin for that project via `"enabledPlugins": { "<plugin-name>@<origin>": false }` in `.claude/settings.json` (shared) or `.claude/settings.local.json` (personal/gitignored).

#### 1.2(b) `syncClaudeAiPlugins`, `syncClaudeAiSkills`, `disableClaudeAiConnectors`
**Label:** `documented`
- **URLs:**
  - `https://code.claude.com/docs/en/settings-reference.md` (lines 3872–3909, 4568–4586)
  - `https://code.claude.com/docs/en/mcp.md` (lines 1495–1533)
- **Exact specifications:**

| Key | Allowed Values | Default | Settings Files (Scope) | What You Lose When Turned Off |
| :--- | :--- | :--- | :--- | :--- |
| `syncClaudeAiPlugins` | `false`, `true` (only `false` is honored; `true` = unset) | unset (syncs on claude.ai login) | User, local, managed, `--settings` (**not** project `.claude/settings.json`) | Stops downloading plugins enabled on your claude.ai account into `~/.claude/plugins/synced/` and stops loading `<name>@synced`. In user/managed settings, moves existing synced plugins to `~/.claude/plugins/.trash/`. *(Requires v2.1.273+; changelog v2.1.275)* |
| `syncClaudeAiSkills` | `false`, `true` (only `false` is honored; `true` = unset) | unset (syncs on claude.ai login) | User, local, managed, `--settings` (**not** project `.claude/settings.json`) | Stops downloading custom skills from your claude.ai account into `~/.claude/skills/synced/` and stops loading them. In user/managed settings, moves them to `~/.claude/skills/.trash/`. |
| `disableClaudeAiConnectors` | `true`, `false` (`false` = unset; **any-source-true** semantics) | `false` | **Any file** (user, project, local, managed, `--settings`). Per-session env: `ENABLE_CLAUDEAI_MCP_SERVERS=false` | Claude Code neither fetches nor connects remote claude.ai MCP connectors (e.g., Slack, Google Drive, Linear connected at `claude.ai/customize/connectors`). Does **not** affect `--mcp-config` servers, nor in-process Desktop/cloud-delivered connectors (`mcp.md` L1500–1503). |

#### 1.2(c) Disabling a plugin for one project (not globally) and the reverse
**Label:** `documented`
- **URLs:**
  - `https://code.claude.com/docs/en/settings-reference.md` (lines 4248–4276)
  - `https://code.claude.com/docs/en/plugins/loading.md` (lines 1–120)
- **Exact precedence:** `managed` > `local` (`.claude/settings.local.json`) > `project` (`.claude/settings.json`) > `user` (`~/.claude/settings.json`).
  - **Disable in one project only (keep enabled globally):** keep `"<name>@<origin>": true` in `~/.claude/settings.json` and set `"<name>@<origin>": false` in the project's `.claude/settings.json` (team-wide) or `.claude/settings.local.json` (your machine only).
  - **Enable in one project only (disabled globally):** set `"<name>@<origin>": false` in `~/.claude/settings.json` and `"<name>@<origin>": true` in that project's `.claude/settings.json` or `.claude/settings.local.json`.
  - **Gotcha (quoted from `settings-reference.md` L4275):** *"Project settings take precedence over user settings, so setting a plugin to `false` in `~/.claude/settings.json` doesn't disable a plugin that the project's `.claude/settings.json` enables. To opt out of a project-enabled plugin on your machine, set it to `false` in `.claude/settings.local.json` instead. Plugins force-enabled by managed settings can't be disabled this way."*
  - Applies to all 4 plugin origins (`@<marketplace>`, `@inline`, `@skills-dir`, `@synced`).

#### 1.2(d) Removing built-in tools from context (`permissions.deny`, `disallowedTools`, `--tools`, largest tool definitions)
**Label:** `documented` (largest definitions: `not found`)
- **URLs:**
  - `https://code.claude.com/docs/en/permissions.md` (lines 60–62, 105, 204)
  - `https://code.claude.com/docs/en/cli-reference.md` (lines 88, 140)
  - `https://code.claude.com/docs/en/tools-reference.md` (lines 80–125, 298)
- **Exact quotes:**
  - **`permissions.deny` with bare tool names (`permissions.md` L60, L105, L204):**
    > *"A bare tool name like `Bash` removes the tool from Claude's context entirely, so Claude never sees it."*
    > *"`Bash(*)` is equivalent to `Bash` and matches all Bash commands. As a deny rule, both forms remove the tool from Claude's context."*
    > *"A tool matched by a bare-name glob deny rule is removed from Claude's context, the same as a bare tool name... `"*"` matches every tool, and `"mcp__*"` matches every MCP tool across all servers."*
  - **`--disallowedTools` / `--disallowed-tools` (`cli-reference.md` L88):**
    > *"A bare tool name removes the matching tools from Claude's context: `"Edit"` removes Edit, `"*"` removes every tool, and `"mcp__*"` removes every MCP tool. A scoped rule such as `Bash(rm *)` leaves the tool available and denies only calls that match as written."*
  - **`--tools` (`cli-reference.md` L140):**
    > *"Restrict which built-in tools Claude can use. Use `""` to disable all, `"default"` for the default set, or tool names like `"Bash,Edit,Read"`. On macOS, Linux, and WSL, the default set leaves out `Glob` and `Grep`... The flag doesn't affect MCP tools; to deny those too, use `--disallowedTools "mcp__*"`."*
  - **Exception:** `EndConversation` cannot be removed by `permissions.deny`, `--disallowedTools`, or `--tools` while any other tool remains (`tools-reference.md` L298).
  - **Which built-in tools have the largest definitions:** `not found` — neither `tools-reference.md`, `context-window.md`, nor `costs.md` publishes per-tool token sizes for built-in tools (users inspect their own session with `/context all`).

#### 1.2(e) Stopping a plugin's `SessionStart` hook output from entering context without disabling the plugin
**Label:** `documented` (individual hook disable: **not possible**)
- **URLs:**
  - `https://code.claude.com/docs/en/hooks.md` (lines 247, 257–266, 735–738, 1285–1318)
  - `https://code.claude.com/docs/en/settings-reference.md` (lines 3613–3670)
- **Exact quote (`hooks.md` L735):**
  > *"To temporarily disable all hooks without removing them, set `"disableAllHooks": true` in your settings file... **There is no way to disable an individual hook while keeping it in the configuration.**"*
- **What IS possible:**
  - `"disableAllHooks": true` (in user, project, local, or `--settings`) turns off **all** user, project, local, and plugin hooks (plus custom status line and `fileSuggestion`), while keeping managed hooks and Agent SDK hooks running (`settings-reference.md` L3663).
  - `"allowManagedHooksOnly": true` (managed settings only) blocks all non-managed plugin hooks unless the plugin is force-enabled in managed `enabledPlugins` (`settings-reference.md` L3634–3636).
  - `--bare` (headless CLI mode) skips all project/plugin hooks, skills, and plugins (`permissions.md` L842).
  - There is **no setting** to mute a single plugin's `SessionStart` hook while leaving that plugin enabled and other hooks active.

#### 1.2(f) Diagnostic commands: what each reports & version needed
**Label:** `documented` + `source code` (CHANGELOG)
- **URLs:**
  - `https://code.claude.com/docs/en/commands.md` (lines 73, 85, 147, 167)
  - `https://code.claude.com/docs/en/debug-your-config.md` (lines 15–48)
  - `https://code.claude.com/docs/en/costs.md` (lines 58–73, 184–223)
  - `https://code.claude.com/docs/en/plugin-evals.md` (lines 9–33, 97–112)
  - `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md`
- **Summary:**
  - **`/context` (and `/context all`)**: Visualizes context window usage as a colored grid broken down by system prompt, system tools, MCP tools, custom subagents, memory files, skills (post-budget since v2.1.196), and messages, with optimization warnings. **Version:** Added in **v1.0.86** (`CHANGELOG.md` L11213; grouped visualization v2.0.74; accurate post-budget skills size v2.1.196).
  - **`/usage`** (aliases `/cost`, `/stats`): Shows session token usage & list-price cost estimate, `Prompt cache (main)` hit rate/misses/rebuilds/TTL (**v2.1.251+**, likely-cause text **v2.1.260+**), plan usage limits, per-skill/subagent/plugin/MCP attribution & actionable fix recommendations (`f` to apply, **v2.1.305+**), and heavy `/loop` rows (**v2.1.242+**). **Version:** Present since **v2.0.47** or earlier (`CHANGELOG.md` L10835); key modern blocks need **v2.1.242–v2.1.305+**.
  - **`/doctor`** (alias `/checkup`, plus `claude doctor` CLI and `/doctor prompt-audit [path]`): Diagnoses installation, settings, extension, skill-listing context cost, and `CLAUDE.md` issues and proposes interactive fixes; `prompt-audit` audits instructions/skills for older-model prompting anti-patterns. **Version:** Base `/doctor` since **v2.0.12** (`CHANGELOG.md` L11037); `prompt-audit` requires **v2.1.283+** (`commands.md` L85, `CHANGELOG.md` L4168).
  - **`/skill-doctor`**: Shows what each loaded skill costs in context and how often it is invoked so you can prune unused skills. **Version:** Requires **v2.1.252+** and feature-flag fetching (`commands.md` L147; announced in `CHANGELOG.md` at **v2.1.261**, L5449).
  - **`claude plugin eval`**: Shell test-runner that runs eval cases **with** and **without** a plugin (no-plugin baseline `WITH` / `W/OUT` / `Δ`), grades outcomes, and outputs JSON + self-contained HTML reports (`--max-cost-usd`, `--ablation none`, `--disallow-tools`). **Version:** Requires **v2.1.269+** (`plugin-evals.md` L29, `CHANGELOG.md` L5129; git >= 2.31 enforced from **v2.1.283+**). *(Bonus: `claude plugin details <name>` added in **v2.1.139** reports always-on vs. on-invoke token cost per plugin component, `plugins/measure.md`.)*

---

### 1.3 Effort: Anthropic's Published Measurements & Subagent/Inherited Effort Rules

**Label:** `measured` + `documented`
- **URLs:**
  - `https://claude.dev/blog/spending-your-effort/` (Sep 25, 2026)
  - `https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence.md` (lines 426–484)
  - `https://claude.dev/blog/which-claude-model-should-you-build-with.md` (lines 71–101)
  - `https://claude.dev/blog/what-a-task-costs-on-opus-5-5.md`
  - `https://code.claude.com/docs/en/model-config.md` (lines 649–769)
  - `https://code.claude.com/docs/en/sub-agents.md` (lines 430–442)
  - `https://platform.claude.com/docs/en/build-with-claude/effort.md`

#### Published numbers on Opus 5.5 and Fable 5.2 (and Fable 5.1)
1. **Official benchmark table (`optimizing-for-cost-and-intelligence.md` L432–L440, L459–L460):**

   | Benchmark | Model | Effort | Score | Cost / task |
   | :--- | :--- | :--- | :--- | :--- |
   | SWE-bench Pro subset | **Claude Opus 5.5** | `low` | 87.4% solved | **$0.11** |
   | SWE-bench Pro subset | **Claude Opus 5.5** | `medium` (default) | 92.8% solved | **$0.21** |
   | SWE-bench Pro subset | **Claude Opus 5.5** | `high` | 95.3% solved | **$0.29** |
   | SWE-bench Pro subset | **Claude Opus 5.5** | `xhigh` | 96.7% solved (1 run) | **$0.74** |
   | DeepResearch Bench II | **Claude Fable 5.2** | `low` | 54.5% rubric | **$1.49** |
   | DeepResearch Bench II | **Claude Fable 5.2** | `medium` | 55.0% rubric | **$2.16** |
   | DeepResearch Bench II | **Claude Fable 5.2** | `high` (API default) | 57.7% rubric | **$4.82** |

   - **Key ratios reported by Anthropic:**
     - On SWE-bench Pro, Opus 5.5 `medium` costs **~70% of `high`** ($0.21 vs. $0.29) for ~2.5 points lower score; `xhigh` jumps **2.55× over `high`** ($0.74 vs. $0.29) for +1.4 points.
     - **Re-run policy (`optimizing-for-cost-and-intelligence.md` L477–L481):** Running Opus 5.5 at `low` and re-running only the 13% failures at `high` achieved **~97% pass rate for ~$0.17/task** — beating everything-at-`high` (95.3% for $0.29, **~41% cheaper**).
     - On DeepResearch Bench II, Fable 5.2 `high` scored +3.2–3.3 pts over `low` for **3.2× the cost** ($4.82 vs. $1.49), and `medium` was *"not measurably higher"* than `low` (+0.5 pts for 1.45× cost).
     - On FrontierCode 2.0 (`which-claude-model-should-you-build-with.md` L77): **Fable 5.2 at `high` tied Opus 5.5 at `max` (45.3% vs. 45.5%) for roughly half the cost per task.**
     - Latency (`optimizing-for-cost-and-intelligence.md` L460): On DRACO, Opus 5.5 at `low` took **84% less time** than at `medium`.
2. **Terminal-Bench 3.0 & worked build timings (`https://claude.dev/blog/spending-your-effort/` + embedded dataset):**
   - **Wall-clock timings on Opus 5.5 builds:**
     - Underspecified fitness app: `low` **1.5 min** → `medium` **4 min** → `high` **11 min** → `max` **67 min**.
     - Lightly specified `/config` redesign: `low` **1 min** → `max` **28 min**.
     - Highly specified fitness app: `low` **16 min** → `medium` **22 min** → `high` **33 min** → `max` **79 min**.
     - Edge-case tasks on Opus 5.5: `mvcc-lsm-compaction` went 0/5 (`low`, ~1 min) → 4/5 (`xhigh`, ~11 min); `cli-2ph-simplex` 0/5 (`low`, ~10k tokens) → 5/5 (`high`); `gsea-proteomics` 0/5 (`low`) → 4/5 (`high`).
   - **Terminal-Bench 3.0 embedded JSON points (370 attempts across 70 tasks; Fable 5.1, Opus 5, Fable 5):**
     - **Fable 5.1:** `low`: 140/370 (37.8%, median **72,982** tokens) → `medium`: 158/370 (42.7%, **92,970** tok) → `xhigh`: 217/370 (58.6%, **177,400** tok) → `max`: 214/370 (57.8%, **222,480** tok). Category lifts (`low`→top): Security 64%→87%, Hardware 34%→75%, ML 54%→73%, Science 41%→61%, Software 43%→56%, Media 18%→30%, Operations 12%→22%.
     - **Opus 5:** `low`: 108/369 (75,655 tok) → `medium`: 141/369 (118,795 tok) → `effort-120`: 178/370 (253,980 tok) → `max`: 193/370 (308,346 tok).
     - *(Note: Opus 5.5's Fig A line in `spending-your-effort` is rendered as static SVG curves without an inline JSON token table.)*
   - **Prompt caching interaction (`prompt-caching.md` L146):** On an Anthropic API key or Claude subscription, changing effort mid-session via `/effort` **preserves the prompt cache** on Fable 5.1+, Opus 5.5, Sonnet 5.5, and Haiku 5.5 (on Bedrock/Vertex/gateways it invalidates the cache).

#### Effort for subagents and inherited effort
**Label:** `documented`
- **URLs:** `https://code.claude.com/docs/en/model-config.md` (L669, L673, L766–L768) & `https://code.claude.com/docs/en/sub-agents.md` (L435–L439)
- **Precedence for a subagent's effort (`sub-agents.md` L435–L439):**
  1. The subagent's own `effort` frontmatter/parameter.
  2. The `effort` set for the parent subagent that spawned it, or a per-turn level (e.g., a skill's `effort` frontmatter).
  3. `CLAUDE_CODE_SUBAGENT_EFFORT` environment variable.
  4. Otherwise: **Exact quote (`model-config.md` L768):**
     > *"On the main conversation's model, a subagent or teammate with no other level set runs at the level you pick in the session or set with `--effort`. On another model, that level doesn't apply. To pick a subagent's level, set `effort` in its definition."*
- **Defaults (`model-config.md` L669):** In Claude Code, **Fable 5.2, Opus 5.5, Sonnet 5.5, and Haiku 5.5 default to `medium`** (whereas Fable 5.2 defaults to `high` on the raw Messages API). All levels can be hard-capped globally or per model via `maxEffortLevel` (`settings-reference.md` L503–L525).

---

### 1.4 Subscription Limits (Official Statements)

**Label:** `documented` (exact cache/output subscription weights: `not found`)
- **URLs:**
  - `https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work`
  - `https://support.claude.com/en/articles/11145838-use-claude-code-with-your-pro-max-or-ultra-plan`
  - `https://support.claude.com/en/articles/15424964-claude-fable-models-on-your-plan`
  - `https://support.claude.com/en/articles/17728952-what-is-the-ultra-plan`
  - `https://support.claude.com/en/articles/17007452-what-is-a-limit-reset`
  - `https://support.claude.com/en/articles/17506494-claude-fable-5-2-launch-offers`
  - `https://support.claude.com/en/articles/9797557-usage-limit-best-practices`
  - `https://support.claude.com/en/articles/12429409-manage-usage-credits-for-paid-claude-plans`
- **How the weekly limit is counted:**
  - Shared across all surfaces (claude.ai, Desktop, Mobile, Claude Code). Resets at a fixed weekly day/time assigned to the account (`11647753`).
  - Exact quote (`11647753`): *"Your usage is affected by several factors, including the length and complexity of your conversations, the features you use, which Claude model you're chatting with, and the effort level you've selected."*
- **Separate limit for Fable (`15424964`):**
  - On **Max, Ultra, Team premium seats, and legacy seat-based Enterprise premium seats**, Fable models (5.2, 5.1, 5) are included up to **50% of your weekly usage limits** at no extra cost.
  - **It is a sub-cap, NOT extra capacity.** Exact quote (`15424964`):
    > *"No. You can use up to 50% of your weekly limit on Fable models, but your use of other models draws from the same usage limits and you can never use more than your weekly limit."*
  - *"They draw from your plan's regular weekly usage limits and use them faster than other Claude models."* On Pro and standard Team/Enterprise seats, Fable runs on pay-as-you-go usage credits.
- **Whether cache reads, cache writes, and output are weighted & how:**
  - **Official subscription formula:** `not found` — Anthropic publishes **no explicit token-weighting formula** for subscription weekly limits.
  - The only official statements on weighting are qualitative: `9797557` states *"Content in projects is cached and counts less against your limits when reused"*, and `claude.dev/blog/lessons-from-building-claude-code-prompt-caching-is-everything.md` states *"A high prompt cache hit rate decreases costs and helps us create more generous rate limits for our subscription plans"*.
  - Once you overflow into **Usage credits**, billing switches explicitly to **standard API pricing rates** (`12429409`): Opus 5.5 **$4 input / $5 (5m write) / $8 (1h write) / $0.20 cache read / $20 output** per MTok; Fable 5.2 **$8 / $10 / $16 / $0.20 / $40** per MTok (`platform.claude.com/docs/en/about-claude/pricing.md`).
- **Highest consumer plan tier (`17728952`):**
  - **Ultra plan** — **$200/month** (formerly Max 20x; Max 5x is now Max).
  - Exact quotes (`17728952`): *"Get 10 times the Pro plan's usage every week."* / *"No five-hour limit for verified accounts: Work at your own pace within your weekly limit."* / *"Monthly API credits: Ultra plans include monthly credits for the Claude API"* (cannot be used for Claude Code or usage credits, `11145838`).
- **November 2026 limit resets (`17506494` & `17007452`):**
  - **Claude Fable 5.2 launch resets (launched Nov 9, 2026; billing cutoff: paid billing active before Nov 9, 2026 at 10:00 am PT):**
    - **Who gets it & how many:**
      - **Max, Ultra, Team premium, legacy seat-based Enterprise premium:** **1 reset** (<6 months since first paid bill), **2 resets** (6–12 months), **3 resets** (>12 months). Refunds weekly usage spent on Fable models (leaving other models untouched) and refills the 5-hour limit if the account has one. **Deadline:** expires **March 8, 2027 at 11:59 pm PT**.
      - **Pro & Team standard seats:** Fable included up to 50% of weekly limit for one week + **1 Fable reset**. **Deadline:** expired **November 16, 2026 at 11:59 pm PT**.
      - Excluded: Free, legacy Enterprise standard, self-serve Enterprise, usage-based Enterprise.
    - **Where claimed (`17007452`):** **Settings > Usage → Limit resets → "Reset for free"** on Claude web, Claude Desktop, or Claude Mobile. **Not available in Claude Code CLI/IDE**, though restored quota applies immediately to Claude Code.
  - *(Note: Search results also mentioned a separate Nov 10, 2026 @ClaudeDevs general full-limit reset expiring Dec 10, 2026, but no official `support.claude.com` or `anthropic.com` page for that second reset could be located/opened directly — see Section 5.)*

---

### 1.5 Review Loops (`solving-code-review-at-anthropic` & `REVIEW.md`)

**Label:** `documented` + `measured`
- **URLs:**
  - `https://claude.dev/blog/solving-code-review-at-anthropic/` (Oct 15, 2026, read in full)
  - `https://code.claude.com/docs/en/code-review.md`
  - `https://code.claude.com/docs/en/code-review-customize.md`
  - `https://code.claude.com/docs/en/code-review-pull-requests.md`
  - `https://code.claude.com/docs/en/code-review-local.md`
- **Complete list of changes Anthropic's Claude Code team made (`solving-code-review-at-anthropic`):**
  - *Outcome:* June–September 2026 merged PRs more than doubled while revert rate fell >40%; internal monorepo landed changes rose from ~2,500/day (May) to >30,000/day (Sep); substantive review comments rose from 16% to 54% of internal PRs.
  1. **Pre-PR verification & fresh-context review:** Agent runs tests/build/screenshots and shows output; a fresh-context subagent + internal skills (bug hunt, diff simplification, end-to-end check, UI design check) run before PR creation. Repeated reviewer flags are moved into `CLAUDE.md`, `REVIEW.md`, a skill, or a lint rule.
  2. **Multi-agent PR review on every PR** (Quick / Standard / Deep levels) + **multi-agent voting & noise filtering** (see below). Every finding must be fixed or thread-resolved with a reason.
  3. **Blocking automated security review:** PR cannot merge while any bug-review or security-review finding is open; auto-clears when a later commit fixes it.
  4. **Blocking CI + structured PR template** ("Testing", "Risk", "How to revert").
  5. **Risk-tiered human approval (change sets tier, not author):**
     - **Low** (docs, copy, internal-only, additive behind flag): 0 human approvals once automated review & security are green (trusted authors only).
     - **Medium** (ordinary product changes, caching, auto-updater, system prompts): 1 human.
     - **High** (highest-risk code, sensitive paths, CI workflows): 2 humans.
     - Two classifiers pick the tier and check for exceptions; bot approval never fills a required human slot; approval is commit-pinned and dismissed on new agent pushes; the human directing an agent-written PR counts as one reviewer.
  6. **Merge queue, feature flags, employee-first dogfooding, latest vs. stable release channels, and fast rollback** with weekly on-call.
  7. **Incident-driven daily evals & noise tracking:** Past incident/revert PRs become a daily eval set; every review comment is tracked by author response (fixed / wrong / intentional / didn't matter) plus clean-PR false-positive runs.
  8. **Convergence changes (to stop 50+ round overnight agent-vs-agent loops, where `git blame` showed ~59% of later-round findings were pre-existing code from round 1):**
     - **A 3-round cap:** *"After three rounds, the reviewer asks the author to step back and reconsider the approach."*
     - **Diff-only re-review** after the first few passes.
     - **Flag the class of bug** so the author fixes every instance, not one line.
     - **No repeats of findings already posted.**
     - **Nightly review agents** over existing code to catch old bugs outside PR loops.
  9. **On-demand code explainers & artifacts** instead of line-by-line trivial diff reading to build team context.
- **What a good `REVIEW.md` contains (`solving-code-review-at-anthropic` & `code-review-customize.md`):**
  - Short, review-only rules (keep general project context in `CLAUDE.md`; note `/code-review` locally reads `CLAUDE.md` only, **not** `REVIEW.md`).
  - Redefine what 🔴 **Important** means for that repo (e.g., docs = "it lies", config = "it misroutes").
  - Cap 🟡 **Nit** count (e.g., max 5 per review).
  - List paths/categories to skip (generated code, lockfiles, CI-enforced lint/formatting/types).
  - Require concrete `file:line` evidence rather than guessing from names.
  - Ask for a summary starting with a count and saying plainly when nothing needs fixing.
  - Supports `@file.md` and `@.claude/skills/<folder>/SKILL.md` includes (up to 8 files, 150 KB combined, 30 KB/skill, <=64 local links) and subdirectory `REVIEW.md` files (ignored under test/example/generated/vendor dirs) — **requires branch protection** on the default/base branch for severity lowering, exclusions, nit caps, and includes to apply (`code-review-customize.md` L331–L361).
  - **Verbatim example from `code-review-customize.md` (L191–L217):**
    ```markdown
    # Review instructions

    ## What Important means here

    Reserve Important for findings that would break behavior, leak data,
    or block a rollback: incorrect logic, unscoped database queries, PII
    in logs or error messages, and migrations that aren't backward
    compatible. Style, naming, and refactoring suggestions are Nit at
    most.

    ## Cap the nits

    Report at most five Nits per review.

    ## Do not report

    - Anything CI already enforces: lint, formatting, type errors
    - Generated files under `src/gen/` and any `*.lock` file
    - Test-only code that intentionally violates production rules

    ## Always check

    - New API routes have an integration test
    - Log lines don't include email addresses, user IDs, or request bodies
    - Database queries are scoped to the caller's tenant
    ```
- **How the three-round cap is enforced:**
  - In Anthropic's internal system (`solving-code-review-at-anthropic`): *"After three rounds, the reviewer asks the author to step back and reconsider the approach."*
  - In the public managed **Code Review** product (`code-review-pull-requests.md` L56–L60, L153–L158): automatic later reviews after pushes **focus only on what changed since the last review**, post fewer nits/pre-existing findings, and **skip re-reviewing** when no new changes or open Important findings warrant it (*"outside the scope of this re-review and not posted"*). Note: **no hard user-configurable numeric round-cap setting is documented** in the public Code Review docs (it's prompt/service behavior internally; for custom/local loops you enforce it in your harness/workflow).
- **How findings are filtered before a person sees them (`solving-code-review-at-anthropic` & `code-review.md` L66–L71):**
  - 4-stage pipeline: **Find** (several parallel agents) → **Verify** (separate group of agents votes on each finding against the code; only posted if enough agree it's real) → **Drop noise** (style, diagnostics, and cleanup-only findings are dropped — A/B tested on Anthropic's own PRs) → **Rank & tag** (deduplicated and sorted into 🔴 `Important`, 🟡 `Nit`, 🟣 `Pre-existing`).

---

## 2. Part 2: Anything New (September–November 2026)

### 2A. Features added since September 2026 that lower usage (not in "What is already known")

1. **Code Mode (experimental)** — `documented` (`v2.1.323` first-party, `v2.1.331` Bedrock/Vertex/Foundry)
   - **URL:** `https://code.claude.com/docs/en/code-mode.md`, `CHANGELOG.md` L5, L1256
   - **What it does:** Claude writes JavaScript to run multiple tool calls, filter outputs, and execute parallel commands in a single step — avoiding sending intermediate raw outputs and saving multi-turn prompt re-reads (Platform's programmatic tool calling docs cite **24% fewer input tokens**, `optimizing-for-cost-and-intelligence.md` L229).
   - **How to turn on:** Experimental opt-in documented at `code.claude.com/docs/en/code-mode` (off by default).
2. **`CLAUDE_CODE_DISABLE_AGENT_PROGRESS_SUMMARIES=1`** — `documented` (`v2.1.318`, documented in changelog `v2.1.323`)
   - **URL:** `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md` (L1268)
   - **What it does:** *"set it to `1` to turn off subagent progress summaries and the requests that write them (anthropics/claude-code#86879)"*. In `v2.1.330` (L256), gateway subagent progress summaries also switched from reading the whole conversation to short notes.
3. **`omitClaudeMd: true` on custom/plugin subagents** — `documented` (`v2.1.271`)
   - **URL:** `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md` (L5030), `https://code.claude.com/docs/en/sub-agents.md` (L224)
   - **What it does:** Lets custom and plugin subagents (or `--agents` JSON) skip user, project, and local `CLAUDE.md` files (managed policy still loads), cutting ~4,200+ startup tokens per subagent.
4. **`skillListingMaxDescChars`** — `documented`
   - **URL:** `https://code.claude.com/docs/en/settings-reference.md` (L2629–L2644)
   - **What it does:** Caps characters shown per skill (`description` + `when_to_use`) in the skill listing (default `1536`); lowering it (e.g., `256`–`512`) shrinks the ~10,700-token skill listing across every turn.
5. **`CLAUDE_CODE_IDLE_COMPACT_MIN_TOKENS`** — `documented` (`v2.1.290` / changelog `v2.1.299`)
   - **URL:** `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md` (L2824), `https://code.claude.com/docs/en/settings-reference.md` (L2545)
   - **What it does:** Changes Claude Code's native idle-compaction minimum from `200000` (cannot go below 100K per T3 comment).
6. **`/doctor prompt-audit [path]`** — `documented` + `measured` (`v2.1.283`)
   - **URL:** `https://code.claude.com/docs/en/commands.md` (L85), `https://claude.dev/blog/what-a-task-costs-on-opus-5-5.md` (L173–L175)
   - **What it does:** Audits `CLAUDE.md`, skills, agents, and commands for stale paths/commands and older-model prompting anti-patterns that make Opus 5.5 repeat tool calls. Anthropic measured **an extra 9% cost reduction** on top of Opus 5.5's 18% migration savings on a 44-ticket benchmark.
7. **Mid-session deferred MCP/plugin tools (`v2.1.267`) & byte-stable tool lists (`v2.1.268`) & `_meta['anthropic/alwaysLoad']: false` (`v2.1.285`)** — `documented`
   - **URL:** `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md` (L5352, L5287, L4005)
   - **What they do:** Prevent late-connecting MCP/plugin tools from rewriting the tool list and busting the prompt cache mid-session.
8. **Prompt suggestion auto-throttling (`v2.1.283`, `v2.1.325`) & `claudeCode.promptSuggestions` (`v2.1.326`)** — `documented`
   - **URL:** `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md` (L4229, L1222, L1094)
   - **What it does:** Refreshes prompt suggestions less often after 20 unused suggestions or 8 consecutive background/scheduled turns, saving background token spend.
9. **`/effort` single-session confirmation (`s` key, `v2.1.257`)** — `documented`
   - **URL:** `https://code.claude.com/docs/en/model-config.md` (L678)
   - **What it does:** Lets you bump effort for one hard turn/session without permanently saving `high`/`xhigh` into `modelSettings` for all future sessions.
10. **Dynamic workflow token budgets** — `documented`
    - **URL:** `https://claude.dev/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code.md`
    - **What it does:** You can instruct a dynamic workflow with an explicit cap (e.g., `"use 10k tokens"`), enforced on the workflow run.
11. **`/usage` prompt cache diagnostics (`v2.1.251`, likely-cause `v2.1.260`) & one-key `f` fix shortcut (`v2.1.305`)** — `documented`
    - **URL:** `https://code.claude.com/docs/en/costs.md` (L209–L220), `CHANGELOG.md` (L2505)
    - **What it does:** Names the exact cause of cache misses (`likely cause: tool definitions changed`) and auto-applies top usage recommendations.
12. **Advisor tool (`/advisor`, `advisorModel`, `--advisor`, `CLAUDE_CODE_DISABLE_ADVISOR_TOOL=1`)** — `documented` + `measured` (caveat!)
    - **URL:** `https://code.claude.com/docs/en/advisor.md`, `https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence.md` (L560–L597)
    - **Caveat from Anthropic's own measurements:** The advisor tool adds ~1,000 prompt tokens per request and **its read of the conversation is NOT cached** (`advisor.md` L173). On Anthropic's internal coding benchmark, Opus 5.5 (`high`) + Fable 5.1 advisor scored +1.7 pts at **2.1× the cost** ($2.92/attempt), and *"none [of the four pairings] clearly beat its executor alone"* (`which-claude-model-should-you-build-with.md` L162–L164). Disable with `CLAUDE_CODE_DISABLE_ADVISOR_TOOL=1` if unused.

---

### 2B. Guidance on Orchestrating Many Agents Cheaply

**Label:** `documented` + `measured`
- **URLs:**
  - `https://code.claude.com/docs/en/sub-agents.md` (L1088–L1312)
  - `https://code.claude.com/docs/en/agents.md` (L1–L48)
  - `https://claude.dev/blog/what-a-task-costs-on-opus-5-5.md` (L155–L163, L245)
  - `https://claude.dev/blog/which-claude-model-should-you-build-with.md` (L121–L170)
  - `https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence.md` (L599–L625)
  - `https://claude.dev/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code.md`
- **Forks vs. subagents vs. separate sessions:**
  - **Forked subagent (`Agent` fork / `/subtask`, default-on from `v2.1.302`; disable with `CLAUDE_CODE_FORK_SUBAGENT=0` or `"Agent(fork)"` deny):** Shares the parent's exact system prompt, tool pool, and message history, so **its first request hits the parent's warm prompt cache** (`sub-agents.md` L1299–L1301: *"Because of that reuse, a fork costs less than a fresh subagent for tasks that need the same context."*). Only its final result returns to the parent.
  - **Non-fork subagent:** Separate cache & fresh context — best when the child needs isolation, a cheaper model (`model: haiku` or `sonnet`), or `omitClaudeMd: true` so large file/log reads stay out of the main window.
  - **Agent teams:** *"Agent teams in plan mode use about **7× the tokens** of a standard session"* (`what-a-task-costs-on-opus-5-5.md` L245).
  - **Separate sessions / T3 delegated child threads:** Each pays a cold start (~70K tokens unless given a lean `CLAUDE_CONFIG_DIR`) and creates an independent cache. In Anthropic's orchestrator benchmark (`optimizing-for-cost-and-intelligence.md` L601–L625), delegation beat a single model **only** on (1) routine work with a long cost tail or (2) work larger than one context window (e.g., defect-search: Opus 5.5 + Sonnet 5.5 subagents matched Opus 5.5 alone at `medium` for **~⅓ less cost**). On dependent chains or single-context work, **one model at lower effort beat multi-agent delegation in every measured case**.
- **Built-in `Explore` subagent trap (`which-claude-model-should-you-build-with.md` L152):**
  > *"In current versions of Claude Code, the built-in Explore subagent runs on your session's model instead of always running on Haiku. On the Claude API, that model is capped at Opus. To keep exploration cheap, define your own subagent named `Explore` with `model: haiku`."*
- **Handoff files & restarting a coordinator (`which-claude-model-should-you-build-with.md` L123 & `what-a-task-costs-on-opus-5-5.md` L155, L220–L224):**
  - Switching models mid-session invalidates the entire prompt cache and drops Fable 5.2 thinking blocks (`which-claude-model-should-you-build-with.md` L129).
  - Instead, run `/compact` while the cache is still warm (rough price at 150K tokens on Opus 5.5: ~$0.25, pays back within ~10 turns by saving ~$0.025/turn), OR **`/clear` and start a fresh coordinator session with a short written plan/handoff file** (note: `what-a-task-costs-on-opus-5-5.md` L212: 30 turns at 150K context spend $0.90 on cache reads alone vs. $0.12 at 20K).
- **Waking a parent / long-gap cost (`what-a-task-costs-on-opus-5-5.md` L191 & `optimizing-for-cost-and-intelligence.md` L203):**
  - At 120K context on Opus 5.5, **one 5m cache write costs ~$0.60 (= 25 cache reads)** and **one 1h cache write costs ~$0.96 (= 48 cache reads)**; on Fable 5.2, a 5m write is **50× a cache read** and a 1h write is **80× a cache read**. Waking a large idle parent coordinator after its cache expires (or waking it repeatedly for small child completions instead of batching) triggers either a full cold re-read or repeated huge cache-read turns.
  - Specific measurements on **waking a parent thread in T3 Code**: `not found` in official docs beyond the per-turn cache math above.

---

### 2C. Tool-Result Clearing and Context Editing Inside Claude Code

**Label:** `documented` + `reported` (GitHub issues)
- **URLs:**
  - `https://code.claude.com/docs/en/costs.md` (lines 212, 218)
  - `https://platform.claude.com/docs/en/build-with-claude/context-editing.md`
  - `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md` (L9953)
  - `https://github.com/anthropics/claude-code/issues/42542`
  - `https://github.com/anthropics/claude-code/issues/57788`
- **Does it exist inside Claude Code? YES — officially confirmed, though sparsely documented.**
  - `costs.md` L212 & L218 explicitly reference it in `/usage`:
    > `1 expected rebuild (compaction or tool-result clearing)`
    > *"when Claude Code has itself just rewritten the conversation, by compaction or by clearing old tool results from context, it counts the same kind of miss as an expected rebuild instead."*
  - `CHANGELOG.md` v2.1.50 (L9953): *"Improved memory usage during long sessions by clearing large tool results after they have been processed."*
- **When it runs (reported from source analysis in `anthropics/claude-code#42542`, open):**
  - Internally called **`microcompact`** (`src/services/compact/microCompact.ts`), operating in two modes controlled by server-side GrowthBook flags:
    1. **Time-based microcompact:** after an inactivity gap past a GrowthBook threshold, replaces old tool result contents with `[Old tool result content cleared]` (causing an expected cache rebuild).
    2. **Cached microcompact:** while the cache is warm, uses server-side `cache_edits` (analogous to the API's `clear_tool_uses_20250919` / `clear_thinking_20251015` documented in `platform.claude.com/docs/en/build-with-claude/context-editing.md`) to drop old tool results without busting the full prefix.
- **Can it be tuned by Claude Code users? NO.**
  - No user-facing setting or environment variable exists in `settings-reference.md` or `env-vars.md` to configure or disable microcompact independently (`#42542` explicitly requests `DISABLE_MICROCOMPACT`; `DISABLE_AUTO_COMPACT` does not stop it, and `DISABLE_COMPACT` disables manual `/compact` too).
  - By contrast, on the direct **Messages API**, `clear_tool_uses_20250919` and `clear_thinking_20251015` are fully tunable (`trigger`, `keep`, `clear_at_least`, `exclude_tools`).

---

### 2D. Measured Context Costs of Popular Plugins & MCP Servers

**Label:** `measured`
- **URLs:**
  - `https://www.anthropic.com/research/advanced-tool-use` (Anthropic official measurements)
  - `https://www.anthropic.com/research/code-execution-with-mcp` (Anthropic official measurements)
  - `https://code.claude.com/docs/en/plugins/measure.md` (Claude Code official measurement tool)
  - `https://scottspence.com/posts/optimising-mcp-server-context-usage-in-claude-code` (third-party `/context` measurement)
  - `https://github.com/mksglu/context-mode` (third-party benchmarked MCP output compression)
- **Measurements & methods:**
  1. **Anthropic (`advanced-tool-use`, upfront schema token count across 5 common MCP servers):**
     - **GitHub MCP:** 35 tools = **~26,000 tokens**
     - **Slack MCP:** 11 tools = **~21,000 tokens**
     - **Jira MCP:** **~17,000 tokens**
     - **Sentry MCP:** 5 tools = **~3,000 tokens**
     - **Grafana MCP:** 5 tools = **~3,000 tokens**
     - **Splunk MCP:** 2 tools = **~2,000 tokens**
     - Total for 58 tools across 5 servers = **~55,000 tokens** upfront (~72,000 tokens for 50+ tools in their Tool Search comparison; Tool Search deferred loading preserved **191,300** tokens of a 200K window vs. **122,800** with upfront loading).
  2. **Anthropic (`code-execution-with-mcp`):**
     - Intermediate raw MCP tool outputs (e.g., a 2-hour meeting transcript from `gdrive.getDocument`) consume **~50,000 tokens** flowing through the model twice; filtering in code before returning to context avoids it.
  3. **Scott Spence (`optimising-mcp-server-context-usage-in-claude-code`, measured via Claude Code's `/context` warning output):**
     - **Playwright MCP:** 21 tools = **~13,647 tokens**
     - **mcp-omnisearch:** 20 tools = **~14,114 tokens**
     - **mcp-sqlite-tools:** 19 tools = **~13,349 tokens**
     - **n8n-workflow-builder:** 10 tools = **~7,018 tokens**
     - 12 active servers total = **~81,986 tokens** before the first prompt.
  4. **Context Mode (`github.com/mksglu/context-mode`, Nov 1–3, 2026 benchmarks, median of 5 runs on Claude Code 2.1.312 / Haiku 5.5 & Opus 5.5):**
     - Sandbox/hook filtering reduced raw tool outputs entering context by **98.8% across a full debug session** (281,261 B → 3,266 B), **99.8% on `git log --stat`** (164 KB → 268 B), and **99.8% on a 265 KB WebFetch** (271 KB → 439 B).
     - Its Recall gateway A/B test on Opus 5.5 (200K window, 2026-10-04) cut mean session cost from **$4.8627 to $3.0353 (-37.6%)**.
  5. **Per-plugin measurement built into Claude Code (`plugins/measure.md`):**
     - Run `claude plugin details <name>` in your shell to get exact **Always-on** (every turn) vs. **On-invoke** token estimates per skill/agent/command, and `/context` for runtime MCP tool schemas.

---

### 2E. Anything Else With Numbers for This Profile

**Label:** `measured` + `documented`
1. **Claude Code cut >80% of its own system prompt for Claude 5-gen models** (`https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models.md`, L3 & L17) — and recommends the same progressive-disclosure approach for your own `CLAUDE.md` and `SKILL.md` files (keep root `CLAUDE.md` tiny, link to a tree of reference files).
2. **Cache-break cost measured on 100K-token cached conversations** (`https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence.md`, L197–L217):
   - Editing the system prompt mid-session cost up to **$1.26**, changing top-level effort (where not preserved) cost up to **$1.02**, whereas putting the update in a user message cost **$0.026** (~48× cheaper).
   - Putting a **25-token dynamic status line** at the front of the system prompt turned every request into a full cache write.
3. **Elapsed-time clock prompting (`optimizing-for-cost-and-intelligence.md` L40):** Telling an agent time matters and showing elapsed time cut run time by **7% to 47%** for only a few points of score.
4. **Haiku 5.5 pricing cliff at 100K tokens (`pricing.md` & `which-claude-model-should-you-build-with.md` L113):** Haiku 5.5 jumps **5× in price** once a prompt crosses 100,000 tokens ($0.10/$0.50 → $0.50/$2.50 per MTok; cache reads $0.01 → $0.05). Keeping Haiku subagents under 100K (easy when their startup context is lean, not 70K!) preserves the 5× cheaper tier.
5. **Opus 5.5 vs. Fable 5.2 cache-read parity (`pricing.md` & `which-claude-model-should-you-build-with.md` L107):** Both Opus 5.5 and Fable 5.2 charge **$0.20/MTok for cache reads** (0.05× on Opus 5.5, 0.025× on Fable 5.2), while Fable 5.2 charges **2× on uncached input ($8 vs $4), cache writes ($10/$16 vs $5/$8), and output ($40 vs $20)**. Thus minimizing **cache writes (cold starts + 1h TTL on short children)** and **output tokens (effort level)** dominates Fable 5.2's weekly-limit burn.

---

## 3. Table of New Settings, Flags, Commands & Environment Variables Found in This Round

| Name | Allowed Values | Where Set | What It Does | Version Needed | URL |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `skillOverrides` | Object: skill name → `"on"`, `"name-only"`, `"user-invocable-only"`, `"off"` | Any `settings.json` (`/skills` writes `.claude/settings.local.json`) | Hides or collapses non-plugin skills from Claude's context and/or `/` menu (does **not** cover plugin skills) | v2.1.129+ (alias keys v2.1.260+) | `https://code.claude.com/docs/en/settings-reference.md` |
| `skillListingMaxDescChars` | Positive integer (default `1536`) | Any `settings.json` | Caps per-skill description + `when_to_use` characters in the skill listing | Current (docs) | `https://code.claude.com/docs/en/settings-reference.md` |
| `syncClaudeAiPlugins` | `false` (`true` = unset) | User, local, managed `settings.json`, `--settings` | Stops downloading/loading claude.ai synced plugins (`<name>@synced`) | v2.1.273+ (changelog v2.1.275) | `https://code.claude.com/docs/en/settings-reference.md` |
| `syncClaudeAiSkills` | `false` (`true` = unset) | User, local, managed `settings.json`, `--settings` | Stops downloading/loading claude.ai synced skills | v2.1.275+ | `https://code.claude.com/docs/en/settings-reference.md` |
| `disableClaudeAiConnectors` | `true`, `false` (any-source-true) | Any `settings.json` | Stops Claude Code from fetching/connecting claude.ai remote MCP connectors | Current (fixes v2.1.273–290) | `https://code.claude.com/docs/en/settings-reference.md` |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | `false` | Env var / `settings.json` `env` | Per-session disable for fetched claude.ai MCP connectors | Current | `https://code.claude.com/docs/en/mcp.md` |
| `promptCacheTtl` / `CLAUDE_CODE_PROMPT_CACHE_TTL` | `"5m"`, `"1h"` | Any `settings.json` / env var | Sets main-conversation (and SDK session) prompt cache TTL | v2.1.242+ | `https://code.claude.com/docs/en/prompt-caching.md` |
| `subagentPromptCacheTtl` / `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` | `"5m"`, `"1h"` | Any `settings.json` / env var | Sets TTL for subagents/workflows/helpers; **T3 also copies this into `CLAUDE_CODE_PROMPT_CACHE_TTL` for delegated children** | v2.1.242+ | `https://code.claude.com/docs/en/settings-reference.md` |
| `FORCE_PROMPT_CACHING_5M` | `1` | Env var | Forces 5m TTL across both buckets, overriding all other TTL settings | v2.1.242+ | `https://code.claude.com/docs/en/prompt-caching.md` |
| `CLAUDE_CODE_IDLE_COMPACT_MIN_TOKENS` | Integer >= `100000` (default `200000`) | Env var / `settings.json` `env` | Adjusts minimum context threshold for Claude Code's native idle compaction | v2.1.290+ (changelog v2.1.299) | `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md` |
| `CLAUDE_CODE_DISABLE_AGENT_PROGRESS_SUMMARIES` | `1` | Env var / `settings.json` `env` | Turns off subagent progress summaries and the API requests that generate them | v2.1.318+ (changelog v2.1.323) | `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md` |
| `omitClaudeMd` | `true`, `false` | Agent frontmatter / `--agents` JSON | Excludes user, project, and local `CLAUDE.md` from custom/plugin subagents | v2.1.271+ | `https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md` |
| `CLAUDE_CODE_SUBAGENT_EFFORT` | `low`, `medium`, `high`, `xhigh`, `max` | Env var / `settings.json` `env` | Sets default effort for subagents without their own `effort` field | Current | `https://code.claude.com/docs/en/sub-agents.md` |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Model alias or ID (e.g., `haiku`, `sonnet`) | Env var / `settings.json` `env` | Forces a model onto all subagents without an explicit model | Current | `https://code.claude.com/docs/en/sub-agents.md` |
| `maxEffortLevel` | `"low"`, `"medium"`, `"high"`, `"xhigh"`, `"max"` | Any `settings.json` (top-level or under `modelSettings.<id>`) | Caps effort across all sources (lowest cap across scopes wins) | v2.1.267+ | `https://code.claude.com/docs/en/settings-reference.md` |
| `CLAUDE_CODE_FORK_SUBAGENT` | `"0"` | Env var / `settings.json` `env` | Disables automatic fork subagents (on by default from v2.1.302) | v2.1.232+ / v2.1.302+ | `https://code.claude.com/docs/en/sub-agents.md` |
| `disableAllHooks` | `true`, `false` | Any `settings.json` / `--settings` | Disables all user/project/local/plugin hooks, status line, and `fileSuggestion` | Current (SDK `managedSettings` v2.1.309+) | `https://code.claude.com/docs/en/settings-reference.md` |
| `--tools` | `""`, `"default"`, or comma list e.g. `"Bash,Edit,Read"` | CLI / T3 Launch arguments | Restricts built-in tools sent to Claude (does not affect MCP or `EndConversation`) | Current | `https://code.claude.com/docs/en/cli-reference.md` |
| `--disallowedTools` / `--disallowed-tools` | Bare tool names, globs (`"mcp__*"`), or scoped rules | CLI / T3 Launch arguments | Bare names and globs remove matching tools from Claude's context | Current | `https://code.claude.com/docs/en/cli-reference.md` |
| `--strict-mcp-config` | Flag | CLI / T3 Launch arguments | Skips `.mcp.json`, `~/.claude.json`, and plugin MCP servers | Current | `https://code.claude.com/docs/en/cli-reference.md` |
| `--disable-slash-commands` | Flag | CLI / T3 Launch arguments | Disables all skills and commands for that session | Current | `https://code.claude.com/docs/en/cli-reference.md` |
| `/skill-doctor` | Command | Interactive session | Audits skill token cost and invocation frequency to prune unused skills | v2.1.252+ (changelog v2.1.261) | `https://code.claude.com/docs/en/commands.md` |
| `/doctor prompt-audit [path]` | Command | Interactive session | Audits `CLAUDE.md`/skills/agents for older-model prompting anti-patterns | v2.1.283+ | `https://code.claude.com/docs/en/commands.md` |
| `claude plugin eval` | CLI subcommand (`--ablation none`, `--max-cost-usd`, `--disallow-tools`, `--json`) | Shell | A/B evaluates a plugin against a no-plugin baseline with scored JSON/HTML reports | v2.1.269+ (git >=2.31 v2.1.283+) | `https://code.claude.com/docs/en/plugin-evals.md` |
| `claude plugin details <name>` | CLI subcommand | Shell | Prints always-on and on-invoke token estimates per plugin component | v2.1.139+ | `https://code.claude.com/docs/en/plugins/measure.md` |
| `advisorModel` / `/advisor` / `--advisor` / `CLAUDE_CODE_DISABLE_ADVISOR_TOOL=1` | `"sonnet"`, `"opus"`, `"fable"`, `"off"` / `1` | `settings.json` / interactive / CLI / env | Enables or disables the server-side advisor tool (warning: advisor reads are uncached) | v2.1.300+ (AWS) / Fable 5.2 v2.1.321+ | `https://code.claude.com/docs/en/advisor.md` |
| Code Mode | Experimental opt-in | Claude Code config | Lets Claude batch/filter tool calls in JavaScript in one turn | v2.1.323+ (cloud providers v2.1.331+) | `https://code.claude.com/docs/en/code-mode.md` |
| T3 `autoResumeLimitedThreads` / `snoozeLimitedThreads` | Boolean | T3 **Settings → Threads** | Auto-resumes or snoozes usage-limited threads when reset time arrives | Present in `3380585e` | `https://github.com/pingdotgg/t3code/blob/3380585e1bfcc99634ee78edf02624d396886fd2/docs/user/thread-sidebar.md#L275-L287` |

---

## 4. Top 8 Changes From This Round With Largest Expected Savings (With Arithmetic)

Baseline profile from prompt: **18,100 Opus 5.5 / Fable 5.2 requests/week**, **239,000 avg prompt tokens/request** (~**4.33 billion** prompt tokens/week), **70,000-token cold start** per new/delegated thread, **~325 delegated child threads/week**. Reference API rates (used as consistent cost proxy since Anthropic doesn't publish exact subscription weights): Opus 5.5 = $4 input / $5 (5m write) / $8 (1h write) / $0.20 cache read / $20 output per MTok; Fable 5.2 = $8 / $10 / $16 / $0.20 / $40 per MTok.

1. **Create a lean second Claude provider instance in T3 (`CLAUDE_CONFIG_DIR=~/.claude_worker`) for delegated children (`delegate_task` with `target.providerInstanceId`)**
   - *Setup:* Disable unused plugins, set `syncClaudeAiPlugins: false`, `syncClaudeAiSkills: false`, `disableClaudeAiConnectors: true`, `disableAllHooks: true` (drops 3.5K SessionStart hook), `skillListingMaxDescChars: 256` or `--disable-slash-commands` (drops 10.7K skills), and `--disallowedTools "mcp__t3-code__preview_*,mcp__t3-code__pull_request_*,mcp__t3-code__t3_project_*,mcp__t3-code__device_*,mcp__t3-code__t3_theme_*,mcp__t3-code__t3_worktree_*"` (cuts deferred tools).
   - *Arithmetic:* Trimming **~25,000–30,000 tokens** off the 70K cold start across 325 delegated children/week (say ~30 requests/child = ~9,750 requests): saves **~9.75M cold cache-write tokens/wk** (325 × 30K = $49–$156 API-eq/wk depending on 5m/1h & Opus/Fable) **plus ~292M cache-read tokens/wk** (9,750 × 30K = ~$58/wk), **total ~300M prompt tokens/wk (~7% of all prompt tokens, ~$110–$215 API-eq/wk)**.

2. **Explicitly set `"subagentPromptCacheTtl": "5m"` in `~/.claude/settings.json` (closes the T3 delegated-child 1h-TTL leak)**
   - *Why:* Per `ClaudeAdapterV2.ts#L2380-L2405`, leaving `subagentPromptCacheTtl` unset causes every T3 delegated child thread to default to a **1-hour (2×)** cache write instead of **5-minute (1.25×)** — a **37.5% penalty on every cache write** for short-lived children that rarely pause >5 minutes.
   - *Arithmetic:* 325 delegated children/week × ~70K startup + ~80K accumulated new input = **~48.8M cache-write tokens/week** across children. Dropping write multiplier from 2× to 1.25× cuts write cost by **0.75× base input**: on Opus 5.5 ($3/MTok saved) = **~$146 API-eq/wk**; on Fable 5.2 ($6/MTok saved) = **~$293 API-eq/wk** (blended ~**$175–$220/wk**) with zero quality change.

3. **Run default coding & delegated implementation on `medium` (or `low` + `high`-on-failure), reserving `high`/`xhigh` for verification/security, and capping runaway effort with `maxEffortLevel`**
   - *Basis:* Anthropic's SWE-bench Pro measurements (`optimizing-for-cost-and-intelligence.md` L434–L479): Opus 5.5 `low` ($0.11) → `medium` ($0.21) → `high` ($0.29) → `xhigh` ($0.74); `low` + `high`-on-failure = **$0.17** (41% below all-`high` at higher pass rate). Fable 5.2 `high` ($4.82) is **2.23×** `medium` ($2.16) and **3.23×** `low` ($1.49). Remember output/thinking tokens cost **100×** cache reads on Opus 5.5 and **200×** on Fable 5.2.
   - *Arithmetic:* Even shifting 30% of 18,100 requests (~5,430 requests) down one effort notch (e.g., `high`→`medium` saves ~28% total task cost; `xhigh`→`high` saves ~61% on Opus 5.5) cuts total output+turn consumption by **~10–20% across the whole weekly workload** — the single largest lever on subscription weekly limits.

4. **Trim the 70K-token baseline in main threads too (`syncClaudeAiPlugins: false`, `syncClaudeAiSkills: false`, `disableClaudeAiConnectors: true`, `/skill-doctor` + `skillOverrides`, `skillListingMaxDescChars: 300`, per-project `enabledPlugins: false`)**
   - *Arithmetic:* Removing ~15,000–20,000 tokens of unused skill descriptions (from 10.7K), deferred MCP names (from 7.3K), and synced plugins across all **18,100 requests/week** saves **271M–362M prompt tokens/week** (at $0.20/MTok cache read = **~$54–$72/wk** in reads alone, plus **~$15–$40/wk** in cold/compaction cache writes; **~6–8% of total prompt tokens**).

5. **Use T3's `Auto-compact after` (e.g., `250000`–`350000`) and `Compact idle threads: Before the cache expires (beta)` (plus `/clear` between unrelated tasks)**
   - *Basis:* Default auto-compact waits until ~967K tokens; `what-a-task-costs-on-opus-5-5.md` L212–L224 shows 30 turns at 150K spend $0.90 in reads vs. $0.12 at 20K, and a warm-cache `/compact` (~$0.25) pays back in ~10 turns while avoiding a cold re-read after 1h expiry (which costs 25–80× a cache read).
   - *Arithmetic:* Lowering average prompt size across 18,100 requests by 25% (239K → 179K = **-60K tokens/request**) removes **1.09 billion cache-read tokens/week** (**~$217 API-eq/wk** on cache reads alone) and prevents cold re-writes on long threads returning after 1 hour (~$1.90–$3.80 saved per 240K-token thread compacted warm instead of re-cached cold).

6. **Route read-only/search/log-reading subagents to Sonnet 5.5 or Haiku 5.5 (including defining a custom `Explore` subagent with `model: haiku` and `omitClaudeMd: true`)**
   - *Basis:* Built-in `Explore` now inherits the session model (Opus/Fable) instead of Haiku (`which-claude-model-should-you-build-with.md` L152); Anthropic's defect-search benchmark cut cost **~33%** using Sonnet 5.5 subagents under Opus 5.5.
   - *Arithmetic:* Moving even 25% of ~325 child agents/wk (~80 search/research children, ~25 turns each = 2,000 requests) from Opus 5.5 ($4/$20/$0.20) to Sonnet 5.5 ($2/$10/$0.10, **50% cheaper**) or Haiku 5.5 (<100K: $0.10/$0.50/$0.01, **95–97.5% cheaper**) cuts those child sessions' cost by **50–95%** (~**$100–$250 API-eq/wk** depending on output volume).

7. **Run `/doctor prompt-audit` on `CLAUDE.md`, skills, and agent definitions + move review nits/caps into `REVIEW.md` and cap review loops at 3 rounds (diff-only after round 1)**
   - *Basis:* Anthropic measured `/doctor prompt-audit` cutting cost by **9%** on Opus 5.5 by removing older-model prompting patterns that trigger extra tool calls (`what-a-task-costs-on-opus-5-5.md` L175). In `solving-code-review-at-anthropic`, ~59% of post-round-1 findings were pre-existing code; capping rounds at 3 and diff-only re-review prevents runaway review loops.
   - *Arithmetic:* A 9% turn reduction on 18,100 requests (~1,630 fewer requests × 239K tokens) saves **~390M prompt tokens/week** plus ~9% of output tokens (~**$80+ reads + ~$150+ output = ~$230+ API-eq/wk**).

8. **Enable `CLAUDE_CODE_DISABLE_AGENT_PROGRESS_SUMMARIES=1` and use native forked subagents (when children need parent context) instead of fresh `delegate_task` threads**
   - *Basis:* Eliminates extra background summarization API calls across 325+ child agents/week (`CHANGELOG.md` L1268), and forked subagents hit the parent's warm prompt cache on turn 1 instead of paying a 70K cold write (`sub-agents.md` L1301).
   - *Arithmetic:* If 100 of 325 weekly children can run as native forks (no cross-provider/worktree need), skipping 100 cold 70K writes saves **7M cache-write tokens/week** (~$35–$112 API-eq/wk), plus eliminating 1–3 progress-summary calls per child cuts **~325–975 background API requests/week**.

---

## 5. What Could Not Be Found or Verified (and Corrections to "What Is Already Known")

1. **Correction to "What is already known" — Delegated child cache TTL in T3 Code:**
   - The prompt noted *"subagents five minutes"* and asked whether T3 copies `subagentPromptCacheTtl` into `CLAUDE_CODE_PROMPT_CACHE_TTL`.
   - **Verified correction:** T3 delegated children run as standalone Agent SDK sessions, so **by default (when `subagentPromptCacheTtl` is unset) they get the 1-hour main-conversation TTL on a subscription, NOT 5 minutes.** T3 copies `subagentPromptCacheTtl` → `CLAUDE_CODE_PROMPT_CACHE_TTL` **only when the user explicitly configures `subagentPromptCacheTtl`** (or `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`) (`ClaudeAdapterV2.ts#L2380-L2405`, `#L6080-L6102`).
2. **Correction to "What is already known" — `skillOverrides` and plugin skills:**
   - `skillOverrides` **does not apply to plugin skills** (`settings-reference.md` L3868). Plugin skills can only be removed from context by disabling the plugin in `enabledPlugins` (at project or local scope).
3. **Pages that failed to load or moved:**
   - `https://claude.com/blog/claude-model-and-effort-level-in-claude-code` returned an HTTP error (`None` / blocked or unavailable via urllib) and was **not** cited.
   - `https://www.anthropic.com/engineering/code-execution-with-mcp` and `https://www.anthropic.com/engineering/advanced-tool-use` failed on `/engineering/`; both loaded cleanly at **`https://www.anthropic.com/research/code-execution-with-mcp`** and **`https://www.anthropic.com/research/advanced-tool-use`**.
4. **Could not find in primary sources:**
   - **Exact subscription weekly-limit token weights** (how cache reads, 5m/1h cache writes, and output tokens map to weekly percentage): Anthropic has not published a formula; only qualitative statements (*"cached and counts less"*) and API overflow rates are documented.
   - **November 10, 2026 general full-limit reset (expiring Dec 10, 2026):** Secondary search summaries attributed this to an `@ClaudeDevs` post on X, but no official `support.claude.com`, `status.claude.com`, or `anthropic.com` page documenting it could be opened directly. Only the **November 9, 2026 Fable 5.2 launch resets** (`support.claude.com/en/articles/17506494`) were verified from an official Anthropic page.
   - **Built-in tool definition token sizes:** No official Claude Code documentation lists per-tool token sizes for built-in tools.
   - **Opus 5.5 numeric table in `spending-your-effort` Fig A:** Embedded JSON in `spending-your-effort` contained full Terminal-Bench 3.0 point arrays for Fable 5.1, Opus 5, and Fable 5, while Opus 5.5's Fig A curve was rendered only as inline SVG coordinates (exact Opus 5.5 effort benchmark numbers were instead verified from `platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence.md`).
   - **Public numeric round-cap setting for Code Review:** Anthropic's blog describes a 3-round cap internally, while the public managed Code Review docs describe diff-only re-reviews and skipping unchanged pushes without exposing a configurable round-cap number.
   - **Official user-facing documentation for tuning/disabling Claude Code's internal `microcompact` tool-result clearing:** Confirmed to exist via `costs.md` L212/218 and `CHANGELOG.md` v2.1.50, but its triggers remain undocumented server-side flags with no user setting (`anthropics/claude-code#42542`).
