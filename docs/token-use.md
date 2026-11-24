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
| 4 | Run threads at **medium** effort unless the task needs more | The model picker in T3 Code | Part of the 25% that is output | Less thinking per step. Medium is the documented default for Opus 5.5 and Fable 5.2. |
| 5 | Start long owner-facing threads on **Opus**, and keep Fable for advice and the sensitive code. Do not switch a thread that is already running. | The model picker in T3 Code | About a third of what those threads cost | Fable's judgment on every turn of the thread. |

Check setting 3 with `/context` in a fresh thread before and after. `/usage` shows which plugin, skill or agent used the
most (documented).

In each range for settings 1 and 2, the lower number uses today's list prices and the higher number uses the simpler
weights the measurement started with (input 1, cache write 1.25, cache read 0.1, output 5). All the names in this table
were checked against the documentation on 2026-11-24 (documented:
[T3 Code Claude provider](https://github.com/pingdotgg/t3code/blob/main/docs/user/providers-claude.md),
[settings reference](https://code.claude.com/docs/en/settings-reference)).

### Habits (what every thread does)

1. **One task, one thread.** When a task is done, or a thread passes about 200,000 tokens (default), write a short
   handoff note (done, in progress, left to do) and continue in a new thread. Anthropic's first cost recommendation is to clear
   between unrelated tasks (documented, costs). Threads of 500 to 900 requests are where the cost is.
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
- **Changing the cache lifetime for child agents.** People who measured it got mixed results (reported). Measure first.

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

## Checked and not confirmed

- A researcher read in T3 Code's source that a delegated child thread gets the one-hour cache lifetime unless
  `subagentPromptCacheTtl` is set. T3's documentation does not say this. Not acted on.
- Posts on X disagree about whether T3 Code compacts by default. On this Mac it does, but the automatic point is the
  default of about 967,000 tokens. The earlier compactions were done by hand.
- Fable usage is its own slice of the weekly limit, according to Anthropic staff on X (@lydiahallie and @bcherny,
  2026-11-09). The size of each slice is not published.

## How to re-measure

The transcripts are in `~/.claude/projects/`, one JSON line per message. Each assistant line has `message.usage` with
`input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens` and `output_tokens`. Count each request once
(the same `message.id` and `requestId` repeat while a reply streams). Re-run after changing a setting and compare the
average context per request and the share of usage that comes from requests over 200,000 tokens.
