# Multi-provider lean agents, token use and shared rules: X research (2026-11-27)

Grok (`grok-4.7`, high reasoning effort) searched X on 2026-11-27 for late-2026 posts on lean child profiles outside
Anthropic, token/cache behavior on Antigravity (Gemini 4 Neon), Codex and Grok, and shared global rules/memory files.
Every claim below is `reported` or `opinion` until backed by primary documentation or local measurement.

---

X search worked. Late-2026 posts are thick on Antigravity cache and quota, and on sharing `AGENTS.md`. They are thin on a named “lean child” profile for Codex, Antigravity, or Grok, and almost empty on prompt-cache survival when you switch providers. Numbers below are labeled as a user’s own count, a relay of a vendor or benchmark, or an opinion.

## 1. Lean children, and what actually burns context or quota

**The lean-profile pattern on X is still mostly Claude Code.** I did not find posts describing a stripped provider profile for Antigravity, Gemini CLI, Codex, or Grok (fewer plugins, MCP servers, skills, and rules, used only for child agents). What people do describe for the other harnesses is a smaller version of the same idea: short rules, MCP schemas not loaded up front, lower default reasoning, and a cheap worker beside an expensive planner.

Closest non-Claude mechanisms people actually posted:

- Antigravity CLI v1.5.13–v1.5.18 can set planner and executor to different models and different thinking strength, adds `/context` for a custom auto-compaction threshold, and scans a shared `~/.agents/skills`. [@erintwalsh_](https://x.com/erintwalsh_/status/2126411148584968475) (27 Nov 2026, announcement) and the feature list in [@ChuckBass8808](https://x.com/ChuckBass8808/status/2126438273874882608) (27 Nov 2026). That is a role split, not a lean profile.
- `/swarm` is the opposite of lean. One user reported about 4–5× quota on an audit, with workers on Gemini 4 Hydrogen. [@carverfomo](https://x.com/carverfomo/status/2126435191145037911) (27 Nov 2026). Anecdote, one run.
- Codex and Claude Code both compact by summarizing and restarting. One measured Opus 5.5 compaction of a ~400k-token session: 38 s and $1.99 after the cache expired, 29 s and $0.17 while it was warm. [@Alezander9](https://x.com/Alezander9/status/2115842962521509968) (29 Oct 2026). His own measurement. He argues the summary is lossy.

### Antigravity / Gemini 4 Neon — the concrete numbers

These are the strongest measurements in the set, and they are user telemetry, not a Google paper. Antigravity’s lead replied to the cache-hit post and called it unexpected. He did not confirm the figure. [@_mohansolo](https://x.com/_mohansolo/status/2123629978311078311) (20 Nov 2026), replying to [@shneural](https://x.com/shneural/status/2123404925677994366) (19 Nov 2026).

| Claim | Number | Who | Date | Kind |
|---|---|---|---|---|
| Neon cache hit vs idle gap | <15 s ~85%; >30 s 33.6%; 1–2 min ~9%. Hydrogen stayed ~90.9% | [@bebuu_ai](https://x.com/bebuu_ai/status/2123414001912861014), 2,031 calls / 195 sessions | 19 Nov 2026 | User telemetry |
| Same dump, session id | CLI v1.5.11 and v1.5.12 sent one empty-string hash as `sessionId` across sessions | same post | 19 Nov 2026 | User telemetry |
| Neon vs others | Neon ~9% hit after 1 minute; Opus 5.5 ~95% at 1 h; GPT in Codex ~97% at 24 h; Neon “~3×” limit burn | [@shneural](https://x.com/shneural/status/2123404925677994366) | 19 Nov 2026 | User claim, widely reposted, not confirmed by Google |
| Misses vs context length, Neon High | 50–100k: 10% zero-cache misses; 250–300k: 43%; 300–500k: 65%. One 3-hour run wiped a 5-hour bar and 58% of weekly quota. 385 calls | [@TekonBurns](https://x.com/TekonBurns/status/2124618423288336531) | 22 Nov 2026 | Secondhand local DB logs |
| Why the TTL is short | Implicit cache wants a similar prefix and a short gap. Explicit caching is not supported on Gemini 4 models | [@KingTowdah](https://x.com/KingTowdah/status/2123429106494685674), reply in the bebu thread | 19 Nov 2026 | User citing Google caching docs |
| Same task, quota | Neon Medium 2.07% of the 5-hour bar vs Hydrogen High 1.43% (31% less) on 4 bug fixes + 5 tests | [@yuhasbeentaken](https://x.com/yuhasbeentaken/status/2126353585369666029) | 27 Nov 2026 | One user’s A/B |
| Subscription vs API | $100 Antigravity ≈ 1.5B tokens, “most of that is cache”; $100 GPT ≈ 5B; $20 Claude Code ≈ 5B/week. Uses Neon as orchestrator for other providers and guesses cache misses | [@FloorIsTired](https://x.com/FloorIsTired/status/2125624205664268378) | 25 Nov 2026 | Personal bill, not a controlled test |
| Vendor cache claims | Neon cache hit +4.5 points, 28% more tasks inside quota, 75% more effective usage over five days. No baseline or method published | [@stretchcloud](https://x.com/stretchcloud/status/2126006570362535946) | 26 Nov 2026 | Relay of Google announcements, author flags the missing method |
| Context-length quota (unshipped) | 512k ≈ 1.3× per turn, 900k ≈ 1.8× | [@thtbee_](https://x.com/thtbee_/status/2114726139923169785) | 26 Oct 2026 | “Potentially coming,” not a measurement of what shipped |
| Open question | Does changing reasoning effort mid-session reset the Neon/Carbon cache? | [@mariofilhoml](https://x.com/mariofilhoml/status/2126390384330670274) | 27 Nov 2026 | Asked, no answer in the thread |

AA cost-per-task, relayed, harness included so this is not a pure model comparison:

- Gemini 4 Neon (xhigh) in Antigravity CLI: $6.86 / score 59. Claude Opus 5.5 (max) in Claude Code: $12.61 / 58. Claude Fable 5.2 (xhigh): $15.47 / 58. [@EvanOtero](https://x.com/EvanOtero/status/2123172006271074720) (18 Nov 2026, Google DeepMind) and [@asifroyal](https://x.com/asifroyal/status/2124888204197388380) (23 Nov 2026).
- A different AA page, Intelligence Index at max effort: Opus 5.5 $10.04/task, Neon xhigh $6.44, GPT-6.2 Sol $1.01. [@fihsar](https://x.com/fihsar/status/2126388281034350982) (27 Nov 2026). Do not mix this with the Coding Agent Index row above.

`thought_level`: people talk about xhigh vs medium as a quota lever ([@yuhasbeentaken](https://x.com/yuhasbeentaken/status/2126353585369666029) keeps Hydrogen Low/High for workers and Neon Medium/High for orchestration). I did not find a token-count table for xhigh vs medium vs high on the same prompt.

### Codex

- **32 KiB cap is repeated as current docs behavior, not as someone’s byte count.** Codex reads `~/.codex` then the repo then subfolders, closest last, and stops once `project_doc_max_bytes` (default 32 KiB) is hit. UTF-8 CJK hits it sooner. [@mhimt](https://x.com/mhimt/status/2126338230304030747) (27 Nov 2026), [@johnnynelai](https://x.com/johnnynelai/status/2126425329359258005) (27 Nov 2026), [@8zero7](https://x.com/8zero7/status/2126408695852462163) (27 Nov 2026), [@Voxyz_ai](https://x.com/Voxyz_ai/status/2107939019091005836) (7 Oct 2026). Docs summaries.
- **Reasoning effort:** OpenAI’s migration note, as relayed, says start GPT-5.5 at low or medium rather than inheriting old prompts. [@mhimt](https://x.com/mhimt/status/2126338230304030747) (27 Nov 2026). No measured low-vs-high token ratio on X in this window.
- **Effort does not apply mid-turn.** The ChatGPT app kept Ultra after it was turned off; one turn ran more than 7 hours after a switch to Standard. Change lands on the next turn. [@Josantonius](https://x.com/Josantonius/status/2126443122750275969) (27 Nov 2026). Anecdote.
- **Cache miss got cheaper, still not free.** A full 475k context used to cost about $9.50 API-equivalent on a miss and then about $0.95. [@petergostev](https://x.com/petergostev/status/2112808451353047427) (21 Oct 2026). His observation, “perma cache,” no method.
- **What fills the window after startup is not MCP results.** 14 days of Claude Code and Codex: 970 sessions, 146k turns, $10,953 list-price equivalent. 21.7% of each turn was leftover shell output. 4.8% was 14,440 screenshots held for an average of 102 turns. MCP tool results were under 1%. [@btsouth](https://x.com/btsouth/status/2125431487461740662) (25 Nov 2026). His own history. This does not contradict the schema cost below. Schemas are paid at the start. Results were small in his logs.
- Alpha diff, not a shipped measurement: “10× additional context,” remote reasoning-effort switching, expanded memory hooks, per-provider quota. [@StatsWire](https://x.com/StatsWire/status/2126389761443021056) (27 Nov 2026), Codex v0.172.0 vs v0.174.0-alpha.9.

### Grok

- **Grok Bot usage export, one app:** 5.97B tokens processed, 5.68B cached (~95%), 142M fresh input, 113M cache writes, 34M output. Two heavy days were ~3.28B. He asks whether Claude Code, Codex, or Gemini CLI would do the same repo for less. No answer in the thread. [@codliva87](https://x.com/codliva87/status/2126083760563142934) (26 Nov 2026).
- **Grok Build release notes with a real context cut:** MCP tools from FastMCP or the official MCP SDK were sending the result twice. v1.0.72 sends it once, “about half the context” per those calls. [@XFreeze](https://x.com/XFreeze/status/2115228120001495117) (27 Oct 2026).
- **Skill bodies are capped.** v1.0.33 caps a skill file at the same 25k-token read limit as other files, and long sessions keep compaction checkpoints through the 30-day sweep. `/memory` browses and deletes notes. [@XFreeze](https://x.com/XFreeze/status/2100070332082389392) (16 Sep 2026).
- The “150,000 tokens of tools down to 2,000, 98.7%” line is Anthropic’s code-mode figure, then pasted into Grok Bot prompts. [@Mahaximus_](https://x.com/Mahaximus_/status/2115097952494465281) (27 Oct 2026), [@Corki_0x](https://x.com/Corki_0x/status/2124890507700469895) (23 Nov 2026). Not an independent Grok measurement.
- I did not find a Grok `reasoningEffort` token table, or a measured cache TTL for Grok Build.

### MCP schemas, the cross-provider startup cost

- 100+ tools: schemas alone 50k–70k tokens before the first prompt. Uber’s writeup, relayed by [@undefinedKi](https://x.com/undefinedKi/status/2116247505193050474) (30 Oct 2026), source named in the reply: `uber.com/us/en/blog/efficient-software-factory/`. Also from that post, presented as Uber’s setup: compact at 400k even on 1M context, default reasoning Medium, main-session cache 1 hour and subagent cache 5 minutes, batching tool calls cut tokens 55–71%. Weekly users 7×, cost per session −52%. Those are Uber’s figures as retold.
- 585 published estimates: 47 tokens (1 tool) up to ~472,113 tokens (445 tools, 236% of a 200k window). [@testedmcp](https://x.com/testedmcp/status/2125581054664729087) (25 Nov 2026). Estimates, not live traces.
- Anthropic tool-use writeup, retold: a 500-token search tool instead of 58 loaded tools, 77,000 → 8,700 tokens. `defer_loading: true` except the top 3–5 tools. Opus 4 accuracy 49% → 74% with tool tokens −85%, same model. [@slash1sol](https://x.com/slash1sol/status/2126297845770715610) (27 Nov 2026). Paper summary, not a user’s repo.
- claude-mem’s own MCP definitions: one user measured ~250 tokens for the history index and ~5,200 tokens fixed overhead on every request, including when used from Codex. [@sketch942001](https://x.com/sketch942001/status/2126447420762447925) (27 Nov 2026). His measurement.

## 2. Shared user-level rules and memory

The pattern that shows up over and over is one short `AGENTS.md`, imported or symlinked, with provider-only rules kept out of it. A single master symlink of a long Claude file is the failure mode people describe.

**Precedence, from people who say they checked the docs on 27 Nov 2026:**

- Claude Code v2.1.277+: reads `AGENTS.md` only when there is no `CLAUDE.md`. If both exist, only `CLAUDE.md`, unless it contains `@AGENTS.md`. A `CLAUDE.local.md` can quietly stop the team `AGENTS.md` from being read. [@johnnynelai](https://x.com/johnnynelai/status/2126425329359258005), [@YH31377807](https://x.com/YH31377807/status/2126338004310745372), [@mhimt](https://x.com/mhimt/status/2126338230304030747).
- Codex: global `~/.codex`, then root down to the launch folder, closest last, hard stop at 32 KiB, read once at session start. It does not reread when you `cd`. [@Voxyz_ai](https://x.com/Voxyz_ai/status/2107939019091005836) (7 Oct 2026).
- Claude Code loads a subfolder `AGENTS.md` only when it reads a file in that folder. Same post.
- `@` imports do not save tokens. The imported file is still loaded in full at launch. Same post. That is the argument against “tidy” imports as a context strategy.
- Gemini CLI: one docs summary says it reads `GEMINI.md`, and `AGENTS.md` only if you add it with `context.fileName` in `settings.json`. [@YH31377807](https://x.com/YH31377807/status/2126338004310745372) (27 Nov 2026). Another post the day before says Gemini CLI already reads `AGENTS.md`. [@lenny_enderle](https://x.com/lenny_enderle/status/2125937624149823901) (26 Nov 2026). I could not settle which is current from X alone.
- I did not find a post that spells out `~/.grok/AGENTS.md` symlink behavior. Grok Build’s September notes cover a `/memory` browser, not a shared rules file.

**Symlink vs split vs skills**

- One folder, symlinked into `~/.claude/CLAUDE.md` and `~/.codex/AGENTS.md`, skills synced with a script. Peter Steinberger’s setup, retold by [@undefinedKi](https://x.com/undefinedKi/status/2104656567505211685) (28 Sep 2026). He warns the published `AGENTS.md` contains that author’s own hosts. Same idea, shorter: [@prayushkale](https://x.com/prayushkale/status/2124843350826389920) (23 Nov 2026), [@mattbwaite](https://x.com/mattbwaite/status/2125235566505853158) (24 Nov 2026), [@soniacuts](https://x.com/soniacuts/status/2098381050951143608) (11 Sep 2026).
- Split is the advice once the file is long. Root under ~200 lines. Folder rules in that folder. Procedures in skills. Anything that must hold goes in a hook or a failing check, not in markdown. [@Voxyz_ai](https://x.com/Voxyz_ai/status/2107939019091005836), [@mhimt](https://x.com/mhimt/status/2126338230304030747), [@Alex_Vainshtein](https://x.com/Alex_Vainshtein/status/2116418200514519205) (31 Oct 2026: an 800-line `CLAUDE.md` still gets ignored once the window is noisy).
- Every installed skill’s name and description is in context on every turn. `/skill-doctor` shows the cost. `/doctor prompt-audit` (v2.1.283+) flags rules written for older models. [@Voxyz_ai](https://x.com/Voxyz_ai/status/2107939019091005836). I did not find posts using the exact phrase `trigger: model_decision`.
- Antigravity now discovering `~/.agents/skills` means a global skill directory is shared with Claude Code whether or not you meant the child to have it. [@ChuckBass8808](https://x.com/ChuckBass8808/status/2126438273874882608) (27 Nov 2026).

**Stale rules**

- Delete the instruction file and add back only what the new model still gets wrong. Relayed as Boris Cherny’s advice, with a claim that roughly 80% of the Claude Code team’s own system prompts were removed that way. [@stretchcloud](https://x.com/stretchcloud/status/2111630593679368210) (18 Oct 2026). Talk retelling, not a dataset.
- Same point from users: old rules written for a weaker model now slow the new one. [@bologabriele](https://x.com/bologabriele/status/2125948928462061586) (26 Nov 2026), [@PublicAI_](https://x.com/PublicAI_/status/2124849063632003443) (23 Nov 2026), [@trsgogogo](https://x.com/trsgogogo/status/2126426569896071460) (27 Nov 2026: every six months, delete `CLAUDE.md`, skills, and hooks).
- Lydia Hallie (Claude Code), retold: ask whether the model needs the line, or can read it from the repo. [@omnisc1entt](https://x.com/omnisc1entt/status/2126450307878244403) (27 Nov 2026).
- HTML comments in `CLAUDE.md` are not sent to the model, so maintainer notes can live in the file without token cost. [@mhimt](https://x.com/mhimt/status/2126338230304030747) (27 Nov 2026). Docs summary.

**Memory systems**

- Claude Code’s own memory loads the first 200 lines as an index. “Better models keep less.” Telling the model how to write memory hurt. A periodic merge-and-delete pass (“dreaming”) is what keeps it from rotting. [@PawelHuryn](https://x.com/PawelHuryn/status/2118826877414490386) (6 Nov 2026). He is retelling their tests.
- claude-mem: small index, full notes through three MCP tools, and a measured ~5,200-token tool-schema tax. Also a warning that the default install offers a cloud trial and anonymous telemetry. [@sketch942001](https://x.com/sketch942001/status/2126447420762447925) (27 Nov 2026).
- Codex memories: the only fresh product note is “expanded memory hook support” in an unreleased alpha. [@StatsWire](https://x.com/StatsWire/status/2126389761443021056) (27 Nov 2026). No measurement of how many tokens those memories add.
- Grok: `/memory` in v1.0.33 deletes real notes and refuses to delete the generated index. [@XFreeze](https://x.com/XFreeze/status/2100070332082389392) (16 Sep 2026). No `memory-v2` discussion on X in this search.
- Git-native memory pitches claim 0 tokens in the system prompt and an `AGENTS.md` capped at 400 tokens. [@fngly343149](https://x.com/fngly343149/status/2126248924067528722) (27 Nov 2026). Vendor claim.
- Compaction as memory is a security hole in some harnesses. A fake `[User]:` line in a doc landed in memory 87–100% of the time in 5 tools that flatten the chat (Cline, Aider, OpenHands named). It worked 0% of the time against Codex and Gemini CLI, which keep user and tool roles as separate fields. A prompt-only rule stopped 13–25%. A code check caught 77%. [@sjandfun](https://x.com/sjandfun/status/2126451044901400883) (27 Nov 2026), citing Pixee and a Cloud Security Alliance note. Third-party test, sample sizes not in the post.

## 3. Other high-leverage ideas for a multi-provider setup

- **Harness beats model on the bill.** AA held the model fixed across 13 harnesses at max reasoning. GPT-6 Luna: $0.11/task and 5.6M tokens in Pi vs $0.62 and 26.9M in Muse Code (5.4×). Score only moved 17.2 to 25.9. Codex was that model’s best harness (25.9) and GLM-5.3-Flash’s worst (27.8 vs 38.9 in Claude Code). Inside Codex, GLM-5.3-Flash switches to terse “telegraphic” reasoning. [@SagarGadhvi9](https://x.com/SagarGadhvi9/status/2125281912965267945) (24 Nov 2026). AA numbers, default settings, so a tuned setup can differ.
- **Defaults, not a smarter model, were Uber’s cut:** compact early, Medium effort, cheap subagents, longer cache on the parent than on the children, do not mount every MCP schema, batch calls into one script. [@undefinedKi](https://x.com/undefinedKi/status/2116247505193050474) (30 Oct 2026).
- **Defer tool schemas.** Search tool, or `defer_loading`, or “tools as files.” The 85% and 98.7% figures are Anthropic writeups. The Grok Build half-context fix is the one Grok-specific measurement. See section 1.
- **Shell output and screenshots dominate ongoing context** in a real Claude+Codex history, more than MCP results. [@btsouth](https://x.com/btsouth/status/2125431487461740662) (25 Nov 2026). A related plugin claims 315 KB of tool output becomes 5.4 KB, and 700 KB of file reads becomes 3.6 KB, and lists Gemini CLI and Codex among 17 platforms. [@thisguyknowsai](https://x.com/thisguyknowsai/status/2112484767812010031) (20 Oct 2026). Plugin marketing, not an audit.
- **Do not share one prompt cache across providers.** Nothing on X measured a cache hit after a delegate from Claude to Codex to Gemini. Caches are per provider. The practical reports are the opposite: Neon’s cache dies in about a minute of idle or tool time ([@bebuu_ai](https://x.com/bebuu_ai/status/2123414001912861014)), while Codex is described as holding ~24 h ([@shneural](https://x.com/shneural/status/2123404925677994366)). A parent that waits on another provider will miss Neon’s cache on the next turn. That last sentence is an inference from those two reports, not a test someone posted.
- **Skills over MCP load only when needed**, and are an open MCP extension, so they are not Claude-only. [@AITensibility](https://x.com/AITensibility/status/2126256235682906294) (27 Nov 2026), retelling [@ClaudeDevs](https://x.com/ClaudeDevs/status/2125289794871963935) (24 Nov 2026). The 85% tool-search reduction in that post is Anthropic’s.
- **Plan on the strong model, grind on the cheap one, and give the checker a clean context.** That shows up independently for Claude ([@mirku21](https://x.com/mirku21/status/2126420893736006025), 27 Nov 2026, prompt-template, the 14k / 400k split is not a measurement), for Antigravity planner vs executor ([@ChuckBass8808](https://x.com/ChuckBass8808/status/2126438273874882608)), and for Hydrogen workers under a Neon orchestrator ([@yuhasbeentaken](https://x.com/yuhasbeentaken/status/2126353585369666029)).

## Nothing found / unverified

- No X posts in Sep–Nov 2026 describing a lean or stripped **child profile** for Antigravity, Gemini CLI, Codex, or Grok. The delete-your-rules advice is about Claude Code’s `CLAUDE.md`.
- No measured `reasoningEffort` or `thought_level` multiplier (tokens or quota per step) for Codex or Grok. Neon vs Hydrogen is one user’s single task, not a curve.
- No confirmation of the 9% / 3× Neon cache story from Google. The product lead asked for a DM.
- No settled answer on whether Gemini CLI reads `AGENTS.md` natively or only `GEMINI.md` plus `context.fileName`.
- No posts on `~/.grok/AGENTS.md`, Grok memory-v2 token cost, or symlink practice for Grok.
- No posts on `trigger: model_decision` by that name. The working version of that idea on X is skill name-plus-description always loaded, body on demand, and folder-scoped rules.
- No measurement of prompt-cache survival across a provider switch or a delegate. One unanswered question on whether changing Neon/Carbon effort mid-session drops the cache.
- Bee’s 1.3× / 1.8× context-length quota and StatsWire’s Codex alpha “10× context” are pre-release claims.
- The 150k→2k and 98.7% figures circulating on Grok Bot posts are Anthropic’s, copied into prompts.
- I did not open the linked blogs or docs, so anything marked “docs summary” or “Uber / AA / Anthropic, as retold” is only as strong as that post.
