# New thread per PR vs. reusing an old thread: X research (2026-11-27)

Grok (`grok-4.7`, high reasoning effort) searched X on 2026-11-27 for 2026 posts comparing a fresh thread/session per
new PR against reusing an existing thread across multiple PRs. Every claim below is `reported` or `opinion` until backed
by primary documentation or local measurement.

---

**Verdict: CONDITIONAL YES.** A fresh thread per new PR is the better default once the old thread is long, idle past the cache window, or about to change tasks. It is worse for a short follow-up while the cache is still warm. No post on X measured “one new thread per PR” against “one thread, many PRs” as a controlled test.

## 1. Cost and cache

**Self-reported logs (not peer-reviewed studies):**

- @quxiaoyin, 6 Aug 2026, https://x.com/quxiaoyin/status/2085408811104534754 — His own Claude Code month: 110 sessions, 46,580 turns. Same message while working: 1x. After coffee: 1.3x. After a call: 1.6x. After lunch: 13x, because the cache expired at 60 minutes and the next turn rebuilt the whole session. A new session cost 4.5x. His rule: summarize before you leave, paste that into a fresh window after the break. A reply from @Santosh74038967 the same day says Claude Code mostly uses the 1-hour cache write, with a 5-minute option, and that Codex behaves differently: https://x.com/Santosh74038967/status/2085476825971769500

- @cprkrn, 10 Jun 2026, https://x.com/cprkrn/status/2064569493142073479 — His own token analysis: 96% of spend was the model re-reading conversation history. A message in a 500k-token chat cost 8x the same message in a fresh chat. He built an automatic handoff. The post does not show the query or the raw log.

- @PawelHuryn, 21 Nov 2026, https://x.com/PawelHuryn/status/2124143958120968366 — On his own long Claude sessions, 55% to 85% of usage was the agent re-reading its own history. His math: double the length and the re-read cost is about 4x, because every turn re-reads the whole conversation. He now starts each phase in a fresh session and keeps state in `STATUS.md`. Quoted and restated by @stretchcloud the same day: https://x.com/stretchcloud/status/2124169213791072630

- @dannygnc, 25 Nov 2026, https://x.com/dannygnc/status/2125436535310876961 — One team’s week, as he reports it: history resent past 200k tokens was $360 of avoidable spend (`/clear` between tasks). Idle cache expiry was $532. Failed tool calls were $68. Same claim repeated later that day: https://x.com/dannygnc/status/2125702523197497803

- @chenYijl, 27 Nov 2026, https://x.com/chenYijl/status/2126204614525784257 — One developer’s API-list accounting: 45 merged PRs in a week, $343.78 ($7.64 each). A month was $8,360, of which Claude Code was $8,056. One session he forgot to stop ran 373 hours, 64,039 tool calls, and $3,465 (47% of the month). That is a cost of an unbounded session, not a fresh-vs-warm A/B.

**Pricing explanations, not logs:**

- @arena, 18 Aug 2026, https://x.com/arena/status/2089755251134083578 — Summarizes cache math from @evan_a_frick: cache hits about 10% of full price; a “hi” after the cache goes cold can cost about $1; cost still grows with the square of context; Claude Code and Codex may compact around 200–300k rather than a 1M window.

- @Iron_Adamant, 21 Nov 2026, https://x.com/Iron_Adamant/status/2123998873840456124 — His sketch: no tools, 119 × 91k = 10.8M tokens; with tools, 201 × 112k = 22.5M. Raw tokens doubled. Because re-reads were cached, the bill rose 56%.

- @arpit_bhayani, 16 Nov 2026, https://x.com/arpit_bhayani/status/2122172874345914759 — Opinion: every tool call resends history; cache is cheaper, not free; extra turns from a weaker start can cost more than the tokens you trimmed. Track cost per finished task.

- @omprashantjain, 21 Nov 2026, https://x.com/omprashantjain/status/2124132100429557795 — Opinion with a diagram: a longer session has a larger cached prefix, so staying warm gets cheaper per step only while the prefix still matches.

- @daniel_mac8, 10 Oct 2026, https://x.com/daniel_mac8/status/2108903166884368408 — Says Claude Code subagent cache TTL defaults to 5 minutes. @rryssf, 13 Oct 2026, https://x.com/rryssf/status/2109940688595153234 — Says stretching it to 1 hour is reported to save up to 25% when subagents sit idle, and can raise the bill if they do not, because the 1-hour cache write costs more. Neither post shows the underlying sessions.

- @Snak1i, 27 Nov 2026, https://x.com/Snak1i/status/2126418602593026273 — On Anthropic’s message-threads beta (`message-threads-2026-08-12`): median time-to-first-token down about 35% at 500k–1M tokens, but token usage stays the same (full conversation still billed at standard rates), it does not cache for you, and a thread expires 5 minutes after the last request. That is an API latency feature, not a cheaper coding session.

**Not measurements:** @noisyb0y1, 11 Aug 2026, https://x.com/noisyb0y1/status/2087218720594706679 (“84% fewer tokens, 39% higher accuracy,” “1,000,000 vs 40,000”) is a viral claim with no log. @DoThisLife, 28 Nov 2026, https://x.com/DoThisLife/status/2126473164813119921 — debunks a similar “engineer PDF” and says its numbers were labeled illustrative, not measured. The durable point in that post is the mechanism: each turn reloads the transcript.

**Break-even, from those logs only.** No post gives a universal turn count. The thresholds people actually used:

| Situation | What the logs say | What to do |
|---|---|---|
| Still working, cache warm | Continue ≈ 1x, new session ≈ 4.5x (@quxiaoyin) | Stay |
| Idle ~60 min on a long Claude Code thread | Continue ≈ 13x, new session ≈ 4.5x | Fresh thread, summary written before you left |
| History past ~200k | $360/week called avoidable by `/clear` (@dannygnc) | Clear or new thread between tasks |
| ~500k chat | One message ≈ 8x a fresh chat (@cprkrn) | Do not keep that thread for the next PR |

Compaction does not remove the shape of the cost. @gkurttech, 25 Nov 2026, https://x.com/gkurttech/status/2125487431994745167 — compact while the cache is still warm so the cold rebuild is smaller. @PawelHuryn says compaction still lives inside one session of one tool, so it does not replace a fresh phase.

## 2. Quality

**Reported study, secondhand (not opened directly in this search):**

- @ApollonVisual, 27 Nov 2026, https://x.com/ApollonVisual/status/2126231381797642410 — Summarizes Paradigm 3, “Swarms Do Not Yet Raise The Roof,” 24 Nov 2026, https://www.paradigm3.org/research/swarms. Not peer-reviewed: >1,100 runs, 4 long-horizon tasks, >80 configurations, equal token budgets. Teams of 8 finished ~6x sooner and scored ~7% lower; swarms scored lower in 54 of 70 comparisons. At a 32M-token budget on one task: uninterrupted single agent 83.7%, 8-agent team 93.2%, same agent restarted as fresh sequential sessions 97.0%. @fawadhsdev, 26 Nov 2026, https://x.com/fawadhsdev/status/2126068258558386677 and @barnhartguy, 27 Nov 2026, https://x.com/barnhartguy/status/2126436913271746836 cite the same 97.0% vs 93.2% result.

**Opinions, no study of multi-PR thread reuse:**

- @MohakBajaj5, 25 Nov 2026, https://x.com/MohakBajaj5/status/2125514986219069938 — Recommends a fresh session per task plus a 10-line handoff instead of riding auto-compact.
- @kakarot80t, 27 Nov 2026, https://x.com/kakarot80t/status/2126453887439958496 — Link roundup pointing at context-rot docs.
- @PromptSmithAI, 27 Nov 2026, https://x.com/PromptSmithAI/status/2126451091374272771 — Opinion: piping full history into subagents causes context rot.

**Related measured trap (stale docs / handoffs):** @Voxyz_ai, 25 Oct 2026, https://x.com/Voxyz_ai/status/2114446907355648062 and @HowToPrompt__, 5 Nov 2026, https://x.com/HowToPrompt__/status/2118225768237215979 — citing the 40-project stale-docs study where agents obeyed outdated written instructions unless told to follow the code. A stale handoff note risks the same failure mode.

**Gap:** nobody on X in this search published a log of stale diffs or multi-turn drift specifically from reusing one coding-agent thread across several PRs.

## 3. When staying is the better call

- The PR is a few more turns on the same files and the cache is still warm: continuing ≈ 1x vs. cold start ≈ 4.5x (@quxiaoyin).
- Cached re-reads soften raw token growth (@Iron_Adamant, +56% bill on 2x tokens; @omprashantjain).
- A reset that forces repo rediscovery can add turns that cost more than the trimmed history (@arpit_bhayani).
- @RGWorldz, 28 Nov 2026, https://x.com/RGWorldz/status/2126490563050676707 — opinion: skipping a handoff note makes resets painful.

## 4. Automatic new thread per PR

People mostly trigger fresh threads by hand or via product commands (`/fork`, `/btw`, `claude -w`, `codex fork`, `/batch`). No X post described an agent automatically spawning a fresh thread for every new PR, nor recursive-spawn / idle-wakeup bugs:

- @amorriscode (Anthropic), 21 Aug 2026, https://x.com/amorriscode/status/2090861991187620115 — Claude Code desktop suggests a new session for side work spotted mid-session.
- @Voxyz_ai, 13 Nov 2026, https://x.com/Voxyz_ai/status/2121380177020157963 — `/btw` and `/fork` in Claude Code Projects.
- @johnnynelai, 26 Nov 2026, https://x.com/johnnynelai/status/2126028716216062436 — Claude `/fork` with worktree; Codex `codex fork --last`.
- @danpdc, 26 Nov 2026, https://x.com/danpdc/status/2125882281097117749 — `claude -w <worktree>`.
- @beamnxw, 27 Nov 2026, https://x.com/beamnxw/status/2126452489750716439 — `/batch` per-worker worktrees.
- @FullstackVibe, 26 Nov 2026, https://x.com/FullstackVibe/status/2125844867502989577 — Per-worker worktrees with handoff.
- @obie, 4 Oct 2026, https://x.com/obie/status/2106767245275967651 — Plugin writing a handoff at 50 minutes while cache is warm (`github.com/obie/auto-handoff`).
- @ishuagra02, 10 Oct 2026, https://x.com/ishuagra02/status/2108891293749526823 — `HANDOFF.md` rule in `CLAUDE.md` (anecdote).
- @theo, 1 Oct 2026, https://x.com/theo/status/2105452015942095306 — T3 per-thread directories under `~/.t3`.

**Thin or empty:** Antigravity / Gemini CLI had no on-point posts in this search; nothing found on automated T3 thread spawning or idle-thread wakeups.
