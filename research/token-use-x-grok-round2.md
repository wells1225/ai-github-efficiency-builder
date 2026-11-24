# Cutting token use in agent threads: second round on X

Raw research result, saved as returned on 2026-11-24. Researcher: grok-4.7. Not edited. This round dug into the leads from round one and searched topics round one did not cover. Claims found only on X are opinion until another source backs them.

---

X search worked. I read about 70 posts and 9 linked pages (the review write-up, two GitHub repos, one gist, and the current Claude Code docs for costs, prompt caching, skills, and the changelog).

## Part 1

### 1. T3 Code and compaction — holds with limits

The limit is the "20% more usage a week" and the claim that Claude Code in T3 has no auto compaction. Both are one person's say-so. The settings he names are real, and several people use a smaller window for a different reason: recall near 1M.

@lucastohdev (23 Nov 2026) wrote that Claude Code in T3 "defaults to 1M context and no auto compaction", Codex to 256k, and that `CLAUDE_CODE_AUTO_COMPACT_WINDOW` of `300000` plus T3's "Providers -> Advanced -> Compact Idle Thread" gave "up to 20% more Claude usage a week" and "0 waiting on compaction". https://x.com/lucastohdev/status/2124809609068327249

@atarutin (24 Nov 2026) replied "Misinformation. Autocompactiom works out of the box in t3code for both: Claude and Codex." https://x.com/atarutin/status/2125119222397161510

Lucas answered the same day: "Yea ofc it does? I'm saying that depending on your usage setting the number to autocompact something else it will be better for your usage." In a later reply he narrowed "no autocompaction" to compaction that runs before the cache expires, "so that when you go back you dont need to wait". https://x.com/lucastohdev/status/2125120097698402399 and https://x.com/lucastohdev/status/2125142015520424060

No reply in that thread posted a before-and-after log. I found no post from @theo repeating the 20% or the "no auto compaction" claim. His posts that week are about who uses T3 (below), not about this setting.

What other people actually set:

- @OmiidSayfun (17 Nov 2026), with a screenshot, says T3's "Compact idle threads" compacts Claude threads over 200,000 tokens before the cache expires. He also says T3 was 91% of his logged Claude usage over 7 days. That is a share of his usage, not a saving. https://x.com/OmiidSayfun/status/2122577732253188098
- @AdamHoltererer (23 Nov 2026) asks T3 to add Antigravity to "Compact idle threads before cache expiry", which is the feature name heavy users are using. https://x.com/AdamHoltererer/status/2124846737760157897
- @nimi_101 (23 Nov 2026) says the default compact is about 960k and to set `CLAUDE_CODE_AUTO_COMPACT_WINDOW=400000`, because otherwise "claude code will keep on hallucinating". Tradeoff he names: it compacts more often. https://x.com/nimi_101/status/2124798618104135913
- @notlikeryo_ (23 Nov 2026) says the same ~967k default, recommends `/autocompact` or `"autoCompactWindow": 400000`, and credits @nateherk. I did not open Nate's post. https://x.com/notlikeryo_/status/2124929981008551983
- @LeMastellini (24 Nov 2026) lists `/autocompact 400k`, `"autoCompactWindow": 400000`, and `claude --autocompact 400k`. No result attached. https://x.com/LeMastellini/status/2125217482088550427

The current docs I opened say a custom gateway that stops at 200k should use `/autocompact 200k`, and that plan-mode compaction uses the model that is answering, "so it can cost more". https://code.claude.com/docs/en/changelog

Quality when you compact earlier: @lbzepoqo (24 Nov 2026) compacts at task boundaries instead of a fixed count, plugin at https://github.com/lbzepoqo/compact-timing (opened). Defaults there are `boundaryTokens` 200000 and `softTokens` 300000, and the README says to keep Claude Code's own `autoCompactWindow` above `softTokens`. A `busy` mark refuses the plugin's compaction mid-step. In three weeks of his sessions at 120k/200k, "Claude Code's hard-limit compaction never ran and the plugin compacted 251 times. 37% of those ran at 200k or more because a long turn kept growing." Pointer mode replaces the summary with a handoff: goal, what is done, next step, open decisions, the user's own requests, git status. He also notes the plugin's `/compact` works in T3 and `claude -p`.

Official cost page: add compact instructions with `/compact Focus on code samples and API usage`, or a "Compact instructions" section in `CLAUDE.md`. https://code.claude.com/docs/en/costs

### 2. Review loops — holds

The cap, the 50-round PR, and 8.9% to 1.5% are in Anthropic's own write-up, not only in the tweets.

@ClaudeDevs (16 Oct 2026) pointed at the post. https://x.com/ClaudeDevs/status/2111168901723906176

@davekiss (16 Oct 2026): "Cap your agent review loops at 3 rounds… one Claude Code PR ran 50+ rounds… PRs needing 5+ rounds went from 8.9% to 1.5%." https://x.com/davekiss/status/2111188919660290362

The write-up, published 15 Oct 2026 by Addy Osmani, Morgan Westlee Lunt, and Qing Wang: https://claude.dev/blog/solving-code-review-at-anthropic/

What it says, exactly:

- One CLI PR "went through more than fifty rounds of review and fixes overnight".
- git blame: "about 59% of it was code that had been there since the reviewer's first pass."
- The cap: "After three rounds, the reviewer asks the author to step back and reconsider the approach."
- Also: "Diff-only re-review after the first few passes." "Flag the class of bug, so the author fixes every instance, not one line." "No repeats of findings already posted."
- Figure text: PRs passing after one round rose from 65% to 78%. PRs needing five or more rounds fell from 8.9% to 1.5%. Mean reviews to pass fell from 2.08 to 1.32.
- `REVIEW.md` lives with the code owners. Good ones "cap the nits, list paths to skip, and ask for a file and line as evidence". The summary "starts with a count and says plainly when there's nothing to fix."
- Findings are voted on by a second group of agents before a human sees them. Style, diagnostics, and cleanup are dropped. Tags are Important, Nit, or Pre-existing.
- They tell readers: "Cap review rounds; after the first few, re-review only the new diff."

Replies on Dave's post add no second measurement. @TriadicLAYLINES (16 Oct 2026) says the cap treats the symptom and that "Passing each round's resolved items into the next review fixes the cause." Opinion. https://x.com/TriadicLAYLINES/status/2111230194354618774 @shmoogy said he once hit 46 iterations on an earlier model. @Keivansamani said he still finds the cap "icky" and that designing the structure first spent fewer tokens. Opinion.

I found no credible dispute of the 8.9% to 1.5% figures.

### 3. Returning to an idle thread — holds with limits

The multipliers are one month of one person's Claude Code. The 60-minute expiry matches today's subscription default for the main conversation. It does not match subagents, API keys, or usage credits.

@quxiaoyin (6 Aug 2026), "110 sessions, 46,580 turns": while working, 1x; after coffee, 1.3x; after a call, 1.6x; after lunch, 13x. A new session was 4.5x. Advice: "summarize where we are" before leaving, then paste that into a fresh window. https://x.com/quxiaoyin/status/2085408811104534754

He did not publish a script in that thread. @fyzanshaik (7 Aug 2026) answered with cachewatch: https://github.com/fyzanshaik/cachewatch (opened). It is a macOS menu-bar app, marked unmaintained on the repo title. It reads local transcripts, shows a TTL countdown, and estimates "cost to resume". Its own README says cache reads are 0.1x at API prices, writes after expiry are 1.25x to 2x, and "Anthropic does not publish how tokens convert to quota percent." @Santosh74038967 pointed at SuperBased Observer in the same thread. @andresg747's advice was to train a `/handoff` habit.

Disputes on the TTL, same week: @jasonhaala said "The default for Claude is now 5 mins." @erdinc_akan said the default is 5 minutes and the 1-hour retention is a paid add-on. Those match API and subagent defaults, not the subscription main thread.

Current docs, opened: on a Claude subscription, inside plan usage, the main conversation (interactive, `claude -p`, and Agent SDK turns) gets one hour. "Everything else" (subagents, workflows, in-process teammates, forks, compaction) gets five minutes. API keys, usage credits, and cloud providers get five minutes for both. The one-hour write costs more, so it loses on short bursts that never idle past five minutes. https://code.claude.com/docs/en/prompt-caching

@dannygnc (22 Nov 2026): one team, one week, "$2,495 at API list prices, $960 avoidable. Failed tool calls: $68. Cache rebuilt after idle breaks: $532. History resent past 200k: $360." https://x.com/dannygnc/status/2124310666458902988 The post says to scan sessions "in your browser" and then the text I received ends. I did not get a URL, so I am not guessing one. On 23 Nov he added that no `CLAUDE.md` line fixes this and that `/clear` between tasks does. https://x.com/dannygnc/status/2124964657643561238

@ClaudeDevs (3 Nov 2026): "Claude Code now compacts long idle sessions right before the prompt cache expires (~1h). If you come back to one later, it only has to reprocess the summary instead of the entire conversation." Pro and Max also get a heads-up if resuming a very long conversation would use a big chunk of the 5-hour limit. https://x.com/ClaudeDevs/status/2117670659715375344 and the changelog link in that thread, https://code.claude.com/docs/en/changelog

I found no post with a before-and-after of that 3 Nov change. Danny's week is after 3 Nov and the idle-rebuild line is still the largest avoidable item in it. He does not say whether that feature was on.

Qu's later posts are not a re-measure. On 18 Sep 2026 he pointed at an AgentSky handoff so you can switch from Claude to Codex or OpenCode mid-session when you hit the 5-hour limit. https://x.com/quxiaoyin/status/2100973191141409013

### 4. Starting many agents at once — holds with limits

The limit: one run, by the author, and it applies to a workflow fan-out of same-prefix agents. It does not apply to separate Agent SDK sessions.

@noname_nonce (24 Nov 2026), Opus 5.5, same effort, no tools, numbers from Claude Code's own usage logs, one run each way. Three subagents in one message each wrote about 126k of fresh cache. The same three inside a workflow were staggered 1 to 2 seconds; agents 2 and 3 read about 113k and wrote about 13k. At API weights (cache write 1.25x, cache read 0.1x) the workflow's input cost was about 45% of the parallel subagents. He says a subagent he started on its own got the same cache hit, and "Anthropic doesn't publish how subscription limits weigh cache reads, so treat the 45% as a direction, not a guarantee." https://x.com/noname_nonce/status/2125169945759940759

I found no second person who repeated the 126k test.

The docs now describe the mechanism. A subagent "starts its own conversation with its own system prompt and tool set… Its first request doesn't read the parent's cache." A fork "inherits the parent's system prompt, tools, and conversation history exactly, so its first request reads the parent's cache." For a workflow fan-out of same-prefix agents, "Claude Code holds all but the first for up to 5 seconds by default, so their first requests can read the prefix that the first agent cached." https://code.claude.com/docs/en/prompt-caching

Agent SDK turns are in the "main conversation" bucket. Each child session an app launches is its own main conversation. Nothing in that page says those sessions are staggered, and a subagent's prefix is different from the parent's, so a later child does not read the parent's cache unless it is a fork or a same-prefix workflow agent.

### 5. Cache lifetime for subagents — holds with limits

The default is still 5 minutes for subagents, including on a subscription. One hour is opt-in, and the people who measured it mostly kept 5 minutes except for tasks that actually sit idle.

@diegohaz (9 Oct 2026) used 76% of his weekly limit in under 24 hours with 5–7 subagents. He said the subagent cache is 5 minutes versus 1 hour for the main agent, and that the next input "would cost 25x more" on up to 1M tokens, often 500k+. https://x.com/diegohaz/status/2108662928501035481

@lydiahallie, Claude Code at Anthropic (10 Oct 2026), corrected three points: only the subagent's own context is rewritten, not the parent's history, unless it is a forked subagent; a cache hit is 0.05x normal input, so expiry is 25x the discounted price and 1.25x versus normal input; the 5 minutes run from the last request, including tool calls. "if you have a lot of idle subagents, you can set subagentPromptCacheTtl to `1h`". https://x.com/lydiahallie/status/2108995470265815103

@bcherny (10 Oct 2026): "API is usually used for Agent SDK shaped workloads, where 1h TTL would result in higher costs on average, so we stick to 5m to be safe. Interactive usage defaults to 1h because that is cheaper for people on average." https://x.com/bcherny/status/2108768259516281038

@daniel_mac8 (10 Oct 2026) first posted `"subagentPromptCacheTtl": 60`, then ran his own history and switched back to 5m. The script is public: https://gist.github.com/DannyMac180/9b9196d2434709f3ac04bc394d02a562 (opened). It is read-only, scans `~/.claude/projects/**/*.jsonl`, prices 5-minute writes at 1.25x and 1-hour writes at 2.0x, and calls 1 hour "likely worth it" only above a 5% estimated saving. https://x.com/daniel_mac8/status/2108971441102172620

@Alpaca_Capital (10 Oct 2026): about 1% of pauses caused nearly half of his subagent cache writes (long builds, polling, waiting on Codex, resumed subagents). A flat 60-minute cache was about 5% cheaper overall, and about 80% of sessions got more expensive. He now routes only the long tasks to a 60-minute subagent, and estimates 6–8% from that, "theoretical" 10%. https://x.com/Alpaca_Capital/status/2108789031953985549

Haz's own change, same thread: stop using subagents for this work and have the orchestrator spawn background threads. He later wrote that a skill doing that cut his usage "by at least 30%" versus subagents. The gist URL is in his post; I did not open the gist. https://x.com/diegohaz/status/2108822553883980037

Docs today match Lydia and Boris: subagents stay at five minutes on a subscription until you set `subagentPromptCacheTtl` or `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` (Claude Code v2.1.242 or later). A per-subagent `cacheTtl` in frontmatter needs v2.1.248 or later, and a `1h` there is ignored while the subscription is on usage credits. `ENABLE_PROMPT_CACHING_1H=1` asks for one hour on both buckets. `FORCE_PROMPT_CACHING_5M=1` forces five minutes.

### 6. Inherited effort — holds, with no published before-and-after on the current models

@bcherny (28 Oct 2026), after Anthropic read the session IDs people sent: no single root cause and no Claude Code regression. Faster limit burn "traced back to one or two heavy sessions (xhigh effort, /goal, Workflows, Agent Teams, long SDK jobs)." "Subagents inherit it by default and it can add up fast. /usage also shows where a session's tokens went." https://x.com/bcherny/status/2115592371224346734

@sam_commonly asked the same day if subagents can default to a lower effort. https://x.com/sam_commonly/status/2115598844901917174

@ClaudeDevs (3 Nov 2026) shipped the control: "You can now also tell Claude which effort level to run a subagent at." `ultrathink` in a message bumps that one turn to xhigh. https://x.com/ClaudeDevs/status/2117670661221134775

I did not find a heavy user posting a weekly-limit number for "main on high, children on medium" on Opus 5.5 or Fable 5.2. What the docs add: on a subscription, changing effort keeps the cache on Fable 5.1 and later, Opus 5.5, Sonnet 5.5, and Haiku 5.5. On most other models, an effort change recomputes the whole request. The cost page says Sonnet for most coding, Opus for architecture, `model: haiku` for simple subagents, and Sonnet for agent teammates. `/usage` can recommend a row like "Switch from max to high effort · /effort high", and `f` runs that command (v2.1.305 or later). https://code.claude.com/docs/en/costs and https://code.claude.com/docs/en/prompt-caching

@lydiahallie (9 Nov 2026) says she uses Fable 5.2 as the orchestrator with Opus 5.5. No usage delta. https://x.com/lydiahallie/status/2119871667400999166

### 7. Whether cache reads count against subscription limits — the "free" claim does not hold

@KentonVarda (22 Mar 2026): "under the subscription plan, input token cache reads are *free* (don't count against quota)". He framed it as what "people pointed out", and he was comparing with OpenCode on API pricing. https://x.com/KentonVarda/status/2035842457309475037

Same day, @denisandrejew: "cached input tokens being free on subs seems extremely unlikely. i've researched and can only find speculation". https://x.com/denisandrejew/status/2035852553896731056

Newest staff statements I found:

- @lydiahallie (1 Sep 2026): "Cache reads have always been counted at a reduced rate toward subscription usage!" https://x.com/lydiahallie/status/2094855759398113569
- @ClaudeDevs (1 Sep 2026): Fable 5.1 cache reads on usage-based billing are "$0.25/MTok, down from $1/MTok on Fable 5." "In Claude Code, cache reads count at a reduced rate toward subscription usage, which remains unchanged. Long sessions billed at API rates can net out 25%-45% cheaper." https://x.com/ClaudeDevs/status/2094851235694497836
- Lydia's 10 Oct reply, above, gives the price relationship she was using then: hits at 0.05x normal input, a miss at 1.25x normal input. She was correcting the 25x claim. She did not say the weekly bar uses 0.05x.

@noname_nonce (24 Nov 2026) still writes that Anthropic does not publish how subscription limits weigh cache reads. Cachewatch's README, opened, says the same and fits a quota percent from the user's own account instead of using a published formula. I found no newer staff post that gives the weekly-limit weight for cache reads, cache writes, output, and model as separate numbers.

### 8. Empty instruction file and 350+ skills off — holds with limits

The limit: it is his logs, nobody else in the thread said they tried it, and the post does not name Claude Code. His bio says he is "obsessing over token costs, my pi coding agent".

@tianrendong (23 Nov 2026): two weeks in a large monorepo, "all 350+ repo skills turned off and an empty AGENTS.md", "about 25% lower token costs, and the model didn't act dumber." https://x.com/tianrendong/status/2124948368569069678

His follow-up, same hour, "across ~8k turns": "saved ~60k tokens/turn (350+ skills descriptions + 40kb AGENTS.md)", "~25% lower spend calculated from cache hit/miss logs", "tool call error rates stayed flat", "didn't have to correct the agent more often." An accidental worktree that still had the instructions, "~600 turns", "didn't feel any smarter". Asked whether the 25% was fewer skill descriptions or fewer tool calls, he answered "yep just fewer skill descriptions and system prompt". https://x.com/tianrendong/status/2124948479776821565 and https://x.com/tianrendong/status/2124961790694154407

I found no other person reporting the same experiment.

## Part 2

### A. T3 Code

Claim: T3 has a setting that compacts idle Claude threads over 200,000 tokens before the cache expires. The 20% weekly gain is still only Lucas's claim. Shown for the setting existing: @OmiidSayfun, 17 Nov 2026, https://x.com/OmiidSayfun/status/2122577732253188098 and @AdamHoltererer, 23 Nov 2026, https://x.com/AdamHoltererer/status/2124846737760157897. Opinion for the 20%: @lucastohdev, 23 Nov 2026, https://x.com/lucastohdev/status/2124809609068327249.

Claim: almost all T3 use is people bringing their own subscriptions, and almost all of that is Codex, Claude, and Antigravity. @theo (21 Nov 2026): "99% of T3 Code usage is with subscriptions" and "~98.5% of all T3 Code usage is from Codex, Claude and Antigravity." Shown as the developer's own figures, not a saving. https://x.com/theo/status/2124284420060578088 and https://x.com/theo/status/2123965784888377541

I found no T3 developer post that names a second, lean Claude config for child agents, a T3 usage view with a token breakdown, or a measured fix for a T3-specific token waste beyond the compact-idle setting. The env var people pair with it is `CLAUDE_CODE_AUTO_COMPACT_WINDOW` (also `autoCompactWindow` and `/autocompact`).

### B. Plugins, skills, hooks, MCP

Claim: the skill listing costs context on every turn, at 1% of the context window, and unused skills should be turned off. Shown in the current docs: https://code.claude.com/docs/en/skills. The listing always keeps every name. Descriptions are dropped starting with the skills you use least. Each description is capped at 1,536 characters (`skillListingMaxDescChars`). Raise the budget with `skillListingBudgetFraction` (for example `0.02`) or `SLASH_COMMAND_TOOL_CHAR_BUDGET`. `skillOverrides` values are `on`, `user-invocable-only`, `name-only`, and `off`. `/doctor` estimates the listing cost. `/skill-doctor` (v2.1.252 or later) flags skills never invoked and lists unused plugins. Plugin skills are not controlled by `skillOverrides`; those go through `/plugin`.

@stewy0x (24 Nov 2026) repeats that 1% budget and the same commands. Shown as a restatement of the docs, not a new measurement. https://x.com/stewy0x/status/2125144044116299841

@daniel_mac8 (21 Aug 2026): `/doctor` "saved me 24k tokens of unnecessary skills, CLAUDE.md context and unused MCPs". Shown as his number, with a video. https://x.com/daniel_mac8/status/2090884114958422136

@iamleannmuller (19 Nov 2026): `/context` showed 194,400 tokens used, 174,500 of them the old chat. Her list is `/clear`, `/mcp` to turn off unused servers, `CLAUDE.md` under 200 lines, `/model` before the task, and a subagent for repo search. The 194,400 / 174,500 split is shown; the five habits are opinion plus that one reading. https://x.com/iamleannmuller/status/2123249929762079173

Official cost page, same direction: keep `CLAUDE.md` under 200 lines and move workflow instructions into skills; MCP tool definitions are deferred until used; `gh`, `aws`, `gcloud`, and `sentry-cli` add no per-tool listing. https://code.claude.com/docs/en/costs

I did not find an independent measured token count for superpowers, a memory plugin, or a browser MCP. The superpowers posts I opened are install pitches (15 skills, 2–5 minute tasks, a fresh subagent per task) with star counts, not context measurements. Repo-claim roundups, not independent runs: @undefinedKi (9 Oct 2026) relays context-mode's "315 KB of output becomes 5.4 KB" and "up to 98%", https://x.com/undefinedKi/status/2108661690162466859. @0xbobaaa (22 Nov 2026) relays six repos' own benchmarks (96.3% fewer tokens per lookup, 61% less context, 0.71x cost, ~1,200 tokens instead of 23,000, 25% then 85%+) and says so. https://x.com/0xbobaaa/status/2124489177605542052

### C. Claude Code features since September 2026

These are in the docs and the 3 Nov recap. I did not find a user posting "I turned this on and my week dropped by N%" for Code Mode, tool search, or forks.

- `/usage` shows plan bars, a Session token block, and a `Prompt cache (main)` line (v2.1.251+): hit share, misses, expected rebuilds from compaction, warm or cold, and the TTL. It does not include subagents. Attribution by skills, subagents, plugins, and MCP servers; `d` and `w` switch 24 hours and 7 days. A row's share is how much of your usage the row applies to, and shares can add past 100%. `f` applies the rows it can (v2.1.305+), including an `/effort` change. Before v2.1.222, one MCP call caused every later request to be attributed to that server. @ClaudeDevs, 3 Nov 2026: "/usage in the CLI now tells you in plain words what's using up your limits. If there's a quick fix, you can press f to apply it right there." https://x.com/ClaudeDevs/status/2117670661221134775. Shown. https://code.claude.com/docs/en/costs
- `/doctor` and `/skill-doctor`, `skillOverrides`, and the 1% listing budget, as in B. Shown in https://code.claude.com/docs/en/skills
- Idle compaction before the ~1 hour expiry, and the Pro/Max heads-up before a resume that would take a big piece of the 5-hour limit. Shown as shipped: https://x.com/ClaudeDevs/status/2117670659715375344. No user before-and-after found.
- Per-model auto-compact: `/autocompact`, `autoCompactWindow`, `CLAUDE_CODE_AUTO_COMPACT_WINDOW`. Changelog: in-process teammates on another model compact at their own limit, at that model's rate. Shown in https://code.claude.com/docs/en/changelog
- Status line can read the same cache numbers from the `prompt_cache` object. Shown in the costs page.
- Code Mode and tool search appear in the changelog I opened (Code Mode can drop the prompt cache on resume in another permission mode; tool search shows more of an MCP tool's description; MCP tools are deferred by default). I did not open a user measurement that either one lowered a weekly limit.
- Forks read the parent cache on the first request. Shown in the prompt-caching page. No dollar figure from a user.
- I found no posts or doc hits for `/subtask`.

### D. Model routing

Claim: xhigh, Workflows, Agent Teams, and long SDK jobs were what burned the limits in the sessions Anthropic reviewed. Shown: @bcherny, 28 Oct 2026, https://x.com/bcherny/status/2115592371224346734

Claim: Sonnet for ordinary work and for teammates, Opus for architecture, Haiku for simple subagents. Shown as the current cost-page guidance, not as someone's A/B. https://code.claude.com/docs/en/costs

Claim: Fable 5.2 as orchestrator and Opus 5.5 under it. Opinion, no usage number: @lydiahallie, 9 Nov 2026, https://x.com/lydiahallie/status/2119871667400999166

Claim: replacing idle subagents with background Claude threads cut usage at least 30%. Shown as his test: @diegohaz, 10 Oct 2026, https://x.com/diegohaz/status/2108822553883980037

Claim: inside T3, Codex, Claude, and Antigravity are ~98.5% of use, and 99% of use is subscriptions. Shown as @theo's figures, 21 Nov 2026, https://x.com/theo/status/2123965784888377541. That is a mix, not a measured saving from moving review off Claude.

I did not find a careful "top model versus the next one down saved this much of my Claude week" post in this pass.

### E. Orchestration shape

Claim: a directory-scoped handoff file, written before compaction, lets a new session start from the summary instead of the transcript. @VicVijayakumar (14 Sep 2026) uses `/handoff` to write a timestamped `handoff.md`, `/handon` to read the newest one, and a pre-compaction hook that runs `/handoff`. He says it "seems to work really well". Opinion, no token count. https://x.com/VicVijayakumar/status/2099322374378168716

Claim: six short lines in `AGENTS.md` (grep before reading a file, subagent for whole-repo questions, show the diff, don't restate decisions, suggest `/compact` or a new session on a task switch, short answers). Opinion: @Voxyz_ai, 8 Nov 2026, https://x.com/Voxyz_ai/status/2119535433969500172

Claim: compact at a saved boundary and keep a pointer handoff, including under T3's idle compaction. Shown as the author's three-week log (251 compactions, hard-limit compaction never ran): the compact-timing README, https://github.com/lbzepoqo/compact-timing, linked from @lbzepoqo on 24 Nov 2026, https://x.com/lbzepoqo/status/2125130290662441275

Claim: a subagent's result appends to the parent and leaves the parent's prefix intact, so a warm parent pays a cache read plus the new result. A parent idle past its TTL pays a full rewrite. Shown as the mechanism in https://code.claude.com/docs/en/prompt-caching. I found no user measurement of the cost each time a child finishes and wakes a large parent.

Claim: agent-team tokens scale with team size, teammates load `CLAUDE.md`, MCP, and skills on their own, and spawn-prompt text is in their context from the start. Shut them down when the work is done. Shown in https://code.claude.com/docs/en/costs. Enable with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`.

### F. Subscription limits, October and November 2026

Claim: Fable usage is a separate slice of the weekly limit. @lydiahallie (9 Nov 2026): "When you use it the Fable usage on your weekly limit gets reset, but what you've used on other models stays the same" and "it only resets the part of your weekly limit you used on Fable". Shown. https://x.com/lydiahallie/status/2119877841559097526 and https://x.com/lydiahallie/status/2119940023307124820

@bcherny (9 Nov 2026): Pro gets one Fable reset, and Max, Ultra, and Team Premium get up to three, depending on how long you have subscribed. Shown. https://x.com/bcherny/status/2119890820186976763

Claim: a few bugs used more limits than intended, and a Fable-only reset did not help people who had already hit the weekly limit on every model. @ClaudeDevs (10 Nov 2026): one free full reset for Pro, Max, Ultra, and Team, all models and limits, usable before 10 Dec 2026 at 2:00 PM PT, in Settings > Usage. Shown. https://x.com/ClaudeDevs/status/2120275296637321476

Claim: Max, Ultra, and Team plans include a monthly API credit, and Pro and Max warn before a resume that would take a large piece of the 5-hour limit. Shown: @ClaudeDevs, 3 Nov 2026, https://x.com/ClaudeDevs/status/2117670664262009016 and https://x.com/ClaudeDevs/status/2117670659715375344

I did not find, in the posts I opened, the numeric weekly allowance for each plan or for the top model.

### G. Other numbers

- Review convergence, already in Part 1: 8.9% to 1.5% of PRs needing 5+ rounds, mean reviews 2.08 to 1.32. Shown in the 15 Oct 2026 write-up.
- Background threads instead of idle subagents: at least 30%. Shown: @diegohaz, 10 Oct 2026.
- One week at list prices: $532 of $960 avoidable was idle cache rebuilds, $360 was history past 200k, $68 was failed tool calls. Shown: @dannygnc, 22 Nov 2026.
- @0xSolty (24 Nov 2026) says one-function edits were burning 40–60k tokens because the agent searched the repo in the main session, and he moved that search to a subagent over a graph. He states it as his repo. A video is attached; the text has no log table. Treat the 40–60k as his report. https://x.com/0xSolty/status/2125302896309022752
- Cachewatch's README: a cold 400k session is "real money, or real quota", and the app's status line can show model, context percent, 5-hour percent, and 7-day percent. The 400k example is their illustration, not a published customer bill.

## The 8 most useful things this round adds

Ordered by how much I would expect them to save a heavy multi-agent user. The 20%, the 3-round headline, the 13x lunch figure, the 5-minute subagent default, "subagents inherit effort", "cache reads are not free", and the 60k-tokens-per-turn experiment were already in round one. These are the parts that were not.

1. Launch same-prefix agents as a workflow fan-out, or fork the parent. Claude Code holds every agent after the first for up to 5 seconds so they read the first agent's cache. Three subagents started in one message did not share a cache (about 126k written each, one run). A separate Agent SDK session is its own main conversation and does not get that hold. https://code.claude.com/docs/en/prompt-caching and https://x.com/noname_nonce/status/2125169945759940759
2. Stop leaving subagents idle past 5 minutes. Lydia's correction plus the docs: they stay at 5 minutes even on a subscription. Haz measured at least 30% less usage by having the orchestrator spawn background threads instead. Alpaca found about 1% of pauses causing nearly half of subagent cache writes, and a blanket 1-hour TTL made about 80% of sessions more expensive. https://x.com/diegohaz/status/2108822553883980037 and https://x.com/Alpaca_Capital/status/2108789031953985549
3. The review loop needs more than a cap of 3. Also re-review only the new diff, do not repeat a finding, flag the class of bug, and put a short `REVIEW.md` on the base branch that asks for a file and line and says when there is nothing to fix. About 59% of what the reviewer kept flagging was already in the first pass. Five-or-more-round PRs fell from 8.9% to 1.5%, and mean reviews to pass from 2.08 to 1.32. https://claude.dev/blog/solving-code-review-at-anthropic/
4. Turn off skills the session never calls, and keep the instruction file short. `/skill-doctor` and `/doctor` report the cost. `skillOverrides` of `off` or `name-only` drops them. The listing budget is 1% of the window. Official guidance is `CLAUDE.md` under 200 lines, with the rest in skills that load on demand. One `/doctor` pass was reported at 24k tokens of skills, instructions, and unused MCP. https://code.claude.com/docs/en/skills and https://x.com/daniel_mac8/status/2090884114958422136
5. Compact at a task boundary, with a handoff, and compact an idle thread before the hour is up. T3's "Compact idle threads" setting, as users describe it, does the idle half for Claude threads over 200,000 tokens. The compact-timing plugin refused to compact mid-step, compacted 251 times in three weeks, and never hit Claude Code's hard limit. Danny's week still put $532 on idle rebuilds and $360 on history resent past 200k, after the 3 Nov idle-compact ship date. https://github.com/lbzepoqo/compact-timing and https://x.com/OmiidSayfun/status/2122577732253188098
6. Set the child's effort in the prompt, and keep the parent off xhigh. Anthropic's review of real sessions named xhigh, `/goal`, Workflows, Agent Teams, and long SDK jobs as the fast burns, and said subagents inherit effort. As of 3 Nov you can name the subagent's effort. On a subscription, Opus 5.5, Fable 5.1 and later, Sonnet 5.5, and Haiku 5.5 keep the cache across an effort change, so the drop does not force a full rewrite. No one published the weekly-limit delta for that change. https://x.com/bcherny/status/2115592371224346734 and https://code.claude.com/docs/en/prompt-caching
7. `/clear`, or a new session from a handoff, when the task changes. One `/context` reading was 174,500 of 194,400 tokens from the previous chat. The cost page says stale context is paid on every later message. Qu's month said a fresh session after a long break was 4.5x against 13x to continue the cold thread. https://x.com/iamleannmuller/status/2123249929762079173 and https://x.com/quxiaoyin/status/2085408811104534754
8. Treat Fable as its own slice of the weekly limit, and keep it on the coordinator. Resetting Fable on 9 Nov left every other model's usage in place. On 10 Nov Anthropic said a few small issues had used more limits than intended and gave Pro, Max, Ultra, and Team one full reset before 10 Dec 2026, 2:00 PM PT. Lydia's own split is Fable 5.2 orchestrating Opus 5.5, with no number attached. https://x.com/ClaudeDevs/status/2120275296637321476 and https://x.com/lydiahallie/status/2119940023307124820

## What I looked for and did not find

- A T3 Code developer confirming or denying the 20%, or the "1M and no auto compaction" wording. @theo's posts that I opened from this period do not mention compaction.
- Anyone else's before-and-after of `CLAUDE_CODE_AUTO_COMPACT_WINDOW=300000` plus Compact Idle Thread.
- A measured before-and-after of the 3 Nov idle compaction.
- A second run of the 126k parallel-subagent test.
- A staff number, newer than "reduced rate" (1 Sep 2026), for how the weekly limit weighs cache reads, cache writes, output, and model. Lydia's 0.05x / 1.25x on 10 Oct is the price relationship she used to correct the 25x claim, not a published weekly-limit formula.
- Anyone who repeated the empty `AGENTS.md` and 350-skills-off experiment.
- An independent measured context cost for superpowers, a memory plugin, or a browser MCP.
- A user result for `/subtask`, or a user result that Code Mode or tool search lowered a weekly limit.
- A measured cost of waking a large parent each time a child finishes.
- The numeric weekly allowance per plan, or per top model, in the October and November posts I opened.
- A heavy user's before-and-after of dropping xhigh to high on Opus 5.5 or Fable 5.2.
- The URL at the end of @dannygnc's 22 Nov post. The text I received stops at "in your browser:".
