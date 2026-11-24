# AI GitHub efficiency builder

The owner's standards for working with coding agents: how to lay out a repository so agents find their way around it,
how to restructure one that already exists, and how to spend fewer tokens in threads. This repo holds documents only.
There is no code, no build and no test suite yet, so there is no full check to run.

## What is here

- `README.md`: the short version of both standards. Other threads are pointed at this file.
- `docs/repo-layout.md` and `docs/token-use.md`: the full guides, with a source for every point.
- `research/`: the researchers' results exactly as they came back.

## Rules to follow

- Every point in a guide carries a source and one of the labels the guide defines (documented, measured, reported,
  default, opinion). A claim found only on X is "reported" or "opinion", never "measured".
- Open the page before citing it. Tools change quickly: when a page and memory disagree, the page wins.
- Files in `research/` are records. Do not edit them. New research goes in a new file with its date in the heading.
- The usage numbers in `docs/token-use.md` come from the owner's own transcripts on one Mac. Say the date range when
  quoting them, and re-measure before changing a recommendation that rests on them.
- Keep `README.md` short enough to read in a few minutes. Detail goes in `docs/`.
- Write in plain English, for a reader who has not seen the research.
- No detail about the owner or his other projects goes into a web search or a research brief.

## Pull requests

- Work on a branch and open a pull request. The owner merges.
- A change to a recommendation says in the pull request which source or measurement changed.

## Only the owner starts

- Changes to `~/.claude/CLAUDE.md`, `~/.claude/settings.json` and T3 Code's settings. A thread proposes the change and
  shows the diff, and makes it only when the owner has asked for it.
