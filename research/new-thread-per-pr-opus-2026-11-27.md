# New thread per PR vs. reusing an old thread: Opus verification (2026-11-27)

Researched on 2026-11-27 by Claude Opus 5.5 (`medium` effort) from local transcripts on one Mac (2026-11-20 to
2026-11-27) and primary provider pricing/documentation pages. Sanitized for a public repository (private project names
and active thread lists removed).

**Verdict: CONDITIONAL YES.** For each PR, a new thread is clearly cheaper here, usually by 10 to 20 times on the
context it would otherwise drag along. Across the whole week the saving is small (about 2.6% of list-price input cost)
because few threads carried more than one PR. Nothing local measures output quality, so the quality case is indirect.

## Pricing correction (documented, checked 2026-11-27)

On Anthropic's [pricing page](https://platform.claude.com/docs/en/about-claude/pricing):
- **Claude Opus 5.5:** $4/MTok input, $20/MTok output, $5/MTok 5-minute cache write (1.25x), $8/MTok 1-hour cache write
  (2x), **$0.20/MTok cache hits & refreshes (0.05x base input)**.
- **Claude Fable 5.2:** $8/MTok input, $40/MTok output, $10/MTok 5-minute cache write (1.25x), $16/MTok 1-hour cache
  write (2x), **$0.20/MTok cache hits & refreshes (0.025x base input)**.

Earlier sketches that assumed 0.1x cache reads understated the break-even turn count by roughly 2x on Opus and 4x on
Fable.

## 1. Local measurements (measured here, one Mac, 2026-11-20 to 2026-11-27)

### a. Claude (324 sessions, 16,624 requests)

- **Measurement notes:** Requests that include an advisor call sum multiple internal steps on their usage line; using
  the last step's context gives a true maximum of 965k (instead of inflated multi-step sums). PRs are matched on the PR
  creation tool only, deduplicated by PR URL.

| Measure | Result |
|---|---|
| Requests up to opening a PR | median **39** (p25 26, p75 111) |
| Context carried into the first request after a thread's first PR | median **258k** (p75 371k) |
| Requests after the first PR in the same thread | median **6.5** (mostly CI fixes and review rounds on that PR) |
| Sessions with 2 or more PRs | 3 of 324 |
| Later-PR segments (work toward a 2nd+ PR in an existing thread) | 12 (10 in one long-running project thread) |
| Fresh start, first request | 72k context / 41k cache write before 2026-11-24; 55k context / 23k write after (includes lean children) |

- **Advisor calls grow fastest with thread size:** each `advisor()` call sends the whole thread to Fable as uncached
  input. Across the week there were 511 advisor calls at a median of 170k tokens each (101M tokens total; subscription
  weighting is unpublished). Inside the 12 later-PR segments alone, advisor calls added about $18 at list prices.
- **Replay of the 12 later-PR segments:** context carried beyond a fresh start cost **$53 plus $18 in advisor calls**
  ($71 total) at list prices, against **$3 to $8** for fresh starts (including re-reading 30k of files) — about $65
  saved (~2.6% of ~$2,487 weekly list-price input). This is conservative because it excludes 4 segments that sat idle
  over 1 hour (where reuse also rewrites the whole cache at 2x).
- **Idle expiry across all sessions:** returning after >1 hour idle happened 57 times, writing a median of 189k tokens
  at 2x (12.6M tokens total).

### Break-even turn counts

For Opus 5.5, a fresh start costs ~47k to 107k base-input units (23k written at 2x, plus up to 30k of re-read files).
Each request in an old thread pays an extra `(C_old - 55k) * 0.05`. For Gemini 4 Neon ($3/M input, $0.15/M cached =
0.05x, no cache-write surcharge), a fresh start costs ~43k–73k base-input units:

| Prior thread context (`C_old`) | Opus 5.5: requests to break even | Gemini 4 Neon: requests to break even |
|---|---:|---:|
| 100k | 21–48 | 15–26 |
| 150k | 10–23 | 8–14 |
| 250k | 5–11 | 4–7 |
| 350k | 3–7 | 3–5 |

With a 72k fresh start, Opus break-even rises ~1.5x; Fable 5.2 break-even is ~2x Opus. At the measured median post-PR
context (258k Claude, 231k Neon) and typical PR length (median 39 requests Claude, 24 requests Neon), a new thread
clearly wins.

### b. Antigravity / Gemini 4 Neon (147 sessions)

- Median requests per PR: **24**; median context carried after a PR: **231k**; fresh start: median **43k**; sessions
  with >1 PR: 1 (3 distinct PRs).
- Break-even at measured medians: **5 to 8 requests**.

### c. Codex

- On GPT-5.6 and later, default prompt cache retention is 30 minutes, writes cost 1.25x and reads 0.1x (documented).
  One-shot review delegates already start fresh.

### d. Quality evidence (documented / published, indirect)

- Laban et al. ([arXiv 2505.06120](https://arxiv.org/abs/2505.06120)) measured an average 39% performance drop when
  tasks are spread across multiple turns.
- [Chroma Context Rot](https://www.trychroma.com/research/context-rot) measured consistent performance degradation with
  increasing input length.
- Anthropic's [costs page](https://code.claude.com/docs/en/costs) recommends clearing between tasks: *"Stale context
  wastes tokens on every subsequent message."*

None of these directly tests "one thread per PR" as a controlled experiment.

## 2. When staying in the same thread is better

- Opening the PR for work already built in the thread (`pull_request_create`), and CI fixes or review rounds on that
  thread's own PR (measured median: 6.5 requests).
- Work that depends on uncommitted edits in the current checkout (`t3_thread_launch` does not copy uncommitted edits).
- A small, warm thread (under ~100k tokens, used within 1 hour on Claude or 15 minutes on Neon) when the user chooses
  to stay.
- Stacked PRs still benefit from a new thread launched from the local base branch (`startFromOrigin: false`).

## 3. Global rule behavior & reload mechanics (documented)

- **Claude Code** reads `CLAUDE.md` at session start and holds it in memory until `/clear`, `/compact`, or restart.
- **Antigravity** injects always-on rules via the symlinked global file; active sessions should be treated as holding
  their startup snapshot unless compacted or restarted.
- **Codex** builds its rules once per run.
- **Grok** loads `~/.claude/CLAUDE.md` via Claude compatibility.
- **Avoid waking cold idle threads** just to announce a rule change: waking a thread idle >1 hour on Claude rewrites a
  median 189k tokens at 2x.
