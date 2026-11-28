# How to spend fewer tokens in T3 Code threads

Researched on 2026-11-24. Three kinds of evidence, in order of weight:

- **measured here**: counted from the Claude Code transcripts on the owner's Mac, 2026-11-17 to 2026-11-24.
- **documented**: Anthropic's or T3 Code's own documentation, opened on 2026-11-24.
- **reported**: what people posted on X (Grok read about 90 posts). Opinion until something else backs it.
- **default**: nobody has measured it. It is our choice.

The raw research is in [`research/`](../research/). The numbers cover Claude models only. Codex, Gemini and Grok run on
their own plans and are not counted.

## Where the tokens go

One week, Opus 5.5 and Fable 5.2 together (measured here):

| What | Number |
|---|---|
| Requests | about 18,100 |
| Tokens of context carried by those requests | 4.3 billion |
| Average context per request | about 239,000 tokens |
| New text returned by tools in the whole week | about 31 million tokens |
| Output written by the models | about 24 million tokens |

Every request re-reads the whole thread so far. So the cost is the number of requests multiplied by the size of the
thread. New tool output is under 1% of the tokens read in the week: what costs is that every later turn reads it again. "A one-line question in a session
that has been open all day still draws usage for the whole conversation" (documented,
[costs](https://code.claude.com/docs/en/costs)).

At list prices the week comes to roughly $2,700, split like this (measured here, priced from the
[price list](https://platform.claude.com/docs/en/about-claude/pricing)):

| Part | Share | What drives it |
|---|---|---|
| Writing context into the cache | about 44% | New text each turn, every thread or agent that starts, and every return to a thread that sat idle |
| Re-reading cached context | about 31% | Thread size multiplied by the number of requests |
| Output, including thinking | about 25% | Effort level and model |

Anthropic does not publish how the subscription weighs these, so treat the shares as a guide. It does say cached content
counts against plan limits at a reduced rate, not zero (documented).

Four findings explain most of it:

1. **Threads grow very large before they shrink.** 139 of 337 sessions passed 400,000 tokens, and two thirds of the
   usage came from requests carrying more than 200,000. Threads were compacted 25 times in the week: 17 by hand, at a
   median of 416,000 tokens, and 8 automatically, at a median of 959,000 (three of the eight at 200,000 to 240,000). The reason: these models have a 1 million token
   window, and Claude Code by default compacts at about 967,000 tokens (documented,
   [model config](https://code.claude.com/docs/en/model-config)). T3 Code's "Auto-compact after" setting is empty on
   this Mac, so nothing compacts a thread earlier unless someone does it by hand.
2. **Every thread and every delegated child thread starts with about 70,000 tokens** before anyone types. That is 29%
   of all the context re-read in the week. About 325 Claude child agents were started in the week, most of them
   delegated threads, and each paid for most of its start at the cache-write price (on the one fresh thread examined,
   42,000 of 68,000 tokens were written and the rest were read from cache). A subagent started with Claude
   Code's own Agent tool gets a smaller prompt (documented), but still starts cold.
3. **Fable costs twice what Opus does** for output and for cache writes, and the same for cache reads (documented, price
   list). Fable made a third of the requests and a little over half of the cost.
4. **Coming back to a thread after its cache has expired** re-writes the whole thread. That was 5.3% of the cost with the simpler weights, and an estimated 9% at list prices,
   where a cache write costs more. T3 Code's "Compact idle threads" is set to "When you return" on this Mac, which offers
   a Compact button only after the cache has already gone.

What is in the 70,000-token start, measured on one fresh thread (approximate):

| Piece | Tokens |
|---|---|
| Claude Code's own system prompt and tool definitions | about 37,000 |
| List of skills (over 200 of them) | about 10,700 |
| Names of tools that load on demand | about 7,300 |
| Global rules file, `~/.claude/CLAUDE.md` | about 4,200 |
| Hook output at session start (superpowers, pstack, claude-mem) | about 3,500 |
| Instructions from connected servers | about 3,000 |
| List of agent types | about 1,800 |

Only 7 plugins are installed on this Mac. Most of the skills and tool names come from plugin sets and connectors synced
from the claude.ai account: sales, legal, marketing, small-business, design, cargo, Gmail, Calendar, Drive and others.

## What to change, in order

Savings are estimates from the measured requests, not promises. They overlap, so they do not add up.

### Settings (the owner decides; each is one switch and can be undone)

| # | Change | Where | Expected saving | What you give up |
|---|---|---|---|---|
| 1 | Compact threads earlier: set **Auto-compact after** to `300000` | T3 Code, Settings, Providers, Claude | 5 to 15%. At `200000`: 10 to 25%. Several users on X settle on `400000`, which saves less (not estimated here). | A compacted thread keeps a summary, not every detail. |
| 2 | Compact idle threads **before the cache expires** (beta) | T3 Code, Settings, Providers, Advanced | Most of the 5 to 9% spent on returning to idle threads | A thread you never return to is still compacted once. |
| 3 | Stop loading plugin sets and connectors you do not use for coding | `~/.claude/settings.json`: `syncClaudeAiPlugins: false`, `syncClaudeAiSkills: false`, `disableClaudeAiConnectors: true`, or `"<name>@<marketplace>": false` under `enabledPlugins` for a single plugin | About 5 to 9% if the start drops by 15,000 to 25,000 tokens | Those skills and connectors (Gmail, Drive and so on) are no longer available inside threads. They still work on claude.ai. |
| 4 | Run threads at **medium** effort unless the task needs more | The model picker in T3 Code | Part of the 25% that is output. Anthropic's own tests: Opus 5.5 at high costs $0.29 a task against $0.21 at medium, and Fable 5.2 at high costs 2.2 times medium. | Less thinking per step. On Anthropic's coding test, medium solved 92.8% and high 95.3%. Medium is the documented default for Opus 5.5 and Fable 5.2. |
| 5 | Start long owner-facing threads on **Opus**, and keep Fable for advice and the sensitive code. Do not switch a thread that is already running. | The model picker in T3 Code | About a third of what those threads cost | Fable's judgment on every turn of the thread. |

Check setting 3 with `/context` in a fresh thread before and after. `/usage` shows which plugin, skill or agent used the
most (documented).

In each range for settings 1 and 2, the lower number uses today's list prices and the higher number uses the simpler
weights the measurement started with (input 1, cache write 1.25, cache read 0.1, output 5). All the names in this table
were checked against the documentation on 2026-11-24 (documented:
[T3 Code Claude provider](https://github.com/pingdotgg/t3code/blob/main/docs/user/providers-claude.md),
[settings reference](https://code.claude.com/docs/en/settings-reference)).

### Habits (what every thread does)

1. **One PR (or task), one thread.** When a thread has already opened or finished a PR or task, start the next PR in a
   new thread with a short handoff note (goal, files, branch, decisions) rather than reusing the old context. Stay in
   the same thread for opening the PR for work built there, CI fixes and review rounds on that PR, work needing that
   checkout's uncommitted edits, or a tiny warm follow-up under ~100k tokens. Anthropic's first cost recommendation is
   to clear between unrelated tasks (documented, costs). Threads of 500 to 900 requests are where the cost is.
2. **Fewer requests.** Send independent tool calls together in one turn, and chain shell checks in one command. Each
   extra turn re-reads the whole thread. The researcher reports that Anthropic's parallel tool use page says Fable
   models batch less in long loops unless asked to (not re-checked).
3. **Start a child agent only when it earns its start.** Each one costs about 70,000 tokens before it does anything.
   Anthropic now lists "use subagents to protect your context window" under things to stop doing: "Running out of
   context is rarely the problem anymore" (documented, best practices). Use one for parallel work, noisy output or an
   independent review, and give it the file paths and the question so it does not re-read the repo.
4. **Stop a review loop after 3 rounds** and ask the owner. Loops here ran 4 to 9 rounds per pull request, and each
   round starts new agents and wakes a large parent thread. Anthropic had the same problem with its own review tool:
   one pull request "went through more than fifty rounds of review and fixes overnight", and about 59% of what the
   reviewer kept flagging was code that had been there since its first pass. Its fixes: after three rounds the reviewer
   asks the author to reconsider the approach, later rounds review only the new diff, no finding is repeated, and the
   reviewer flags the class of bug so every instance is fixed at once. Pull requests needing five or more rounds fell
   from 8.9% to 1.5%, and the average number of reviews to pass fell from 2.08 to 1.32 (measured by Anthropic,
   [Solving code review at Anthropic](https://claude.dev/blog/solving-code-review-at-anthropic/), 2026-10-15).
5. **Do not wake a large idle thread for a small message.** The first request after the cache expires re-writes the
   whole thread (documented, [prompt caching](https://code.claude.com/docs/en/prompt-caching)). The cache lasts about an
   hour on a subscription.
6. **Orchestrate from a small thread** (default). Each child that finishes wakes its parent, and the parent re-reads
   everything it holds. A parent at 400,000 tokens pays that on every wake.
7. **Read part of a file when part is enough,** and keep long output such as logs and test runs in a file, not in the
   thread. Anthropic's cost page recommends filtering noisy output (documented, costs).

### What is not worth the effort

- **Fine-tuning small tool outputs.** New tool output is under 1% of what gets read (measured here). Keep the large
  ones out (habit 7) and leave the rest alone.
- **Switching models inside a thread to save money.** A model switch rebuilds the cache (documented, prompt caching).
  Changing effort does not, on the current models.
- **The memory plugin's note-taker.** It runs on Haiku: about 1,900 requests in the week, around 1% of the cost at list
  prices (measured here).
- **Shortening the cache lifetime for child agents.** Measured here in the third round (see below): it would have cost
  more, not less.

## Added after a second round of research

A second round on X dug into the leads above and looked for new ones. What changed or was added:

- **The review-loop cap is Anthropic's own practice, with numbers** (habit 4, now rewritten). A short `REVIEW.md` kept
  with the code says what failure looks like in that repo, so the reviewer stops raising style points (documented, same
  write-up).
- **Agents that start together do not share a cache, unless they are forks or a workflow fan-out.** A subagent's first
  request "doesn't read the parent's cache". A fork inherits the parent's context and does. In a workflow, Claude Code
  holds every agent after the first for up to 5 seconds so the rest can read the first one's cache (documented, prompt
  caching). Separate delegated threads get neither, so each pays for its own start.
- **A child finishing does not break the parent's cache.** Its result is appended to the parent (documented, prompt
  caching). The cost of a wake is the parent's size, not a rebuild, unless the parent sat idle past its cache lifetime.
- **Subagents keep a 5-minute cache even on a subscription** (documented). One that waits longer than that on a build
  or on another agent re-writes its whole context when it continues. One user found about 1% of pauses caused nearly
  half of his subagent cache writes, and another cut usage "by at least 30%" by having the coordinator start background
  threads in place of idle subagents (reported on X, 2026-10-10).
- **Tools that find the waste for you:** `/usage` names the skill, plugin, subagent or server using the most and can
  apply a fix with `f`. `/doctor` compares unused skills, servers and plugins with what they cost in context
  (documented). The researcher also reports `/skill-doctor`, and that `skillOverrides` takes `on`, `name-only`,
  `user-invocable-only` or `off` but does not control skills that come from plugins (documentation, read only in
  summary).
- **Another team's week looks like this one.** Of $2,495 at list prices, $532 was cache rebuilt after idle breaks and
  $360 was history resent past 200,000 tokens, against $68 for failed tool calls (reported on X by @dannygnc,
  2026-11-22).
- **A free reset may be waiting.** Anthropic said on 2026-11-10 that bugs had used more of people's limits than
  intended, and gave Pro, Max, Ultra and Team accounts one full reset of all limits, to be used before 2026-12-10 in
  Settings, Usage (reported on X by @ClaudeDevs). Not checked on the owner's account.

Looked for and not found: anyone else's before and after for the compaction settings, an independent measurement of
what the superpowers or memory plugins cost in context, and any published formula for how the weekly limit weighs cache
reads, cache writes and output.

## Added after a third round of research

Gemini 4 Neon read T3 Code's source and the official documentation to close the open questions. The source lines,
Anthropic's effort table and the support pages were then opened again to check them, and two more things were measured
on the owner's Mac.

### How T3 Code starts a delegated child thread

- **A child gets everything a normal thread gets.** T3 Code passes the same system prompt, tools, plugins, skills,
  hooks and rules files to a delegated child as to a thread you type in. Only two things differ: prompt suggestions are
  off, and the cache lifetime can be set separately (source, T3 Code `ClaudeAdapterV2.ts` at commit 3380585, as read by
  the researcher). This is why each child starts at about 70,000 tokens.
- **A child keeps the one-hour cache.** The source says: "A delegated child is its own SDK session, so Claude Code
  gives it the main-conversation cache TTL." It follows `subagentPromptCacheTtl` only when that setting is present
  (source, checked). The setting is absent on this Mac, and 75.6 million of the week's 76 million cache-write tokens
  were written at the one-hour rate, which costs 2 times base input against 1.25 times for five minutes (measured
  here).
- **Do not switch children to the five-minute cache.** The researcher recommended it. Replaying the week says no: for
  sessions with a single prompt, which stand in for delegated children, a five-minute cache would have cost about 12%
  more on the prompt side ($709 against $636 at list prices), because 117 requests came after a pause of 5 to 60
  minutes, usually a build or a wait on another agent, and each would have re-written its whole context. For threads
  you type in, it would nearly double the cost (measured here; single-prompt sessions are a stand-in, not an exact
  match for child threads).
- **"Auto-compact after" applies to child threads too** (source, as read by the researcher). **"Compact idle threads"
  does not:** it skips delegated children (source, checked). Where it does apply, it fires two minutes before the cache
  expires, for threads of 200,000 tokens or more.
- **Children can run on a leaner setup.** T3 Code supports a second Claude provider instance with its own configuration
  folder, environment variables and launch arguments ("Use a separate Claude config directory for each account. This
  also works for named presets", documented, T3 Code). `delegate_task` can target an instance by its id (source, as
  read by the researcher). A lean instance with no synced plugin sets could start 25,000 to 30,000 tokens lighter. The
  price: a separate configuration folder does not load the owner's global rules file or plugins unless they are copied
  in, and it needs its own sign-in. Worth a trial, not a quick switch.

### What each installed plugin adds to every turn

From `claude plugin details <name>`, Claude Code's own estimate (measured here, 2026-11-24):

| Plugin | Added to every session |
|---|---|
| t3-pstack | about 5,260 tokens |
| claude-mem | about 2,890 |
| superpowers | about 850, plus about 2,500 that its session-start hook prints |
| obsidian | about 600 |
| frontend-design | about 80 |
| context7, swift-lsp | 0 |

The seven installed plugins come to about 12,000 tokens with the hook output. `skillOverrides` cannot hide a plugin's
skills. Only turning the plugin off does, under `enabledPlugins`, which can be set for one project (documentation, as
read by the researcher).

### Effort, with Anthropic's numbers

| Model and test | Effort | Result | Cost per task |
|---|---|---|---|
| Opus 5.5, coding (SWE-bench Pro subset) | low | 87.4% solved | $0.11 |
| | medium (default) | 92.8% | $0.21 |
| | high | 95.3% | $0.29 |
| | xhigh | 96.7% | $0.74 |
| Fable 5.2, research (DeepResearch Bench II) | low | 54.5% | $1.49 |
| | medium | 55.0% | $2.16 |
| | high | 57.7% | $4.82 |

Running Opus 5.5 at low and re-running only the failures at high solved about 97% for about $0.17 a task, better and
cheaper than everything at high (documented and measured by Anthropic,
[Optimizing for cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence), checked).
A child agent with no effort of its own runs at the level picked for the session (documented, as read by the
researcher). The thread that produced this guide ran at high.

### Limits on the Ultra plan

- **Fable is capped at half the weekly limit.** "You can use up to 50% of your weekly usage limits on Fable models", and
  Fable models "use them faster than other Claude models" (documented, Anthropic support, checked). Fable was a little
  over half of this Mac's cost at list prices, so the Fable cap is probably the first limit reached. That makes setting
  5 (start long threads on Opus) worth more than its list-price saving suggests.
- **Fable resets are available.** Max and Ultra accounts that were paying before 2026-11-09 get one to three Fable
  limit resets, depending on how long they have subscribed. A reset restores the weekly usage spent on Fable. They
  expire on 2027-03-08. Claim one at Settings, Usage, Limit resets, "Reset for free", on the web, desktop or mobile
  app. It cannot be claimed from Claude Code, but the restored usage applies there (documented, Anthropic support,
  checked).
- **The general reset reported on X (2026-11-10) has no official page** that the researcher could find.
- **The weights are still unpublished.** No official page says how cache reads, cache writes and output count toward
  the weekly limit.

### Smaller points

- T3 Code itself adds about 1,200 to 1,500 tokens of instructions and the names of its 92 tools, which load on demand
  (source, as read by the researcher).
- T3 Code has a usage view (`mod+u`) and can resume threads by itself after a limit resets (documented, T3 Code, as
  read by the researcher).
- Anthropic reports that `/doctor prompt-audit`, which removes instructions written for older models, cut cost by 9% on
  Opus 5.5 (as read by the researcher, not re-checked).
- Changing something at the front of the prompt mid-thread is expensive: on a 100,000-token conversation Anthropic
  measured up to $1.26 for editing the system prompt against $0.026 for putting the same update in a message (as read
  by the researcher, not re-checked).

## Trial: a lean setup for child agents (2026-11-24)

**Result: a child on the lean setup started 20,600 tokens lighter, 50,600 against 71,200, and lost nothing it needs.**

How it is set up, with no second sign-in and no second configuration folder:

- An overlay file, `~/.claude/lean-child-settings.json`, sets `syncClaudeAiPlugins` and `syncClaudeAiSkills` to
  `false`, `disableClaudeAiConnectors` to `true`, and turns five plugins off under `enabledPlugins` (t3-pstack,
  claude-mem, superpowers, obsidian, frontend-design). The normal settings file is not changed.
- A second Claude instance in T3 Code, "Claude · Lean (child agents)", with id `claudeLean`, has one launch argument:
  `--settings ~/.claude/lean-child-settings.json`. A delegate uses it only when its target names that instance.

The same six-question read-only task went to both instances, on Opus 5.5 at medium effort (measured here):

| | Default instance | Lean instance |
|---|---|---|
| Tokens in the first request | 71,194 | 50,579 |
| Skills listed | about 255, from 19 plugin sets | about 18, none from a plugin |
| Session-start messages from plugins | 3 (t3-pstack, superpowers, claude-mem) | none |
| claude.ai connectors (Gmail, Drive and others) | listed | not listed |
| Global rules file loaded | yes | yes |
| Test-worker cap (`VITEST_MAX_WORKERS`) | 2 | 2 |
| T3 Code's own tools, such as `delegate_task` | available | available |
| Cost of the whole two-request task at list prices | about $0.36 | about $0.22 |

What a lean child gives up: the pstack and superpowers skills, memory capture by claude-mem, the Obsidian and
frontend-design skills, and the claude.ai connectors and plugin sets. A child that needs one of those should run on the
default instance. pstack runs stay on the default instance.

What it is worth: about 20,600 fewer tokens on every request a child makes, and about 16,000 fewer written when it
starts. If every Claude child used it, that is roughly 2 to 4% of the measured week at list prices. It is a small,
safe saving, and smaller than compacting earlier or moving long threads off Fable.

To remove it: delete the `claudeLean` entry in T3 Code's Settings, Providers (or restore
`~/.t3/userdata/settings.before-claude-lean.json`), and delete the overlay file.

## What was applied (2026-11-24)

The owner asked for all five settings. Four are in place and one is his to pick.

| # | Setting | Status | Where it lives |
|---|---|---|---|
| 1 | Compact threads at 300,000 tokens | Applied | `autoCompactWindow: 300000` in `~/.claude/settings.json` |
| 2 | Compact idle threads before the cache expires | Applied | `idleCompaction: "beforeCacheExpires"` in T3 Code's settings |
| 3 | Stop loading claude.ai plugin sets and connectors | Applied and measured | `syncClaudeAiPlugins: false`, `syncClaudeAiSkills: false`, `disableClaudeAiConnectors: true` in `~/.claude/settings.json` |
| 4 | Medium effort by default | Applied for Opus | The saved level for Opus 5.5 in `modelSettings` went from `high` to `medium`. In T3 Code a thread's own effort still wins. |
| 5 | Start long threads on Opus | Not set | T3 Code's project default did not accept the value. Pick Opus in the model picker. |

- **Setting 3, measured:** a new session started at 60,436 tokens before and 51,673 after, which is 8,763 fewer on
  every request (measured here, a one-request Opus session outside T3 Code).
- **What setting 3 removes from coding threads:** the claude.ai connectors (Gmail, Calendar, Drive, Docs, Sheets,
  Slides, Netlify, Claude Docs), the plugin sets synced from the account (small-business with its Shopify tools, sales,
  marketing, legal, design, engineering, cargo and others), and the synced skills for Word, PDF, PowerPoint and
  spreadsheet files. They still work on claude.ai. Whether one set can be kept while the rest stay off was not tested.
- **Settings 1 and 2 are set but their effect has not been observed yet.** Any thread already past 300,000 tokens
  summarizes on its next message.
- **The lean instance is now the default for ordinary Claude child agents,** by a rule in the owner's global rules
  files. The gap in the trial above was measured before setting 3. With setting 3 on, a default session starts at
  51,673 and a lean one at 42,374, so the lean instance now saves about 9,300 tokens per request, not 20,600.
- **To undo:** restore `~/.claude/settings.before-usage-savings-2026-11-24.json` for settings 1, 3 and 4, and
  `~/.t3/userdata/settings.before-idle-compaction.json` for setting 2.

## Follow-up and other providers (2026-11-27)

Three days after the 2026-11-24 Claude settings were applied, Claude was re-measured and Antigravity (Gemini 4 Neon),
Codex and Grok were measured for the first time (all local figures: **measured here**, one Mac; raw notes in
[`research/multi-provider-token-use-2026-11-27.md`](../research/multi-provider-token-use-2026-11-27.md) and
[`research/multi-provider-x-grok-2026-11-27.md`](../research/multi-provider-x-grok-2026-11-27.md)).

### Claude after three days (2026-11-24 to 2026-11-27)

Non-Haiku requests, same deduplication method as above (measured here, one Mac):

| Window | Requests | Avg context | Share from >200k | Requests >300k | Compactions |
|---|---:|---:|---:|---:|---:|
| Before (2026-11-17 to 2026-11-24) | 20,896 | 227,166 | 69.9% | 4,825 (23.1%) | 25 |
| After (2026-11-24 to 2026-11-27) | 4,550 | 159,053 (-30%) | 42.6% | 187 (4.1%) | 22 |

### Are lean child instances useful outside Anthropic?

**Mostly no.** A separate lean child instance (`claudeLean`) is uniquely practical on Claude Code because T3 Code lets a
second Claude instance pass `--settings` without a second login, and Claude carries a 2x 1-hour cache-write fee plus
several heavy plugins. On the other three harnesses, trimming the single configuration directly works better:

| Provider (2026-11-20 to 2026-11-27, one Mac) | Requests | Avg context | Fresh startup (median) | Cache hit | Best move |
|---|---:|---:|---:|---:|---|
| Antigravity (Gemini 4 Neon & others) | 4,439 | 206,996 | 43,249 (16,361 native subagent) | 93.2% | Do not make a second instance; update T3 Code for lazy MCP tools, default to `medium`/`high` instead of `xhigh`, and compact manually around 200k–250k. |
| Codex (`gpt-6.2-astra` / `sol`) | 347 | 67,782 | 28,564 → **24,727** after change | 92.6% | Disable unused plugins directly in `~/.codex/config.toml` instead of cloning an instance. |
| Grok (`grok-4.7`) | 25 turns | — | — | 88.5% | Keep `~/.grok/AGENTS.md` as a short Grok-only footer and disable unused inherited MCP servers in `~/.grok/config.toml`. |

### What actually drives Antigravity / Gemini 4 Neon usage

- **Pricing and caching mechanics (`documented`, [Gemini API pricing](https://ai.google.dev/gemini-api/docs/pricing)):**
  Gemini 4 Neon lists at $3.00 / 1M input tokens, $15.00 / 1M output tokens (including thinking tokens), and $0.15 / 1M
  cached input tokens (95% discount, implicit caching only; explicit caching is not supported). Thinking tokens cost 100x
  cached input.
- **Effort dominates output spend (measured here, 2026-11-20 to 2026-11-27):** 46% of all Antigravity requests (2,048)
  ran on `gemini-4-neon-xhigh` (averaging 272,135 tokens of context) and 27% (1,207) on `high`. Only 1.5% (67) ran on
  `medium`, which is Google's documented default ([Antigravity models](https://antigravity.google/docs/models)).
- **No automatic early compaction (measured here / documented):** 70.9% of Antigravity context came from requests over
  200,000 tokens, 1,046 requests passed 300,000, and sessions reached 511,925 (Neon's documented 512K Antigravity
  window) with 0 compactions. T3 Code's `autoCompactWindow` and `idleCompaction` are Claude-only
  ([providers-claude.md](https://github.com/pingdotgg/t3code/blob/main/docs/user/providers-claude.md)). Use `/compact`
  manually or hand off around 200,000–250,000 tokens ([Compact command](https://antigravity.google/docs/commands/compact)).
- **Eager T3 MCP tools inflated startup, now fixed upstream (`source`):** T3 Code previously listed ~74–82 MCP tools
  upfront (~50 KB) on Antigravity ACP, which pushed fresh T3 sessions from ~23.8k to ~43.2k tokens (measured here).
  [T3 Code PR #39362](https://github.com/pingdotgg/t3code/pull/39362) (merged 2026-11-27) adds a `deferred-proxy` mode
  exposing 5 direct tools (`orchestrator_capabilities`, `delegate_task`, `task_status`, `tool_search`, `tool_call`).
  Updating T3 Code applies that cut to every Antigravity thread without a second instance.
- **The cache cliff on this Mac is past 15 minutes idle, not 1 minute (measured here):** viral X reports from earlier
  CLI builds (`1.5.11`–`1.5.12`) claimed Neon's cache fell to ~9% after 1 minute and missed 65% at 300–500k (reported on
  X). Across 3,328 Neon calls here on Antigravity ACP `1.5.15`, cache hit stayed **90–94%** up through 15 minutes
  (91.6% at 5–15 minutes) and **96.8%** at 300–512k, falling sharply only **past 15 minutes idle** (40.2% hit, 58.8%
  zero-cache misses). Avoid waking a large Neon thread after more than 15 minutes idle.
- **Global rules file size cap (`documented`):** Antigravity truncates any rule file over 24,000 bytes on line
  boundaries ([Antigravity rules](https://antigravity.google/docs/rules)). Keep a symlinked global rules file safely
  under 24 KB (20,688 bytes here after 2026-11-27).

### What was applied on 2026-11-27 (and what was deferred)

Applied after an independent review and verified with live sessions on Codex, Grok and Antigravity (measured here, one
Mac):

- **Codex:** turned `t3-pstack` off in `~/.codex/config.toml` (read-only review sessions do not run pstack, and absolute
  `worker.md` paths outside Codex remain readable). `<skills_instructions>` fell from 27.1 KB to 11.5 KB (-15.6 KB), and
  first-turn input dropped from 28,441 to 24,727 tokens even after syncing 4.5 KB of missing global rules (~4,850 net
  tokens saved).
- **Grok rules:** `grok inspect` showed Grok already loads `~/.claude/CLAUDE.md` (~5,172 tokens) via Claude
  compatibility alongside `~/.grok/AGENTS.md` (~3,008 tokens). Replacing `~/.grok/AGENTS.md` with a Grok-only footer cut
  it to ~282 tokens (~2,726 saved per turn) while keeping full global rules active.
- **Grok MCPs:** disabled three unused local/plugin MCP servers (0 all-time calls) via `grok mcp disable <name>`
  (`disabled_mcp_servers` in `~/.grok/config.toml`), leaving Claude's own MCP config untouched.
- **Global rules and memory hygiene:** clarified that `claudeLean` covers Fable children as well as Opus; synced three
  missing sections into `~/.codex/AGENTS.md`; deleted four superseded Claude memory files that contradicted current
  rules (backups kept locally).
- **Deferred to the owner:** whether to disable `claude-mem` globally (still actively called ~980 times in transcripts;
  ~2.7 GB on disk) and whether to lower the `t3-pstack` model-sheet budget via `/setup-pstack`.

### New thread per PR: break-even and local replay (2026-11-27)

Verified independently by Opus 5.5 against local transcripts (2026-11-20 to 2026-11-27, one Mac) and by Grok on X (raw
notes in [`research/new-thread-per-pr-opus-2026-11-27.md`](../research/new-thread-per-pr-opus-2026-11-27.md) and
[`research/new-thread-per-pr-x-grok-2026-11-27.md`](../research/new-thread-per-pr-x-grok-2026-11-27.md)):

- **Pricing correction (`documented`, [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)):**
  cache hits & refreshes cost **0.05x** base input on Opus 5.5 ($0.20 vs. $4.00/MTok) and **0.025x** on Fable 5.2
  ($0.20 vs. $8.00/MTok), not 0.1x.
- **Measured per-PR shape (measured here, 2026-11-20 to 2026-11-27, one Mac):**
  - Claude (324 sessions, 16,624 requests): median **39** requests to open a PR, median **258k** context carried into
    the turn after a thread's first PR, median **6.5** follow-up requests (CI fixes/reviews) on that PR.
  - Antigravity / Neon (147 sessions): median **24** requests per PR, median **231k** context carried after a PR.
- **Break-even turn count (fresh start + up to 30k re-read files vs. carrying prior context `C_old`):**

| Prior thread context (`C_old`) | Opus 5.5 (after Nov 24 ~55k start) | Gemini 4 Neon (~43k start) |
|---|---:|---:|
| 100k | 21–48 requests | 15–26 requests |
| 150k | 10–23 requests | 8–14 requests |
| 250k | 5–11 requests | 4–7 requests |
| 350k | 3–7 requests | 3–5 requests |

- **Replay of the 12 later-PR segments (measured here):** reusing an old thread for a second or later PR cost **$53 in
  extra cached context plus $18 in uncached advisor calls** ($71 total at list prices; advisor calls send the full
  thread to Fable uncached, totaling 511 calls / 101M tokens across the week) against **$3 to $8** for fresh starts —
  10–20x cheaper per PR (~2.6% of total weekly input cost because only 3 of 324 Claude sessions and 1 of 147 Antigravity
  sessions carried more than one PR).
- **Rule applied on 2026-11-27:** added `## A new PR gets a new thread` to `~/.claude/CLAUDE.md` and
  `~/.codex/AGENTS.md` (propagated automatically to Antigravity via symlink and Grok via Claude compatibility), keeping
  `~/.claude/CLAUDE.md` under 22 KB. Warm active top-level threads were notified directly; cold idle threads were left
  sleeping so they do not pay a 2x cache rebuild and will pick up the rule on their next compaction or restart.

## Checked and not confirmed

- Posts on X disagree about whether T3 Code compacts by default. On this Mac it does, but the automatic point is the
  default of about 967,000 tokens. The earlier compactions were done by hand.
- Fable usage is its own slice of the weekly limit, according to Anthropic staff on X (@lydiahallie and @bcherny,
  2026-11-09). The size of each slice is not published.

## How to re-measure

The transcripts are in `~/.claude/projects/`, one JSON line per message. Each assistant line has `message.usage` with
`input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens` and `output_tokens`. Count each request once
(the same `message.id` and `requestId` repeat while a reply streams). Re-run after changing a setting and compare the
average context per request and the share of usage that comes from requests over 200,000 tokens.
