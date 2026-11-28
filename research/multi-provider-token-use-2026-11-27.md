# Multi-provider token use, lean instances and shared rules (2026-11-27)

Researched on 2026-11-27 on one Mac to answer whether the lean child-agent pattern used for Claude Code (`claudeLean`)
helps Google Antigravity (Gemini 4 Neon), OpenAI Codex and xAI Grok, and to audit cross-provider global rules and memory
files.

Grok's companion X search is in [multi-provider-x-grok-2026-11-27.md](multi-provider-x-grok-2026-11-27.md).

## 1. Local measurements (measured here, one Mac)

### Claude follow-up: before vs. after the 2026-11-24 settings

Counted from local Claude Code JSONL transcripts (non-Haiku requests, deduplicated by message id and request id):

| Window | Requests | Avg context / request | Context share >200k | Requests >300k | Compactions |
|---|---:|---:|---:|---:|---:|
| Before (2026-11-17 21:00 UTC to 2026-11-24 21:10 UTC) | 20,896 | 227,166 | 69.9% | 4,825 (23.1%) | 25 (prior week) |
| After (2026-11-24 21:10 UTC to 2026-11-27) | 4,550 | 159,053 (-30.0%) | 42.6% | 187 (4.1%) | 22 (in 3 days) |

### Antigravity / Gemini 4 Neon (2026-11-20 to 2026-11-27)

Counted from local Antigravity ACP `transcript.jsonl` files (`PLANNER_RESPONSE` steps, where `input_tokens` records
uncached input and `cache_read_tokens` records cached input, so total context per request is
`input_tokens + cache_read_tokens`). Runtime observed: Antigravity ACP `1.5.15` inside T3 Code nightly
`0.1.13-nightly.20261125.5258`.

- **Volume (2026-11-20 to 2026-11-27):** 4,439 requests across all Antigravity models; 918.9M total context tokens
  (62.6M uncached input, 856.3M cache reads, 93.2% cache hit); 8.70M output tokens (including thinking tokens).
- **Context growth:** average context 206,996 tokens; 70.9% of all context tokens came from requests over 200,000;
  1,046 requests exceeded 300,000; 428 exceeded 400,000; max context observed 511,925 (at Gemini 4 Neon's documented
  512K Antigravity ceiling); 0 compaction steps recorded across 180 sessions.
- **Model and effort mix (2026-11-20 to 2026-11-27):**
  - `gemini-4-neon-xhigh`: 2,048 requests (46%), avg context 272,135, 94.8% cache hit, 4.16M output tokens.
  - `gemini-4-neon-high`: 1,207 requests (27%), avg context 179,497, 92.8% cache hit, 2.59M output tokens.
  - `gemini-4-neon-medium`: 67 requests (1.5%), avg context 66,271, 83.1% cache hit, 108k output tokens.
  - `gemini-4-neon-low`: 3 requests, avg context 39,593.
  - Remainder: `unknown` (657), `gemini-4-carbon-high` (160), `claude-opus-5-5-high` (116), `claude-opus-5-5-medium`
    (101), `gemini-4-carbon-medium` (80).
- **Startup tokens (all 180 sessions):**
  - Fresh T3-launched sessions jumped from median 15,537 (2026-11-18) and 23,788 (early 2026-11-23, after symlinking
    global rules) to **43,249** median (2026-11-23 to 2026-11-27, with T3's full eager MCP tool surface and plugin
    hooks).
  - Native Antigravity subagents (without T3's MCP server or session-start hook) started at **16,361** median.
- **Cache hit rate vs. inter-call gap and context length (3,328 Neon calls, 2026-11-20 to 2026-11-27):**

| Inter-call gap (same session, same Neon model) | Calls | Avg cache hit | Zero-cache misses |
|---|---:|---:|---:|
| `<15s` | 1,409 | 92.9% | 0.2% |
| `15–30s` | 861 | 94.4% | 0.7% |
| `30–60s` | 530 | 93.5% | 1.1% |
| `1–2m` | 223 | 90.1% | 4.9% |
| `2–5m` | 126 | 84.5% | 10.3% |
| `5–15m` | 56 | 91.6% | 5.4% |
| `>15m` | 17 | 40.2% | 58.8% |

| Context size (non-first turns, same Neon model) | Calls | Avg cache hit | Zero-cache misses |
|---|---:|---:|---:|
| `<50k` | 54 | 77.2% | 14.8% |
| `50–100k` | 398 | 82.3% | 3.0% |
| `100–200k` | 879 | 90.2% | 1.4% |
| `200–300k` | 879 | 95.6% | 0.9% |
| `300–512k` | 1,012 | 96.8% | 1.2% |

This contradicts the 1-minute and high-context cache-collapse telemetry reported on X for earlier Antigravity CLI
releases (`1.5.11`–`1.5.12`): on Antigravity ACP `1.5.15`, the cache held ~90–94% up to 15 minutes and 96.8% at
300–512k, dropping sharply only past 15 minutes idle.

### Codex (2026-11-20 to 2026-11-27)

Counted from local Codex JSONL session rollouts (30 sessions total with token counts):

- **Volume (2026-11-20 to 2026-11-27):** 347 requests (290 on `gpt-6.2-astra`, 47 on `gpt-6.2-sol`, 10 on `gpt-6-astra`),
  23.5M input tokens, 21.8M cached (92.6% overall, 94.2% on `gpt-6.2-astra`), 77k output tokens, average context 67,782,
  max context 123,253, 0 sessions >200k.
- **Startup tokens before change:** median 28,564 (last session before change: 28,441), with `<skills_instructions>`
  carrying 27,087 chars (68 `t3-pstack` mentions) plus a 1.85 KB session-start hook.
- **After disabling `t3-pstack` in `~/.codex/config.toml` and adding 4.5 KB of missing global rules (live test,
  2026-11-27):** `<skills_instructions>` dropped from 27,087 chars to 11,474 chars (-15.6 KB; 0 `t3-pstack` mentions) and
  first-turn input dropped from 28,441 to **24,727** tokens (~4,850 net tokens saved after accounting for the added
  rules). The session still read an external `worker.md` file by absolute path cleanly.

### Grok (2026-11-20 to 2026-11-27)

Counted from T3 Code's local usage cache and Grok CLI session logs (`grok 1.0.92`):

- **Volume:** 25 completed turns in the last 7 days (5.55M uncached input, 44.6M cached reads, 751k output); 36 turns
  recorded all-time at 88.5% cache hit.
- **Rules duplication:** `grok inspect` confirmed Grok loads `~/.claude/CLAUDE.md` (~5,172 tokens) via Claude
  compatibility alongside `~/.grok/AGENTS.md` (~3,008 tokens). Replacing `~/.grok/AGENTS.md` with a Grok-only footer cut
  it to ~282 tokens (~2,726 tokens saved per turn), verified by a live Grok session quoting the last heading of
  `~/.claude/CLAUDE.md`.
- **Unused MCP servers:** every Grok session resolved 5 MCP servers (3 inherited from Claude configs/plugins that had 0
  all-time tool calls across all local Grok session logs). Running `grok mcp disable <name>` for those 3 servers wrote
  `disabled_mcp_servers` into `~/.grok/config.toml` without modifying `~/.claude.json`; a live session confirmed only
  the remaining 2 servers started.

## 2. Primary documentation and source checks (opened 2026-11-27)

- **Gemini 4 Neon API pricing (`documented`, [Gemini API pricing](https://ai.google.dev/gemini-api/docs/pricing)):**
  $3.00 / 1M input tokens, $15.00 / 1M output tokens (including thinking tokens), $0.15 / 1M context caching tokens;
  *"Implicit caching only (explicit caching is not supported)."*
- **Antigravity models and thinking levels (`documented`, [Antigravity models](https://antigravity.google/docs/models)):**
  Gemini 4 Neon supports `Low`, `Medium` (default), `High` and `Extra` (`xhigh`), with a context window up to 512K in
  Antigravity. `Orchestrator` pairs a planner model with a faster executor model in one conversation.
- **Antigravity rule size caps (`documented`, [Antigravity rules](https://antigravity.google/docs/rules) and bundled
  customization docs):** 24,000-byte per-file cap (truncated on line boundaries) and 20,000-token aggregate rules budget.
  On this Mac, the symlinked global rules file is 20,688 bytes (86% of the 24,000-byte cap).
- **Antigravity compaction and MCP loading (`documented`,
  [Context windows and tokens](https://antigravity.google/docs/learn/context-and-tokens),
  [Compact command](https://antigravity.google/docs/commands/compact)):** Antigravity supports `/compact [message]` and
  lazy-loads native MCP tools by default, but T3 Code's ACP bridge previously sent all ~74–82 T3 MCP tools eagerly.
- **T3 Code PR #39362 (`source`, [pingdotgg/t3code#39362](https://github.com/pingdotgg/t3code/pull/39362), merged
  2026-11-27):** *"fix(antigravity): defer optional T3 MCP tools via tool_call proxy"*. Notes that T3 previously listed
  ~74–82 MCP tools upfront (~50 KB) on Antigravity ACP and adds `deferred-proxy` mode exposing 5 direct tools
  (`orchestrator_capabilities`, `delegate_task`, `task_status`, `tool_search`, `tool_call`).
- **T3 Code provider settings (`documented` / `source`,
  [providers-claude.md](https://github.com/pingdotgg/t3code/blob/main/docs/user/providers-claude.md),
  [providers-antigravity.md](https://github.com/pingdotgg/t3code/blob/main/docs/user/providers-antigravity.md),
  [providers-codex.md](https://github.com/pingdotgg/t3code/blob/main/docs/user/providers-codex.md),
  [providers-grok.md](https://github.com/pingdotgg/t3code/blob/main/docs/user/providers-grok.md),
  `AntigravityDriver.ts`):**
  - `autoCompactWindow` and `idleCompaction` apply to Claude instances, not Antigravity.
  - Antigravity instances each maintain an isolated profile directory and sign-in, with no `--settings` overlay flag
    comparable to Claude's `claudeLean`.
  - Codex supports `--profile` / `-c` in T3 launch arguments and separate/shadow `CODEX_HOME`.
  - Grok supports `GROK_HOME` per instance and `disabled_mcp_servers` in `~/.grok/config.toml`.

## 3. Conclusions and actions taken (reviewed 2026-11-27)

1. **A separate lean child instance is mostly an Anthropic win.** Keep `claudeLean` for ordinary Claude children (both
   Opus and Fable). Do not create a second Antigravity instance; updating T3 Code to pick up PR #39362 removes the ~50 KB
   eager T3 MCP overhead across every Antigravity session automatically.
2. **Trim Codex and Grok directly instead of cloning instances:**
   - Disabled `t3-pstack` in `~/.codex/config.toml` (for read-only deep review sessions): verified -15.6 KB skills text
     and ~4,850 net startup tokens saved.
   - Replaced `~/.grok/AGENTS.md` with a Grok-only footer (since Grok already loads `~/.claude/CLAUDE.md`): verified
     ~2,726 tokens saved per turn.
   - Disabled 3 unused MCP servers in `~/.grok/config.toml` via `grok mcp disable`: verified live without touching
     `~/.claude.json`.
3. **Sync missing rules to Codex and prune stale Claude memories:**
   - Appended 3 missing sections to `~/.codex/AGENTS.md` and verified full loading on fresh Codex, Grok and Antigravity
     sessions.
   - Removed 4 superseded Claude memory files that contradicted current model/watcher rules (backed up first).
4. **Deferred to the owner:** whether to disable `claude-mem` globally (still actively invoked ~980 times in Claude
   transcripts; ~2.7 GB on disk) and whether to lower the `t3-pstack` model-sheet budget via `/setup-pstack`.
