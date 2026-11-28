# Context-management techniques: X research on bisect, path-scoped rules and MCP filtering (2026-11-28)

Grok (`grok-4.7`) searched X on 2026-11-28 for public examples of coding-agent token savings via `git bisect run`,
Claude path-scoped rules (`.claude/rules/` `paths:`), and filtering structured MCP results in code. Preserved verbatim
from Grok's reply (nothing removed). Every claim below is `reported` or `opinion` until backed by primary documentation
or independent measurement.

---

X search worked. Public posts back one measured result for filtering MCP payloads in code. They do not back a measured token saving for `git bisect run` or for Claude path-scoped rules.

## git bisect run

No post found that reports model-token counts for `git bisect run`. What is public is anecdote or a restatement.

- **Anecdote, no token count.** Mitchell Hashimoto (@mitchellh), 20 Jan 2026. He asked an agent to `git bisect` a CI break from that morning and said it fixed the commit. https://x.com/mitchellh/status/2013679011315196144
- **Self-reported demo, not model tokens.** Heisbei (@Heisbei02), 26 Nov 2026. `firstbad` drives `git bisect run` with a browser check, then gives Claude Code only the culprit diff. On a 40-commit shop with one planted bug: 11 runs, about 22 minutes, about 200 Kane credits. Those are Kane credits, not LLM tokens. https://x.com/Heisbei02/status/2125790166484652176
- **Opinion, no numbers.** Gagan (@gagu_sh), 26 Nov 2026. Reply that `git bisect run` with one end-to-end test is the right oracle, and that small agent commits make the search sharper. https://x.com/gagu_sh/status/2125819503112286645
- **Guide restatement, no measurement.** meng shao (@shao__meng), 11 Jun 2026, quoting Simon Willison (@simonw). The guide lists "use git bisect to find when this bug was introduced" as a pattern, and treats subagents, not bisect, as the context saver. https://x.com/shao__meng/status/2064867069167087782 and https://x.com/simonw/status/2064802267983946135
- **Summary of someone else's workflow.** yibie (@yibie), 7 Aug 2026. Chinese summary of Addy Osmani: agents are patient at walking history with `git bisect`. No token figure. https://x.com/yibie/status/2085536770758996033
- **Promo, human bisect.** slash1s (@slash1sol), 13 Jun 2026. Recap of a Google engineer's talk: about 13 steps through 10,000 commits. Mentions agent commit volume. No agent token measurement. https://x.com/slash1sol/status/2065585744513499220

Nearby, not the same claim: JFrog (@jfrog), 27 Nov 2026, says agents waste tokens re-reading `.git` history and long grep dumps. https://x.com/jfrog/status/2126240871918059807. Kentaro Wada (@wkentaro_), 13 Aug 2026, says `git-hunk` lets an agent stage hunks by ID "with the fewest tokens possible" so history is easier to bisect. https://x.com/wkentaro_/status/2087844928156221788

## Claude path-scoped rules

No before/after token study. Posts describe the mechanism or repeat the docs.

- **Tip, no numbers.** Kornel (@indiekornel), 23 Nov 2026. A `.claude/rules/` file with `paths:` loads only when Claude touches matching files. https://x.com/indiekornel/status/2124942066220204091
- **How-to, no numbers.** Kunal (@kunal_twts), 24 Nov 2026. Same `paths:` frontmatter for `src/api/**/*.ts`. https://x.com/kunal_twts/status/2125333712586101039
- **Advice inside a thread, not a finished measurement.** DuoBitAI (@DuoBitAI), 27–28 Nov 2026. Recommends one path-scoped rule instead of pasting the same block into many nested `CLAUDE.md` files. The sizes in the thread (a 488 KB root file, a 65 KB slice, about 22k tokens versus about 220k) are the other person's files, offered as what to re-check with `/context all` after a split. https://x.com/DuoBitAI/status/2126414939661250600 and https://x.com/DuoBitAI/status/2126521433849311408
- **Docs restatement.** Michie (@mhimt), 27 Nov 2026, and Towards AI (@towards_AI), 26 Nov 2026. Both say directory-only rules belong in `.claude/rules/` with `paths`, and that longer always-loaded files cost more context. No measured delta. https://x.com/mhimt/status/2126338230304030747 and https://x.com/towards_AI/status/2125798615637545278
- **Unverified self-report, different mechanism.** Argona (@Argona0x), 23 Mar 2026. Nested per-directory `CLAUDE.md`, not `paths:` frontmatter. Claims 70–80% less context bloat, plus about $2,400 and about 60% less debugging time. No method or raw counts in the post. https://x.com/Argona0x/status/2036106517678624820
- **Feature note only.** Daniel San (@dani_avila7), 10 Dec 2025. Announces `.claude/rules/`. No savings figure. https://x.com/dani_avila7/status/1998600207487918380

## Filtering structured MCP results in code

- **Self-run experiment, backed by a writeup.** Kelly Norton (@kellegous), 27 Nov 2026. Optional `jq` on MCP JSON before it enters context, tested two ways (a `jq` parameter, and a `jq` meta-tool) on quantitative weather questions. Accuracy unchanged on his set. Total input tokens down 85.5% and 81.9% for Claude Opus 5.5, and 91.7% and 93.3% for GPT 6.2 Sol. The post rounds this to 82–93%. Code is linked from the post. https://x.com/kellegous/status/2126302434305749037 and https://kellegous.com/j/2026/11/27/filtering-mcp-tools-with-jq-pt2/
- **Vendor example, then widely copied.** Anthropic's 4 Nov 2025 engineering post, "Code execution with MCP," is the source of 150,000 tokens down to 2,000 (98.7%), including filtering a 10,000-row sheet in code so about 5 rows reach the model. https://www.anthropic.com/engineering/code-execution-with-mcp. X posts that repeat 150k to 2k are citing that example, not a new run:
  - Kieran Klaassen (@kieranklaassen), 5 Nov 2025. https://x.com/kieranklaassen/status/1985940599560945753
  - Dhanush N (@Dhanush_Nehru), 28 Nov 2026. Also names Cloudflare Code Mode. https://x.com/Dhanush_Nehru/status/2126498561995026680
  - Skills Studio (@sumika45379), 27 Nov 2026. Attributes it to Anthropic's blog. https://x.com/sumika45379/status/2126461066066694451
  - slash1s (@slash1sol), 27 Nov 2026. A prompt pack of the same six steps. https://x.com/slash1sol/status/2126409413397245968
- **Same numbers, weak attribution.** 观心 (@8x8x8_x), 27 Nov 2026, claims a personal sandbox at 148,200 to 1,940 tokens (98.7%) across 42 MCP servers. The figure matches Anthropic's published example closely enough that it should not be treated as an independent measurement. https://x.com/8x8x8_x/status/2126375503380586756. Corki (@Corki_0x), 23 Nov 2026, repeats 150,000 to 2,000 and credits "xAI engineers." https://x.com/Corki_0x/status/2124890507700469895
- **Practice note, no own before/after.** abhishek tatikella (@tatikella52515), 28 Nov 2026. In Pi, MCP tools stay out of the prompt. A QuickJS script gets the full result, filters it, and returns a short value. Direct MCP results over 20 KB have the middle cut for the model. https://x.com/tatikella52515/status/2126534128505929740
- **Different mechanism.** Rohan Paul (@rohanpaul_ai), 11 May 2025. RAG-MCP pre-selects tool schemas. The paper's claim is over 50% fewer prompt tokens and tool-selection accuracy from 13.62% to 43.13%. That filters the catalog, not a structured tool result. https://x.com/rohanpaul_ai/status/1921489203726303374

X posts are opinion until another source backs them. Norton's blog backs his jq numbers. Anthropic's blog backs the 98.7% code-execution example. Bisect and path-scoped rules stay anecdotes and documentation restatements.
