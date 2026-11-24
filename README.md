<img src="assets/icon.svg" width="64" height="64" alt="">

# AI GitHub efficiency builder

Two standards for working with coding agents, researched on 2026-11-24:

1. **How to lay out a repository** so agents find their way around it, and how to fix one that already exists.
   Full guide: [docs/repo-layout.md](docs/repo-layout.md).
2. **How to spend fewer tokens** in T3 Code threads. Full guide: [docs/token-use.md](docs/token-use.md).

Each guide gives a source for every point and says whether it is documented by the tool maker, measured, reported by one
person, an opinion, or only our default. The researchers' raw results are in [research/](research/).

## 1. Repository layout: the short version

The main finding: agents do not need a tour of the repo. They need a way to check their work. A study of four agents
found that overview files did not help them find code sooner, and that auto-generated rules files cost 20 to 23% more
(arXiv 2602.11988). Anthropic's documentation likewise says to stop describing the layout.

### A new repository

1. **`AGENTS.md` at the root, written by hand, under about 100 lines.** Beside it a `CLAUDE.md` whose only line is
   `@AGENTS.md`. Without that import, Claude Code ignores `AGENTS.md` as soon as any `CLAUDE.md` exists.
2. **In that file:** the exact commands (install, run one test file, lint, typecheck, full check), the rules a check
   enforces, decisions the code does not show, gotchas, and what only the owner starts. **Not in it:** a description of
   the folders, an architecture essay, a style guide, or raw `/init` output.
3. **A rule that must hold goes into a check:** a test, a lint rule, a hook, a deny rule, CI. Prose is only a request.
4. **Keep generated and vendored files out of the way:** `.gitignore` for build output, and `Read(...)` deny rules in
   `.claude/settings.json` for generated or vendored code that is checked in.
5. **One home for each thing the product does.** A folder per domain, and inside it separate files for data, logic and
   screens. File names that say what the file does (`pricing-rules.ts`, not `utils.ts`). No `index.ts` files that only
   re-export, except the entry file of a package that others install. Split a file that does two jobs.
6. **A second `AGENTS.md` inside a folder** only when that folder has its own commands or gotchas.
7. **Documents only for what code and history cannot show:** decisions and their reasons, and plans not built yet.
8. **Keep the rules file true.** Agents follow a stale rules file over the code about 90% of the time, and they do not
   fix it. Correct or delete a wrong line (arXiv 2610.30571 and 2611.09802).
9. **Say which wins.** Put this in your user-level rules file: "When a repository's documentation and its existing code
   disagree, follow the conventions used in the existing code, and report each mismatch." In the study it raised "follow
   the code" from 4% to 91%. Keep rules about safety and secrets binding either way.

### A repository that already exists

Do these in order. The first four move no code, so they cost the least and carry the least risk.

1. Make the check commands work: one test file, lint, typecheck, full check.
2. Add the ignore and deny rules. Never untrack a checked-in file to do it.
3. Write the short `AGENTS.md` by hand. Move an existing rules file's rules into it before trimming anything.
4. Move each must-hold rule from prose into a check, and see the check fail before deleting the prose.
5. Only then restructure code, and only the part that hurts most: count what is wrong, plan small steps that change no
   behavior, one pull request each, tests green at every step, files moved with `git mv` in commits that only move, and a
   check added so the mess cannot return. Migrate in place. Do not rewrite from scratch.
6. After a new model release, list the instructions that only patched an older model.

**For agents working in the owner's repos:** steps 1 to 4 are ordinary setup. Adding is setup, removing is not: cutting
or deleting lines from an existing rules file is proposed to the owner with the diff first. Step 5 is never done
unasked: propose it first with the counts and the steps, and never move files that an open pull request touches.

## 2. Token use: the short version

Measured on the owner's Mac for the week to 2026-11-24, Opus and Fable together: about 18,100 requests, each carrying an
average of 239,000 tokens of context. Every request re-reads its whole thread, so the cost is the number of requests
multiplied by the size of the thread.

What the measurement found:

- Threads grow very large: 139 of 337 sessions passed 400,000 tokens. Nothing compacts a thread automatically until
  Claude Code's default of about 967,000 tokens, because T3 Code's "Auto-compact after" setting is empty.
- Every thread and every delegated child thread starts with about 70,000 tokens before anyone types. About 325 child
  agents were started in the week.
- Fable costs twice what Opus does for output and cache writes, and made a little over half of the cost.
- Returning to a thread after its cache expired (about an hour) re-writes the whole thread: 5 to 9% of the cost.

Settings worth changing, in order (the owner decides):

1. T3 Code, Settings, Providers, Claude: **Auto-compact after** `300000`.
2. T3 Code, Settings, Providers, Advanced: **Compact idle threads** before the cache expires.
3. Stop loading the claude.ai plugin sets and connectors that are not used for coding.
4. Medium effort unless the task needs more.
5. Start long owner-facing threads on Opus, and keep Fable for advice and the sensitive code.

Habits for every thread:

1. One task, one thread. Hand off with a short note and start fresh at about 200,000 tokens.
2. Send independent tool calls together. Each extra turn re-reads the whole thread.
3. Start a child agent only when it earns its 70,000-token start.
4. Stop a review loop after 3 rounds and ask the owner. Loops here ran 4 to 9. Anthropic capped its own at 3 and saw
   pull requests needing five or more rounds fall from 8.9% to 1.5%.
5. Do not wake a large idle thread for a small message.

## How this was researched

- Grok searched X in two rounds: about 190 posts on layout and about 160 on token use, with the link, author and date
  of each post it cites. The second round re-examined the first round's leads and gave each a verdict.
- Gemini 4 Neon read the tool makers' documentation, the published studies and T3 Code's own docs.
- The main claims and every setting name were then checked against the original pages.
- The usage numbers were counted from the Claude Code transcripts on the owner's Mac.

Created in [T3 Code](https://t3.codes).
