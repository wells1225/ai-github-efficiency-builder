# Repo layout for coding agents: second round on X

Raw research result, saved as returned on 2026-11-24. Researcher: grok-4.7. Not edited. This round dug into the leads from round one and searched topics round one did not cover. Claims found only on X are opinion until another source backs them.

---

X search worked. I read about 80 posts and 11 linked pages (two papers and their data file, two public AGENTS.md files, the Codex instructions page, the AAIF write-up, Movesafe’s quickstart, and one public repo).

## Part 1

### 1. @tvykruta’s restructuring account — holds with limits

The 30 Sep 2026 thread is his own before/after on a private app, with the rules pasted in the thread and no published logs. The direction of the story is consistent with what he posted later. The 9× and 7× figures are not something anyone else has reproduced in public.

What he claims, in https://x.com/tvykruta/status/2105302045573959697 (30 Sep 2026) and the replies in that thread: one pricing feature took 5 days, 49 pull requests and about $640. After the rules, "$7 → $1 per PR. 9× fewer tokens." He calls the numbers "measured, not estimated: a before/after analysis of the codebase and the agents' own session logs." The mess he describes is 31 files of price logic (he also says 32), a 2,916-line main file, then 11 behavior-preserving steps into one package. The method is in https://x.com/tvykruta/status/2105305072817037419: audit with numbers, one behavior-preserving pull request per step, tests green, a check that fails if the mess returns, a second audit, and only then the feature.

The repo for that app is not public. The only public repo of his I opened, https://github.com/tvykruta/imessage-agent-bridge (linked from his 14 Oct 2026 post https://x.com/tvykruta/status/2110386433932378521), is a different project. It has `AGENT_SETUP.md`, not this AGENTS.md. On 5 Oct 2026 he wrote, in https://x.com/tvykruta/status/2106983478059278707, "My agents.md is 217 lines, but it fans out to many other dedicated rules files." The excerpt in https://x.com/tvykruta/status/2105306637904863429 is the principles section, not the whole file. Its hard rules are: one owning module per domain, never duplicate logic (move, switch the old caller, tests green, then the new use), no business logic in templates, refactor and change in separate diffs, and "A prompted 'never' alone does not protect the SoT; a rule that matters is enforced by CI or a hook."

Critics in that thread: @codeandsleep (30 Sep 2026) asked why the posts are written with AI. @smusamashah (2 Oct 2026, https://x.com/smusamashah/status/2106074639805845981) wrote "This is word salad and reads like uncle bob." @garvan_k (30 Sep 2026) said people in the industry recommend removing this level of detail, and that he keeps a similar file anyway. I found no reply that re-ran the measurement.

Since then he has kept the same process and added numbers of his own. On 2 Oct 2026, https://x.com/tvykruta/status/2106138620780261842, a fresh reviewer with only the diff found 17 of 23 issues where the session that wrote the code found 3 of 23. One rule, "Write your test inputs before you read the implementation. Then run them," moved logic-bug recall from 4 of 8 to 7 of 8 on the same model, "$0.72 → $0.74". On 5 Oct 2026, https://x.com/tvykruta/status/2107177934314225902, he reports 112 pull requests merged in five days, design-doc review falling from about $188 to $35 on the same five docs, and one session whose context grew from 75k to 782k tokens, with "57% of the bill" spent re-reading context. On 5 Oct 2026, https://x.com/tvykruta/status/2107196376488845768, he says a refactor in its own pull request has a "100% success rate so far," proved by tests whose assertions were not edited and by diffing real outputs.

Limit: one person, one private codebase, no logs, no second team.

### 2. @tianrendong’s empty instruction file — holds with limits

https://x.com/tianrendong/status/2124948368569069678 (23 Nov 2026), engineer at Ramp, running "a massive monorepo with all 350+ repo skills turned off and an empty AGENTS.md." After two weeks and about 8,000 turns (https://x.com/tianrendong/status/2124948479776821565): about 60k tokens saved per turn from "350+ skills descriptions + 40kb AGENTS.md", about 25% lower spend "calculated from cache hit/miss logs", tool-call error rates flat, and he "didn't have to correct the agent more often." An accidental comparison (https://x.com/tianrendong/status/2124949796071030869): one worktree still had the old file, and "the ~600 turns in it didn't feel any smarter."

@yagcidev asked whether the 25% was fewer skill descriptions or fewer tool calls. I did not find an answer. He points at @austospumanto as having predicted it. I found no one else posting their own trial. What he keeps: the local docs already in the repo. He says "the latest models are really good at reading local docs when they need them." The post had 12 likes and 548 views when I opened it.

Limit: one author, one repo, no public logs, and the "not dumber" half is his judgment plus a flat error rate.

### 3. Claude Code, Codex, Cursor, Amp — holds, with one personal claim that is only a sentence

Claude Code, @trq212, 24 Jul 2026, https://x.com/trq212/status/2080710971228918066: "We removed over 80% of Claude Code’s system prompt for models like Claude Opus 5 and Claude Fable 5 with no measurable loss on our coding evaluations." For a repo file: "spend most of the tokens on gotchas inside of the codebase." Leave out "the obvious." Put verification in a skill. He points `/doctor` at oversized skills and CLAUDE.md files. The same post is on the Claude blog from his reply the same day.

Later from him: 25 Aug 2026, https://x.com/trq212/status/2092302273099796842, model families are not interchangeable, so Claude Code keeps a system prompt per model; you can `@Agents.MD` from CLAUDE.md. 11 Sep 2026, https://x.com/trq212/status/2098531560643539440, `claude plugin eval init` runs a plugin with and without itself. 18 Sep 2026, https://x.com/trq212/status/2101009392611278961, from version 2.1.277 Claude Code reads AGENTS.md when a folder has no CLAUDE.md. 23 Sep 2026, https://x.com/trq212/status/2102899188505965013, people are replacing the default planner with their own planning skills. 28 Sep 2026, https://x.com/trq212/status/2104608785696440510, "it's basically impossible for someone to just 'show you their prompt' now, because everything is about references, skills and examples."

@bcherny, 11 Sep 2026, https://x.com/bcherny/status/2098217573276131577: production code needs "lots of lint rules, lots of tests," agent end-to-end tests, daily fuzzers, and automated review. If the code is still poor, "Invest in your CLAUDE.md and skills to succinctly teach Claude how to work in your codebase," or "have Claude fix accumulated debt and rewrite your codebase." On 6 Oct 2026, https://x.com/bcherny/status/2107591357170950471, answering a request for his system instructions: "I don't use a custom CLAUDE.md." He did not say why in that thread. @JuddLehmkuhl asked; I did not find his answer. On 22 Oct 2026, https://x.com/bcherny/status/2113333859907747871, Agent Skills went to the Agentic AI Foundation, "70+ agent platforms." On 24 Nov 2026, https://x.com/bcherny/status/2125292098165612585, MCP servers can ship skills.

A line circulating as his, "Every six months, delete your CLAUDE.md. Delete your skills. Delete your hooks," is in https://x.com/0xSanabi/status/2123830407321755696 (20 Nov 2026), attributed to a 135-second video. Searching his account did not turn up that sentence. @kevintheo, 10 Nov 2026, https://x.com/kevintheo/status/2120184515637153863, tried a deletion on a tiny test: 47,305 tokens on the first request with his files, 28,115 without, and says the test "says nothing about today's models."

Codex. @embirico, 6 Aug 2026, https://x.com/embirico/status/2085403842154512454: "Started with AGENTS.md, Agent Skills, then .agents config, now Agent Plugins." The current instructions page, https://learn.chatgpt.com/docs/agent-configuration/agents-md, says Codex reads one file per directory from the repo root down to the working directory, later files overriding earlier ones, and "stops adding files once the combined size reaches" `project_doc_max_bytes`, "32 KiB by default." The files closest to the work are the ones cut. @thsottiaux, 29 Sep 2026, https://x.com/thsottiaux/status/2104983716049379472, calls ChatGPT Space "AGENTS.md taken one step further." That post is about shared notes, not repo layout.

Cursor. In this window, @cursor_ai’s posts I pulled were model announcements, not instruction-file guidance. I did not find a Cursor-team post on what to delete.

Amp. @sqs, 21 Nov 2026, https://x.com/sqs/status/2123939091654021165: "The most common reason why people get weird behavior like this is that they have dozens of skills pulling the agent in all different ways. You should remove all such skills." On 18 Nov 2026, https://x.com/sqs/status/2122881764167479398: "We've found skills should come from you and your company." I did not find an Amp post on folder layout.

### 4. One repo or many — holds with limits

@mattzcarey, 11 Nov 2026, https://x.com/mattzcarey/status/2120532474761556475 (Cloudflare): "unless you are willing to reimplement git / github (like anthropic), I think monorepos are an antipattern." His alternative is a small repo plus "a lovely cli/mcp" so agents search the org without a checkout. He thanks an internal CLI and says the small repo is "so much faster and less compute," with no number. @jachands disagreed the same day for a small team.

@jpschroeder, 11 Nov 2026, https://x.com/jpschroeder/status/2120551435788222485, describes the parent folder:

```
project-coordinator
├ repo-1
├ repo-2
├ repo-3
└ repo-4
```

The parent holds a `.gitignore` and an AGENTS.md. The real repos are listed in both, gitignored, and cloned when someone says "set this up." Agents run from the parent. "Worktrees can copy as many or as few of the sub-projects as needed." He ends with "So far so good, but honestly, almost anything works these days so we hold our beliefs loosely." @aienginerd replied "bro replaced a monorepo with a monorepo." @night_shift said they do the same and asked about git submodules. Nobody in the thread reported a broken build, a lost history, or a bad merge.

### 5. Moving files — holds with limits

The tool in https://x.com/lickollariendri/status/2123329256960717052 (19 Nov 2026) is real. Its quickstart, https://endrilickollari.github.io/movesafe/, says `npx movesafe mv … --dry-run` plans a TypeScript move from the import graph, then a second run writes. The post’s rule is `plan_move` produces a `planHash`, and `apply_move` recomputes and refuses if the hash differs. It repairs static imports, type imports, re-exports, `require()`, and literal dynamic imports, including path aliases and workspace packages. "Computed import specifiers are reported as diagnostics and are never guessed." The npm page in search said weekly downloads were 0. Replies on the post are agreement, not field reports.

I did not find a detailed public account of an agent rearranging an existing repo: which command, which imports broke, what happened to history or open branches, and how it was checked. The nearest reports are small. @saurabh_swapnil, 19 Nov 2026, https://x.com/saurabh_swapnil/status/2123375423556129191: an agent renamed a util mid-task and three imports pointed at nothing. @popcoornft, 19 Nov 2026, https://x.com/popcoornft/status/2123337865748725773: an overnight tidy archived the current contract set and broke links in three open reports. @anhnt256, 20 Nov 2026, https://x.com/anhnt256/status/2123845130092450209: worktrees across four branches, and "merge conflicts corrupted our test DB."

### 6. The 79-of-80 claim — holds

The source is a paper, not a thread. Xiaonan Xu and Wenjing Wu, submitted 22 Oct 2026, https://arxiv.org/abs/2610.30571. Data and scripts: https://github.com/WenJing95/stale-docs-data. The numbers file, https://raw.githubusercontent.com/WenJing95/stale-docs-data/main/numbers.md, matches the post.

They built 40 Python projects where the instruction file asks for one convention and the code uses another. Claude Code 2.1.302 on Claude Sonnet 5.5, and Codex CLI 0.163.0 on GPT-6 Sol, 320 runs. Without the sentence, 90.1% of choices (649/720) followed the file. Claude Code named the conflict in 79 of 80 runs and still followed the file in 86.2% of the choices inside those runs. Codex rewrote existing code in 20 of 80 runs; its tests passed in 63 of 80 runs without the sentence and in 80 of 80 with it. The sentence, placed in the user-level file, moved "follow the code" from 4.2% (30/720) to 91.4% (658/720). @Voxyz_ai’s 25 Oct 2026 post, https://x.com/Voxyz_ai/status/2114446907355648062, matches those figures. His later post on 23 Nov 2026, https://x.com/Voxyz_ai/status/2124880659889938848, repeats them and adds a prompt that asks the agent to review the repo’s instruction files and change nothing yet.

Limits, from the same numbers file: this is constructed convention conflicts, not a stale path in a real company repo. Money and config were the exceptions. Claude Code followed the stale file on config only 13.6% of the time (3/22) and followed the code 86.4%. Two agents, two named versions. Seven runs edited the instruction file itself.

## Part 2

### A. Real instruction files

A short file that names commands, generated files, and where the longer rules live is what the measured work supports. A directory tour is what the ETH paper says does not help.

Supabase’s root file, https://raw.githubusercontent.com/supabase/supabase/master/AGENTS.md, is 77 lines. It has the package manager and Node version, a table of apps and ports, the commands (`pnpm dev:studio`, `pnpm test:studio`, `pnpm typecheck`), a ban on hand-editing generated files, and one line that the skills in `.agents/skills/` are the source of truth and should be loaded when relevant. It leaves out style essays and product history.

Next.js’s canary file, https://raw.githubusercontent.com/vercel/next.js/canary/AGENTS.md, is 531 lines. It is a development guide: monorepo map, the exact test command per bundler and mode (`pnpm test-dev-turbo`, `pnpm test-start-webpack`), `pnpm new-test --args` so an agent can create a test without a prompt, a ban on `as any` with the lint rule name, and pull-request rules including a required `### Verification` section. It includes the directory tour the Claude Code post says to skip. I did not measure its byte size. Codex’s own docs say a combined chain past 32 KiB is cut from the end.

OpenAI’s Codex repo file, at the blob the search returned (https://github.com/openai/codex/blob/rust-v0.135.0/AGENTS.md), is 221 lines and is a Rust lint brief: inline `format!` args, collapse `if`, and "Never add or modify any code related to `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR`." I opened that blob page through search results, not the raw file.

The AAIF site’s file is described as about 50 lines. Shown, on a small eval: https://aaif.io/blog/devex-for-your-agents-how-agents-md-cut-token-usage-and-cost (Ana Jiménez Santamaría, 11 Nov 2026). Claude Code, Opus 5.5, three tasks, three runs, 18 runs total. With the file: 9 of 9 fully correct versus 7 of 9, tool calls 13.1 to 8.7, tokens about 447k to 285k (−36%), cost about $0.38 to $0.28 (−26%). She writes that once the file was present, "the agents checked very little else."

### B. Many agents at once

The reports that include a failure are about a shared database and a shared directory, not about git itself.

@anhnt256, 20 Nov 2026, https://x.com/anhnt256/status/2123845130092450209: four worktrees sharing one `.git`, until "merge conflicts corrupted our test DB." The fix in the thread: an environment variable pointing each branch at its own `test.db` with fresh migrations, and only a coordinator merges to main after lint and tests. @LanceBuildsIt, 24 Nov 2026, https://x.com/LanceBuildsIt/status/2125217919235948828: a generated `.env` per worktree that swaps port and database name. "With one symlinked file, one agent's edit changes every branch at once." @dylan2045ad, 19 Nov 2026, https://x.com/dylan2045ad/status/2123508188003889216: one directory per agent, `package.json` locked to one writer. @keislamoglu, 24 Nov 2026, https://x.com/keislamoglu/status/2125234588351905875: one worktree, port, and local files per task, and a pool of test slots. These are shown as their setups. None published a merge-conflict count.

@jpschroeder’s parent folder, above, is the other layout. I found no post on sparse checkouts.

### C. Feedback loops

The numbers are in review traces and in lint that names the rule.

@tvykruta, 2 Oct 2026, https://x.com/tvykruta/status/2106138620780261842: reviewers that only read the diff found zero of eight logic bugs. Every logic bug came from a reviewer that ran the code. Writing the inputs before reading the implementation moved recall from 4 of 8 to 7 of 8. A git pre-push hook blocked a push with no recorded review. "272 passing tests said nothing" about 11 blocking bugs behind green CI. Shown, on two of his pull requests.

@bcherny, 11 Sep 2026, https://x.com/bcherny/status/2098217573276131577: Anthropic’s bar is lint, tests, agent end-to-end tests, daily fuzzers, and automated review. Shown as his description of their process, with no counts.

Next.js’s AGENTS.md, opened above, tells the agent the fast command (`pnpm --filter=next types` at about 10 seconds versus a 60-second build) and names the lint rule (`@next/internal/no-as-any`) plus the comment shape that satisfies it. That is the "error text tells the agent the fix" pattern, in a file rather than in a custom linter. Shown as the file’s contents, not as an eval.

@TanmayK_03, 24 Nov 2026, https://x.com/TanmayK_03/status/2125107765622558841: a pre-commit secret scan can be skipped with `--no-verify`, so he wants the scan in CI too. Opinion, no count.

### D. Mobile and desktop

I did not find a post, from someone who ships a Swift, React Native, Expo, or Electron app, that says which project file an agent broke and what they changed. Searches for `pbxproj`, `xcodeproj`, Expo, and Electron plus "broke" did not return a usable account.

The closest related posts are @tobi, 25 Aug 2026, https://x.com/tobi/status/2092308116524138711: in a Shopify-scale monorepo, a missing CLAUDE.md or AGENTS.md in one directory means "a subset of devs work with lobotomy," and they paper over it with automation. And @fforres, 15 Oct 2026, https://x.com/fforres/status/2110883187920617472: move the projects folder to a volume with `fseventsd` off, because agent file watching was the pain. Neither is a mobile project file.

### E. Skills, commands, and hooks

What earns a place is something the model cannot see in the repo. What people removed is everything else, once they measured it.

@trq212, 24 Jul 2026, the post above: skills for "opinions, knowledge, or best practices that are particular to you, your team, or product," split into files, not a copy of the docs. `/doctor` is the trimmer. @sqs, 21 Nov 2026, the post above: remove stacks of skills that are not yours. @tianrendong turned off 350+ repo skills and reports the token cut above.

@dani_avila7, 5 Sep 2026, https://x.com/dani_avila7/status/2096052993166544914, and @midudev, 2 Oct 2026, https://x.com/midudev/status/2106038403733098713: `/skill-doctor` lists skills loaded, skills never used, and tokens each costs. Shown as the command’s output, not as someone’s before/after deletion. @stewy0x, 24 Nov 2026, https://x.com/stewy0x/status/2125144044116299841, says Anthropic’s docs cap the skill listing at 1% of the window and drop descriptions after that. I did not open that docs page, so this stays his claim.

@koder0x, 23 Nov 2026, https://x.com/koder0x/status/2124717622449012876: a skill earns a place for "internal conventions, private APIs, and project decisions the code itself fails to explain." Opinion.

### F. Tools that prepare or score a repo

I did not find a practitioner who published a before/after for a repo-map, a packer, or a readiness score and then said they dropped it.

What I did open: Codex’s own page says the 32 KiB cap is the scoring that matters, because the loader stops and the deepest file goes first. @trq212’s `/doctor` and `claude plugin eval` are the Claude Code versions of the same idea. @alex09x_dev, 24 Nov 2026, https://x.com/alex09x_dev/status/2125152711616454798: on Bevy, a warm index took 5.5 seconds versus 114.4 seconds for rust-analyzer on the same post-startup workload. That is a language-server number, one project, his measurement.

Lists that name repomix, gitingest, and Serena (for example https://x.com/elessarqq/status/2125248353692696931, 24 Nov 2026) give no before/after from the author’s repo.

### G. Keeping documents from going stale

Two papers, plus one product command. Agents in the second study did not fix the file themselves.

Xu and Wu, above: when the file and the code disagree, the file wins unless the user-level sentence says otherwise. A follow-up, Mehtiyev, Tadesse, Arkoh, and Assunção, submitted 6 Nov 2026, https://arxiv.org/abs/2611.09802: three real stale instructions (a test command, a path, a procedure), 22 tasks, 732 trajectories. On the stale test command, Claude Sonnet 5.5 used about two extra tool calls, "a third more than with the corrected command." Agents recovered and still finished. "None repaired the underlying instruction file." Correcting the file removed the wasted lookups. Removing the stale instruction prevented most of them. Task success hid the cost.

@trq212’s `/doctor` is described, in his July post, as the check for oversized instruction files. I did not open a page that shows it flagging a dead path. @Sithira269, 26 Oct 2026, https://x.com/Sithira269/status/2114514275293446545: a daily GitHub Action opens a pull request to update docs. No count of what it caught.

### H. Other numbers

ETH Zurich, Gloaguen and others, revised 29 Sep 2026, https://arxiv.org/abs/2602.11988: context files "do not generally improve task success rates, while increasing inference cost by over 20% on average," for both generated files and developer-written ones. "Instructions in the context files are well followed." "Repository overviews, although popular and recommended by model providers, are not helpful." Useful part, in their words: "specifying non-standard coding practices."

AAIF, above: on a documentation site, a 50-line file cut tokens 36% and raised fully correct runs from 7 of 9 to 9 of 9. The same post cites Stripe’s checkout experiment, which is a different mechanism (WebMCP, not a repo file): 60 tests, six models, success unchanged, 42% fewer tokens, 38% fewer tool calls, 39% faster.

@tvykruta, 5 Oct 2026, https://x.com/tvykruta/status/2107177934314225902: in one five-hour session, 57% of the bill was re-reading context, and one restart after a 24-minute pause cost $5 to rewrite 604k tokens into cache. Across 96 sessions that month, the longest cost about 45% more per output token than the shortest. His fix is a fresh session per phase, with nothing carried over except the repo.

## The 8 most useful things that were not in round one

1. The 79-of-80 claim is a public paper with a data repo. The sentence works on those 40 constructed Python projects, on those two agent versions. Put it in the user-level file. It does not mean "the code is always right": on config choices, Claude Code already followed the code.

2. A stale test command costs about two extra tool calls and the agent still does not fix the file (Mehtiyev et al., 6 Nov 2026). Deleting or correcting the line beats adding another "be careful" sentence. Success rate will not show the waste.

3. ETH Zurich: an overview in the instruction file does not raise the share of tasks finished, and it raises cost by over 20%. The lines that earn a place are the practices the code does not already show.

4. Codex stops at 32 KiB and drops the file closest to the work. A long root AGENTS.md can erase the nested override you wrote for the package the agent is in.

5. Two public shapes that match that advice. Supabase’s 77 lines: commands, generated files you must not edit, and a pointer to skills. The AAIF file, about 50 lines, is the one with a measured 36% token cut, and the author says the agent then stopped looking elsewhere, so the file has to stay true.

6. Amp’s Quinn Slack: a stack of skills that are not yours is the usual cause of the agent pulling in several directions. Remove them. Skills that stay should be the company’s.

7. Parallel agents fail on a shared database and a shared env file before they fail on git. One database file and one port per worktree, one writer for shared files, and only one coordinator merges.

8. A reviewer that writes its inputs before it reads the implementation, in a fresh session, found the logic bugs that reading the diff missed (4 of 8 to 7 of 8, same model, two cents). Separately, a session that runs for hours spends most of the bill re-reading itself. Split the work so each session starts from the repo.

## What I looked for and did not find

A public copy of the pricing repo, its full 217-line AGENTS.md, or the session logs behind the 9× and 7× figures. Anyone who repeated that trial or the empty-file trial and posted their own numbers. A post from @bcherny that contains the "delete it every six months" line. A Cursor-team post, since August, on instruction files or repo layout. An Amp post on folders, worktrees, or what to put in a repo file. A firsthand account of an agent moving directories in a real repo, with the imports, history, and open branches that broke, and the command that checked it. Any Swift, Xcode, React Native, Expo, or Electron report of a project file an agent corrupted. Sparse checkouts used so agents see less of a repo. A practitioner who dropped a repo-map, a packer, or a readiness score and gave the number that made them drop it. A team’s own CI job that checks instruction files against the tree and reports how many stale lines it found.
