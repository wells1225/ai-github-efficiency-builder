# Repo layout for coding agents: what people post on X

Raw research result, saved as returned on 2026-11-24. Researcher: grok-4.7. Not edited. Claims found only on X are opinion until another source backs them.

---

X search worked. I read about 110 posts and thread replies from the last 12 months, and opened four pages those posts pointed at: the AGENTS.md spec, Anthropic's large-codebase guide, Anthropic's best-practices page, and Real Python's AGENTS.md tutorial.

## A. New repositories

**Keep the always-loaded instruction file short, and put in it only what the code cannot show.** Thariq (@trq212), who works on Claude Code, wrote that Anthropic removed over 80% of Claude Code's system prompt for the newest models "with no measurable loss on our coding evaluations." The article's rule for a repo file: "Keep your CLAUDE.md lightweight and briefly describe what your repo is for, but spend most of the tokens on gotchas." "Avoid stating 'the obvious' things Claude should know by looking at your file system." The current best-practices page says the same, and says the memory guide recommends staying under 200 lines. `/init` drafts the file; "the work is mostly removing lines." Boris Cherny (@bcherny) replied "I don't use a custom CLAUDE.md" when asked for his system instructions. Shown for the prompt cut and the written rule. The eval table is not in the post.

- https://x.com/trq212/status/2080710971228918066 — @trq212 — 24 Jul 2026
- https://code.claude.com/docs/en/best-practices — Anthropic docs, opened 24 Nov 2026
- https://x.com/bcherny/status/2107591357170950471 — @bcherny — 6 Oct 2026

**An empty instruction file can be cheaper and no worse.** Tianren (@tianrendong), an engineer at Ramp, ran a coding agent for two weeks in a large monorepo with 350+ repo skills off and an empty AGENTS.md. He reports about 8,000 turns, about 60,000 tokens saved per turn (skill descriptions plus a 40kb AGENTS.md), about 25% lower spend from cache logs, flat tool-call error rates, and no extra corrections. One worktree accidentally kept the old file for about 600 turns and "didn't feel any smarter." Shown as his own logs. He did not publish the logs.

- https://x.com/tianrendong/status/2124948368569069678 — @tianrendong — 23 Nov 2026
- https://x.com/tianrendong/status/2124948479776821565 — @tianrendong — 23 Nov 2026

**Say how to run, test one unit, lint, and typecheck, and call the task done only when those pass.** Anthropic's sample CLAUDE.md is commands, two conventions, and one gotcha, including "Run one test file: `pnpm vitest run path/to/file.test.ts`." The AGENTS.md spec's sample is the same idea: `pnpm turbo run test --filter <project_name>`, and `pnpm vitest run -t "<test name>"` for one test. Real Python's tutorial shows a before and after on a FastAPI app. With no file, the agent used `len(cars) + 1` for ids, returned 200 for a missing car, and swapped `pathlib` for `os.path`. A short file with domain rules plus `ruff`, `mypy`, and `pytest` as the definition of done changed that on the rerun. Their line: "The agent follows your rules in a lean file, but a sprawling one buries them." Shown as a tutorial with the code, not a production measurement.

- https://code.claude.com/docs/en/best-practices — opened 24 Nov 2026
- https://agents.md/ — opened 24 Nov 2026
- https://x.com/realpython/status/2125274526711718006 — @realpython — 24 Nov 2026
- https://realpython.com/agents-md/ — opened 24 Nov 2026

**A markdown "never" does not hold. A failing check does.** Anthropic's page splits instructions into ones the model may ignore and ones the harness enforces. "If you keep adding emphasis to a CLAUDE.md line and Claude still misses it, that's a sign the line belongs in a hook." Their "stop doing" table says ALL CAPS and "IMPORTANT" make current models over-apply a rule. Alex Vainshtein (@Alex_Vainshtein) wrote that after a month of adding "CRITICAL: ALWAYS DO X" he had an 800-line file "and the agent still ignores half of it." "Markdown files make terrible linters." He moves a repeated mistake into a pre-commit or build step. Boris Cherny's public note says production code written by Claude should have "lots of lint rules, lots of tests," and that CLAUDE.md should "succinctly teach Claude how to work in your codebase." Shown for the docs. Vainshtein's 800 lines and "half" are his account, with no log attached.

- https://code.claude.com/docs/en/best-practices — opened 24 Nov 2026
- https://x.com/Alex_Vainshtein/status/2116418200514519205 — @Alex_Vainshtein — 31 Oct 2026
- https://x.com/bcherny/status/2098217573276131577 — @bcherny — 11 Sep 2026

**Load detail only when the agent is in that part of the tree.** Thariq names this "progressive disclosure": a tree of files loaded at the right time, with verification moved out of the always-on prompt and into a skill. Anthropic's large-codebase guide says a root CLAUDE.md plus one file per package or subsystem. Subdirectory files load on demand. Starting `claude` inside `packages/api/` loads the root file and that package's file, and not `packages/web/`. The AGENTS.md spec says the closest file wins, and states that the main OpenAI repo had 88 AGENTS.md files when they wrote the page. I did not count that repo. Shown as product behavior and the spec.

- https://x.com/trq212/status/2080710971228918066 — @trq212 — 24 Jul 2026
- https://code.claude.com/docs/en/large-codebases — opened 24 Nov 2026
- https://agents.md/ — opened 24 Nov 2026

**One shared AGENTS.md. If CLAUDE.md exists, Claude will skip AGENTS.md unless you import it.** Thariq announced that from Claude Code 2.1.277, Claude reads AGENTS.md only when the folder has no CLAUDE.md. Brandon M. Crenshaw (@brandononchain) wrote the practical version: rules live in AGENTS.md, and CLAUDE.md's first line is `@AGENTS.md`. A CLAUDE.local.md also blocks AGENTS.md. Gustavo Gonzalez (@aguardientico) posted a tiny repeat: the same "add a price" prompt used `price_cents` in 2 of 2 runs with only AGENTS.md, and a decimal column in 2 of 2 runs once a CLAUDE.md was present with no import. Shown as product behavior plus a 2-and-2 demo.

- https://x.com/trq212/status/2101009392611278961 — @trq212 — 18 Sep 2026
- https://x.com/brandononchain/status/2122880126178283613 — @brandononchain — 18 Nov 2026
- https://x.com/aguardientico/status/2125280790393045364 — @aguardientico — 24 Nov 2026

**Keep generated code, vendored code, build output, and secrets out of the agent's reads.** Anthropic's guide says search already respects `.gitignore`, so `node_modules/`, `dist/`, and `build/` stay out of search. Checked-in vendor and generated trees need `Read` deny rules, for example `Read(./**/vendor/**/*)` and `Read(./**/*.generated.*)`. Their sample root file says never edit `packages/*/generated/`, and run codegen instead. Real Python's ignore block says treat `.gitignore` as off limits, and also skip secrets, `.env`, large data, and vendored or generated code. Shown as docs and a tutorial. I did not find a measured token saving for this step alone.

- https://code.claude.com/docs/en/large-codebases — opened 24 Nov 2026
- https://realpython.com/agents-md/ — opened 24 Nov 2026

**Give the agent a symbol lookup instead of a tour of the tree.** The same Anthropic guide says a code-intelligence plugin (TypeScript, Python, Go, Rust) lets Claude jump to a definition instead of scanning files. Their best-practices page now lists "Describe your architecture and file layout in CLAUDE.md" under "What you can stop doing," because "Claude reads the repo itself. Only write down what isn't in there." Shown as current product guidance. This directly cuts against older "put a map in the root file" posts.

- https://code.claude.com/docs/en/large-codebases — opened 24 Nov 2026
- https://code.claude.com/docs/en/best-practices — opened 24 Nov 2026

**Folder shape people actually showed is "one owner per domain," with layers kept apart, not a pure feature-or-layer choice.** Tomas Vykruta (@tvykruta) published the AGENTS.md from his app. Domain logic lives in `services/<domain>/`. Persistence, presentation, and routing stay separate. "If you describe it with 'and', split it." "Names say intent." Pricing had been in 31 files and a 2,916-line app file. After the rules it was one package of 14 small modules. CodeRabbit's CEO Harjot Gill (@harjotsgill) said most of their simulation tests "now live right next to the code," and a failing schedule is kept as a test for the next agent. Real Python's small example does the opposite: tests go in `tests/test_main.py`, and the agent may not create new files unless asked. Shown as three concrete layouts. Nobody published a test of feature folders against layer folders.

- https://x.com/tvykruta/status/2105306637904863429 — @tvykruta — 30 Sep 2026
- https://x.com/tvykruta/status/2105304420644794672 — @tvykruta — 30 Sep 2026
- https://x.com/harjotsgill/status/2125250690243088533 — @harjotsgill — 24 Nov 2026
- https://realpython.com/agents-md/ — opened 24 Nov 2026

**A docs map is a contested extra, not the default.** Vaidehi (@Ai_Vaidehi) says the repo needs a why, a map, rules, and workflows, with a short CLAUDE.md, skills, hooks, a `docs/` folder of architecture and decision records, and a local CLAUDE.md in `src/auth/` and similar. Duy (@zueydotme) says delete ARCHITECTURE.md, PRD.md, CODE_STANDARDS.md, and the pile of specs and ADRs, because they drift, and "source code is the truth. git history is the evidence. AGENTS.md is the guide." Keep only what code and history cannot recover, such as requirements not built yet. Anthropic's current page sides with the second view for layout descriptions. Both posts are opinion. The docs page is shown product guidance.

- https://x.com/Ai_Vaidehi/status/2035439835448557906 — @Ai_Vaidehi — 21 Mar 2026
- https://x.com/zueydotme/status/2124163523492385263 — @zueydotme — 21 Nov 2026

## B. Existing repositories

**The largest measured gain I found was not a new folder tree. It was deleting instructions, or forcing a refactor before the next feature.** Tianren's empty-file result is above. Vykruta's thread is the restructuring account. Before the rules, one pricing feature took 5 days, 49 pull requests, and about $640. He reports pricing logic in 32 files, one formula copied 40 times, and a 2,916-line main file. A 3-hour rewrite across 89 files was thrown away. He calls the textbook answer "rewrite from scratch," and did not do that for the app. He put the rules in AGENTS.md, required a behavior-preserving refactor with green tests before any behavior change, and consolidated pricing in 11 such steps. He reports price code outside the module from 76 to 0, files holding price logic from 31 to 1 package, formula copies from 40 to 10, and 4,803 lines removed. He says a before-and-after of the code and the session logs showed 9× fewer tokens, 7× cheaper per pull request, and 5× faster completion, and that later pull requests merged on the first try. Shown as his published counts. I did not see the logs.

- https://x.com/tvykruta/status/2105302045573959697 — @tvykruta — 30 Sep 2026
- https://x.com/tvykruta/status/2105302369160286335 — @tvykruta — 30 Sep 2026
- https://x.com/tvykruta/status/2105303400246075793 — @tvykruta — 30 Sep 2026
- https://x.com/tvykruta/status/2105305072817037419 — @tvykruta — 30 Sep 2026
- https://x.com/tianrendong/status/2124948368569069678 — @tianrendong — 23 Nov 2026

**His order of work is the only step-by-step "rebuild in place" I found with numbers.** Review one hurting system and count files, copies, and disagreements. Plan small behavior-preserving steps, one pull request each, no feature in that plan. Tests stay green. If a step changes behavior, the refactor is wrong. Add a check that fails when the mess returns. Review again with the same numbers. Only then build the feature. Then pick the next system. "Never both in one unverifiable diff." "A prompted 'never' alone does not protect" the source of truth. Shown as the process he says he ran.

- https://x.com/tvykruta/status/2105305072817037419 — @tvykruta — 30 Sep 2026
- https://x.com/tvykruta/status/2105306637904863429 — @tvykruta — 30 Sep 2026

**Anthropic's advice for a large repo you already have is to scope the session, not to move the files.** Start Claude in the package directory when the task is local. Exclude other teams' CLAUDE.md files with `claudeMdExcludes`. Deny reads of generated and vendored paths. Use sparse worktrees so an agent checkout contains only the directories the task needs. Add a skill next to the code it describes. Revisit the instruction files after a model release and delete rules that only patched an older model. For a mechanical change across many files, `/batch` splits it into 5 to 30 units, each in its own worktree, after you approve the plan. Shown as product docs. No before-and-after token numbers on a customer repo.

- https://code.claude.com/docs/en/large-codebases — opened 24 Nov 2026
- https://code.claude.com/docs/en/best-practices — opened 24 Nov 2026

**Boris Cherny treats a rewrite as a later option.** If succinct CLAUDE.md, skills, a frontier model, and higher effort still fail, "have Claude fix accumulated debt and rewrite your codebase to make it easier to work with. Or, wait for the next model." Opinion from the person who leads Claude Code. He does not describe a migration he ran.

- https://x.com/bcherny/status/2098217573276131577 — @bcherny — 11 Sep 2026

**People who talked about moving files want the move planned and checked before it runs.** Endri Lickollari (@lickollariendri) built Movesafe so an agent does not "move the file, update the imports, and hope." `plan_move` produces a diff and a `planHash`. `apply_move` recomputes the plan and refuses to run if the hash changed. Opinion about his tool. I did not see it used on a public repo in the thread.

- https://x.com/lickollariendri/status/2123329256960717052 — @lickollariendri — 19 Nov 2026

**Two weaker "do this first" posts, both prompts rather than results.** 0xMarioNawfal (@RoundtableSpace) says that on an old repo the agent should map what the task touches and name what it is guessing, and that no edits happen until a person approves the map. Arpit Bhayani (@arpit_bhayani) says sort the existing tree into two buckets before handing it over: throwaway tooling the agent may own, and auth, payments, and data integrity, where the diff is a draft a person reads. Opinion.

- https://x.com/RoundtableSpace/status/2109973675739152452 — @RoundtableSpace — 13 Oct 2026
- https://x.com/arpit_bhayani/status/2092103965655867831 — @arpit_bhayani — 25 Aug 2026

## C. Disagreements and failures

**Long instructions versus almost none.** The shorten camp is Thariq, Anthropic's current docs, Tianren's empty file, Real Python, and Vainshtein. The other camp still wants a dense file. Vykruta's published AGENTS.md is long and specific, and he credits it for the cost drop. g (@garvan_k) replied on that thread: "Despite people recommending to remove this level of detail in the industry my claude.md looks similar to my liking and claude writes godlike code." Mo (@atmoio) says delete the file and leave one paragraph telling the agent to stop and recommend a refactor instead of bolting a feature on. Nader Dabit (@dabit3), at Cognition, says skills are "increasingly unnecessary, counterproductive," and "the new version of code smell." 10x (@10x_er) answered that he still spends a Monday writing a file that explains where the tests live. Opinion on every side except Tianren's logs and Anthropic's prompt cut.

- https://x.com/garvan_k/status/2105441593209893309 — @garvan_k — 30 Sep 2026
- https://x.com/atmoio/status/2119525790723494116 — @atmoio — 8 Nov 2026
- https://x.com/dabit3/status/2124539594905448904 — @dabit3 — 22 Nov 2026
- https://x.com/10x_er/status/2125041680248373661 — @10x_er — 24 Nov 2026

**Monorepo versus many repos.** Matt Carey (@mattzcarey), agents and MCP at Cloudflare, wrote: "unless you are willing to reimplement git / github (like anthropic), I think monorepos are an antipattern in an age of agents." He prefers small repos plus a CLI or MCP to search the org without a checkout, and said the small repo is "so much faster and less compute." Justin Schroeder (@jpschroeder) said his team replaced most monorepos with a parent repo that holds only a `.gitignore` and an AGENTS.md, with the real repos cloned underneath and ignored by git, and agents run from the parent. He also wrote "almost anything works these days so we hold our beliefs loosely." Jacob Hands (@jachands) replied "i completely disagree, at least with a small team like we have. Monorepo makes things so much easier." Anthropic's large-codebase page and the AGENTS.md spec both assume you keep the monorepo and add nested instruction files. Opinion throughout. No shared measurement.

- https://x.com/mattzcarey/status/2120532474761556475 — @mattzcarey — 11 Nov 2026
- https://x.com/jpschroeder/status/2120551435788222485 — @jpschroeder — 11 Nov 2026
- https://x.com/jachands/status/2120618771547299876 — @jachands — 11 Nov 2026

**Where tests live.** Gill keeps the regression beside the code. Real Python uses a separate `tests/` file. Anthropic's sample skill puts API tests in `src/__tests__/`, mirroring `src/`. Adan Zweig (@Adojz) posted the failure mode in one line: "my agent still can't find where the tests live." No one showed which layout used fewer tokens.

- https://x.com/harjotsgill/status/2125250690243088533 — @harjotsgill — 24 Nov 2026
- https://x.com/Adojz/status/2125071519462117827 — @Adojz — 24 Nov 2026
- https://code.claude.com/docs/en/large-codebases — opened 24 Nov 2026

**What made agents worse, from people who showed a mechanism.**

Conflicting always-on rules. Thariq says internal transcripts contained both "leave documentation as appropriate" and "DO NOT add comments" in one request, and the model spent thought resolving the clash. Boris's check for a rabbit hole is `claude --safe-mode`. "If the model stops rabbit holing, then it's something in your CLAUDE.md or skills."

- https://x.com/trq212/status/2080710971228918066 — @trq212 — 24 Jul 2026
- https://x.com/bcherny/status/2085954130435248621 — @bcherny — 8 Aug 2026

A second instruction file that silently disables the first. The Gonzalez demo above. Shown, n=2 per side.

Preloading every skill and a large AGENTS.md on every turn. Tianren's 60,000 tokens per turn. Shown as his logs.

Patching a scattered domain instead of consolidating it first. Vykruta's 49 pull requests and the discarded 89-file rewrite. Shown as his counts.

Stale docs that outrank the code. Duy's argument is above. Vox (@Voxyz_ai) claims researchers ran a trap 80 times, Claude followed the stale doc 79 times, Codex rewrote working code to match the docs in 1 of 4 runs, and one AGENTS.md sentence raised "follow the code" from 4% to 91%. The thread contains no paper, repo, or method. I am not counting those numbers as evidence.

- https://x.com/Voxyz_ai/status/2114446907355648062 — @Voxyz_ai — 25 Oct 2026

A pile of identical filenames. Chai (@chai19536) wrote that nothing humbles an agent faster than five `index.ts` or `config.json` files, "learned that one the hard way." Opinion, one sentence, no repo.

- https://x.com/chai19536/status/2123732378518729152 — @chai19536 — 20 Nov 2026

## Ten practices the most credible posters agree on

Credible here means the Claude Code people, the AGENTS.md spec (OpenAI, Amp, Cursor, Factory, Jules), and practitioners who posted a repo file or a count. Order is how often those sources said the same thing.

1. Write the exact install, test, single-test, lint, and typecheck commands, and treat "done" as those commands passing.
2. Keep the always-loaded file short. Commands, gotchas, and decisions the code does not show. Leave out the tour of the tree.
3. Enforce a rule that cannot have exceptions with a hook, a deny rule, a linter, or CI. A line in markdown is a request.
4. Use progressive disclosure. A short root file, and nested AGENTS.md or CLAUDE.md and skills that load only for the directory in play.
5. Put shared rules in AGENTS.md. If you also have CLAUDE.md, import AGENTS.md. Claude skips AGENTS.md when CLAUDE.md is present and not imported.
6. Keep secrets, build output, generated code, and vendored code out of reads. `.gitignore` covers search. Committed vendor and generated trees need an explicit deny.
7. On a repo you already have, fix the worst subsystem in small behavior-preserving steps with tests green. Do not mix that refactor with the feature. Do not start with a from-scratch rewrite.
8. Let the agent jump to a definition through the language server, or at least give it a one-file test command, so it does not reread the repo to get oriented.
9. After a model upgrade, delete instructions that only existed because an older model needed them. `/doctor` is Anthropic's tool for that.
10. Personal habits stay in a gitignored local file. The team file is what is committed and reviewed.

## What I looked for and did not find

- A measured comparison of feature folders against layer folders, or of flat against nested, for agent cost or accuracy.
- An agreed maximum source-file length. Vykruta showed a 2,916-line file as a failure. He did not publish a line cap. One app-building prompt said "files under 300 lines." That was a prompt, not a result.
- A real discussion of barrel files. One joke about five `index.ts` files. No config, no before and after.
- `GEMINI.md` or `.cursor/rules` as a layout practice. The spec lists Cursor and Gemini CLI as AGENTS.md readers. Cursor's own posts in this window are about a context-usage report, not about how to lay out a repo.
- `llms.txt` used as a map of the repo the agent edits. I found it as an index of public docs sites, including Anthropic's docs and a Bloomberg library's docs. Jeff Posnick (@jeffposnick, 24 Nov 2026, https://x.com/jeffposnick/status/2125268060273786998) tells an agent to read a library's `llms.txt` so it can use that library.
- A careful `git mv` history-preserving playbook with a result. Movesafe is a plan-hash tool. No one posted a migration that kept blame and showed the agent did better afterward.
- A codemod campaign on an existing repo, with what broke.
- Evidence that a generated repo map, a per-folder README, or a committed architecture file reduced tokens. The products advertising those maps did not show a public before and after I could check. Anthropic's current page says to stop describing the layout in CLAUDE.md.
- The Vox 80-run stale-doc experiment, beyond the claim. No method in the thread.
