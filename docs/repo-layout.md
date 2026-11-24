# How to lay out a repository for coding agents

Researched on 2026-11-24. Grok searched X (about 110 posts), Gemini 4 Neon read the tool makers' documentation and the
published measurements, and the main claims were then checked against the original pages. The raw results are in
[`research/`](../research/).

Each point carries one of four labels:

- **documented**: the tool's own documentation says it.
- **measured**: a source published numbers from a test.
- **reported**: one team or person described what happened to them, with their own counts.
- **default**: nobody has measured it. It is our choice, made so every repo is the same.

## What the evidence says, in one paragraph

Agents do not need a tour of the repository. They read the code themselves, and a study of four agents found that an
overview file did not help them find the right files any sooner, while auto-generated instruction files made them slightly
less successful and over 20% more expensive (measured, ETH Zurich and LogicStar.ai, arXiv 2602.11988). What does help is a
short, hand-written file that says how to check the work, plus checks that fail when a rule is broken. So the standard
below is mostly about commands and checks, and only a little about folders.

## Part 1: a new repository

### 1. One short rules file, written by hand

- `AGENTS.md` at the root. It is the shared format that Claude Code, Codex, Cursor, Gemini CLI, Amp, Devin, Copilot and
  Aider all read (documented, [agents.md](https://agents.md/)).
- Beside it, a `CLAUDE.md` whose only line is `@AGENTS.md`. Claude Code reads `AGENTS.md` by itself only when no
  `CLAUDE.md` exists in the folder or above it, so a `CLAUDE.md` without the import silently hides the rules (documented,
  [Claude Code memory](https://code.claude.com/docs/en/memory)).
- Keep it under about 100 lines. Anthropic's limit is 200 lines per file, and says longer files "consume more context and
  reduce adherence" (documented). OpenAI keeps its own root file near 100 lines (reported,
  [Harness engineering](https://openai.com/index/harness-engineering/)).
- Write it by hand from the code. Do not commit the raw output of `/init` or any other generator (measured, arXiv
  2602.11988: generated files lowered success by 0.5 to 2 points and raised cost 20 to 23%).

What goes in it:

| Put in | Leave out |
|---|---|
| The exact commands: install, run one test file, lint, typecheck, full check | A description of the folders and files |
| Rules that a test, linter or hook enforces, and which one | An architecture essay |
| Decisions the code does not show, and why | A style guide (use a linter) |
| Gotchas that have cost someone time | How to use standard tools such as git or npm |
| What only the owner may start (paid calls, deploys, real devices) | Anything a reader could find by reading the code |

Anthropic's test for each line: "Could it find this by reading the code? Would it actually go and look? If both are yes,
leave it out" (documented, [best practices](https://code.claude.com/docs/en/best-practices)). The same page lists
"describe your architecture and file layout in CLAUDE.md" under things to stop doing.

Write rules in plain words. Capitals and "IMPORTANT" make current models apply a rule too widely (documented, same page).

### 2. Commands that check the work

Anthropic calls the check command "the single most useful line" in the file (documented). Every repo needs these, each as
one command that works from a fresh checkout:

- run the tests for one file or one unit
- lint
- typecheck (where the language has types)
- the full check, the same one CI runs

On this Mac, never name a command that starts one test worker per core. Cap the workers in the runner's config, or pass
the cap in the command.

### 3. Rules that must hold go into checks, not prose

A line in a markdown file is a request. A failing check is a rule. "If you keep adding emphasis to a CLAUDE.md line and
Claude still misses it, that's a sign the line belongs in a hook" (documented, best practices). So:

- formatting and naming: a linter
- "these two parts must not import each other": a lint rule or a test
- "never edit this folder": a `permissions.deny` rule or a hook
- everything above, again, in CI

When a custom lint rule fails, make its message say how to fix it. The agent reads that message (reported, OpenAI).

### 4. Keep generated and vendored files out of the way

- `.gitignore` covers build output and caches. Claude's searches respect it (documented,
  [large codebases](https://code.claude.com/docs/en/large-codebases)).
- Generated or vendored code that is checked in needs deny rules in `.claude/settings.json`, for example
  `"Read(./**/vendor/**/*)"` and `"Read(./**/*.generated.*)"` (documented, same page). These rules do not stop a shell
  command from opening the file.
- Other tools have their own files: `.cursorignore`, `.geminiignore`, `.aiderignore` (documented by each tool). Add one
  only for a tool that is used on the repo.

### 5. Folders and names

Nobody has measured feature folders against layer folders for agents, or tests beside the code against a separate tree,
or any maximum file length. Both researchers looked and found nothing. These are defaults:

- **One home for each thing the product does.** A folder per domain (`pricing/`, `bookings/`), and inside it keep data
  access, logic and screens in separate files. One reported case: pricing logic spread over 31 files and a 2,916-line
  main file made one feature take 49 pull requests. After it was gathered into one package of 14 small modules, the
  author says his session logs showed 9 times fewer tokens and a 7 times lower cost per pull request (reported on X by
  @tvykruta, 2026-09-30, his own counts, logs not published).
- **Names that say what the file does.** `pricing-rules.ts`, not `utils.ts` or `helpers.ts`. A name that appears once in
  the repo can be found with one search (default).
- **No barrel files inside the app.** A barrel is an `index.ts` that only re-exports its neighbours. Import from the real
  file. Removing them made Jira's builds 75% faster (measured, Atlassian, 2025), and slow typechecks and tests are what
  an agent waits on. Keep an entry file only for a package that others install.
- **Split a file when it does two jobs, or passes about 500 lines.** The line count is a default, not a finding.
- **Tests go where the language expects them,** and `AGENTS.md` says where in one line.
- **One repo per product.** A monorepo is fine when each package has its own rules file (see 6). Whether agents do better
  in one repo or many is disputed, with no shared measurement.

### 6. A second rules file only where a folder has its own commands

A nested `AGENTS.md` inside a package or subsystem loads only when the agent works on files there (documented for Claude
Code, Cursor, Amp, Gemini CLI and Devin). The closest file to the edited file wins (documented, agents.md). Add one when
a folder has its own test command or its own gotchas, and not before. Codex is the exception: it reads only the files
from the root down to the folder it was started in (documented, OpenAI).

### 7. Documents: only what the code and history cannot show

- Decisions and their reasons, in `docs/decisions/`, one short file each. They stop an agent from undoing a choice that
  looks wrong but is deliberate (opinion, widely held).
- Plans for work not built yet.
- Nothing that repeats the code. A document that has drifted from the code misleads the agent, and practitioners
  disagree on whether an architecture file is worth keeping at all. If one exists, keep it short and name modules and
  types in it, never line numbers.

### 8. A language server, where one exists

A code-intelligence plugin lets Claude jump to a definition instead of searching the tree (documented, large codebases).
Use the one for the repo's language.

### The rules file, as a starting point

```markdown
# <project name>

<Two or three sentences: what it is, who uses it, what it is built with.>

## Commands
- Install: `<command>`
- Run one test file: `<command> <path to the test file>`
- Lint: `<command>`
- Typecheck: `<command>`
- Full check (what CI runs): `<command>`

## Rules a check enforces
- <rule>: enforced by <the test, lint rule, hook or CI job>

## Rules to follow
- <a decision the code does not show, and why>
- <a gotcha that has cost time>

## Where things go
- <only what is not obvious from the tree, such as where tests live>

## Pull requests
- <branch, review and merge rules>

## Only the owner starts
- <paid calls, real devices, deploys>
```

## Part 2: a repository that already exists

Work in this order. The early steps move no code and give most of the gain. Stop after any step: each one stands alone.

| Step | What to do | How to tell it worked |
|---|---|---|
| 1 | Make the check commands work: one test file, lint, typecheck, full check. | Each runs from a fresh checkout and gives the same result as CI. |
| 2 | Add ignore and deny rules for build output, generated code, vendored code and secrets. | `git check-ignore -v <path>` names the rule. Claude is refused when it tries to read a denied file. |
| 3 | Write the short `AGENTS.md` by hand, and `CLAUDE.md` with `@AGENTS.md`. Cut any existing file down: remove the folder tour, the essay and anything stale. | Under about 100 lines. `/memory` in Claude Code shows it loaded. |
| 4 | Move each rule that must hold from prose into a check, then delete the prose. | Break the rule on a scratch branch. The check fails and says how to fix it. |
| 5 | Fix the worst part of the code first, in place. | See below. |
| 6 | After a new model release, delete instructions that only patched an older model. `/doctor` in Claude Code helps. | No dead paths or commands, and no two rules that contradict each other. |

Steps 1 to 4 come first because Anthropic names a runnable check as the most useful single item (documented) and because
a shorter file was the largest change with numbers behind it: one engineer ran two weeks with an empty rules file and all
350 repo skills off, and reported 60,000 fewer tokens per turn and about 25% lower spend, with no rise in errors
(reported on X by @tianrendong, 2026-11-23, his own logs, not published).

### Step 5 in detail: restructuring code

This is the only step that moves code, and the only one with real risk.

1. **Pick one part that hurts.** Count what is wrong: how many files hold this logic, how many copies of the same
   formula, how long the biggest file is. Write the counts down.
2. **Plan small steps that change no behavior.** One pull request each. No feature work in the plan. "Never both in one
   unverifiable diff" (reported, @tvykruta).
3. **Tests stay green at every step.** If a step changes behavior, that step is wrong.
4. **Move files with `git mv`, in a commit that only moves.** Edits go in a separate commit, so history and blame
   survive. Check with `git log --follow <file>` (documented, git).
5. **Update imports with a codemod or the editor's rename,** not by hand and not by search and replace across the repo
   (opinion, martinfowler.com).
6. **Move files only when no open pull request touches them.** A move under an open branch causes conflicts. For a large
   move, raise `merge.renameLimit` so git still detects the renames (documented, git).
7. **Add a check that fails if the mess comes back,** such as a lint rule that forbids pricing code outside `pricing/`.
8. **Count again** with the same counts as step 1. Then build the feature. Then pick the next part.

### Rewrite or migrate in place?

Migrate in place. A full rewrite of a product people use tends to fail, because the old behavior in the edge cases is
hard to write down and new features cannot wait (opinion, Martin Fowler, long-standing). In the one reported case with
numbers, a 3-hour rewrite across 89 files was thrown away and 11 small behavior-preserving steps worked. Starting from
scratch is right only for a new product with nothing to keep.

### Rules for agents in the owner's repos

- Steps 1 to 4 are ordinary setup. Do them when they are missing.
- Step 5 is never done unasked. Propose it to the owner first, with the counts from step 1 and the list of steps.
- Never move files that an open pull request touches.

## Where good sources disagree

| Question | One side | The other side |
|---|---|---|
| How much to write in the rules file | Almost nothing: Anthropic's docs, the ETH study, an empty-file trial at Ramp | A long, specific file: one practitioner credits it for a 7 times cost drop |
| An architecture or map file | Keep a short one with boundaries and invariants (matklad, OpenAI) | Delete it, it drifts, the code is the truth (several practitioners; Anthropic says stop describing layout) |
| Always-loaded rules or skills loaded on demand | Skills are cheaper (Devin, Claude Code docs) | Skills often never fire: a docs index in `AGENTS.md` passed 100% of tests against 53 to 79% for skills (measured, Vercel, 2026) |
| One repo or many | Monorepo: 4 times faster on one cross-project demo (measured, Nx, one demo) | Small repos are faster and cheaper for agents (opinion, engineers at Cloudflare and elsewhere) |

Where they disagree, this standard takes the cheaper side: less text loaded on every turn, and checks in place of prose.

## What nobody has measured

Both researchers looked for these and found no controlled test: feature folders against layer folders, tests beside the
code against a separate tree, a maximum source-file length, the cost of duplicate file names such as `index.ts`, and
whether a README in every folder or decision records raise an agent's success rate.
