# Context-management techniques: Astra local checks and literature review (2026-11-28)

Researched by Codex (`gpt-6.2-astra`) in two rounds on 2026-11-27/28. Preserved verbatim from Astra's two replies, with
two edits for this public repository: local home-directory paths and internal thread links were replaced with
repository-relative links.

---

## Round 1: local checks

I ran three checks myself. **Filtering GitHub responses produced the clearest result.** The history experiment also
changed my recommendation about when to use `git bisect`.

1. **Ask GitHub for the fields the thread needs.** I fetched PR #2 and compared the full response with a summary
   containing its number, state, merged flag, draft flag, and mergeability state. The full response was **22,194
   bytes**. The summary was **91 bytes**, about **244× smaller**, and answered whether the PR was open, merged, or a
   draft. I’d use this for routine PR status checks. If the question is “which test failed?”, select the failed check
   names and links instead. This measures bytes entering the conversation. Billed token savings would need a separate
   measurement. [PR #2](https://github.com/wells1225/ai-github-efficiency-builder/pull/2)

2. **Choose the history search based on what you know.** I tested two ways to find when `300000` first appeared in this
   repo’s README. In a temporary clone, `git bisect run` took **four automated probes**. On the same history,
   `git log --reverse -S300000 --format='%h' -- README.md` returned the same commit, `ddcf8c6`, in one query. My revised
   recommendation is to use `git log -S` when you know the literal that changed, and use automated bisection when you
   can recognize the change only by running a test. Both keep the agent from reading successive diffs into the thread.
   [Introducing commit](https://github.com/wells1225/ai-github-efficiency-builder/commit/ddcf8c6)

3. **Search the research archive for a claim before opening whole reports.** I inventoried this repo’s tracked
   material. The README is **9,287 bytes**, the two guides total **59,217 bytes**, and the twelve research records total
   **430,671 bytes**. That puts **86%** of the material in the underlying research. For a question like “where did the
   300000 setting come from?”, start with `rg -n -C 2 '300000' README.md docs/`, then open the cited research passage if
   the answer needs verification. That gives threads a concrete reading order using files you already maintain. The 86%
   describes the archive’s share of the material, not an observed token saving. [README](../README.md) ·
   [Token-use guide](../docs/token-use.md)

## Round 2: literature review

I found three results in original coding-agent studies that add to your existing guide. **The one I’d apply first is
selective handoffs.** You already start a fresh thread for each new PR. The research shows that *which piece of prior
work you carry forward* can matter more than how short you make the handoff.

### 1. Carry a prior finding into a new thread when you can name the dependency

**Measured.** SWE Context Bench tested five ways of reusing prior work across 99 related coding tasks, for 495 runs.
With no prior experience supplied, the agent solved **26 of 99**. When researchers supplied a summary from a known
relevant task, it solved **34 of 99**. When the agent chose summaries for itself, it solved **22 of 99**. Average
estimated cost followed the same pattern: **$0.79**, **$0.77**, and **$0.85** per task, respectively. The study’s
strongest result was the difference in successful fixes. The direct dollar saving over a fresh start was small.
[SWE Context Bench, Tables 3–5](https://arxiv.org/html/2602.08316v1)

**What I’d change in your workflow.** Keep the goal, branch, and files in the handoff. For each finding carried over,
finish the sentence *“This matters to the next PR because …”* For example, “The previous PR changed the persisted
settings format. This PR reads that format, so run the migration fixture.” That gives the new thread a decision, a
reason, and a way to verify it. The study used a known match between tasks. Naming the dependency is my practical
approximation of that condition.

### 2. Retire old tool output while preserving what the agent decided

**Measured.** JetBrains Research compared keeping the entire interaction, replacing older tool outputs with placeholders
while retaining the agent’s actions and reasoning, and asking another model to summarize the history. On SWE-bench
Verified, masking older outputs roughly halved cost against keeping the raw history and was cheapest in **four of five**
model configurations. On Gemini 2.5 Flash, summarization averaged **52 turns** versus **44** with masking. A shorter
history at a given moment can still produce a longer, more expensive run. The researchers also found that the best
number of turns to retain changed with the agent software.
[JetBrains study and results](https://kairos.jetbrains.com/research/articles/the-complexity-trap)

**What I’d test in T3.** Your guide already covers limiting a log when it first comes in. This finding concerns a log
that was genuinely useful **ten steps ago**. Once an agent has isolated a failure, the durable record might be
“migration fixture fails on the old schema; reproduction command and log path attached.” The original output can stay
available for reopening. Automatic removal from future requests requires support in the thread runtime. I haven’t
verified that T3 exposes that control, so I’d treat this as a runtime experiment before changing your compaction
setting.

### 3. Give a large file read a specific question

**Measured.** SWE-Pruner tested a coding agent that supplied a question with each focused read, such as “Where does this
value get its default?” A small model selected source lines relevant to that question. In a randomized **50-task**
SWE-bench Verified comparison, the baseline solved **31 of 50** and the question-guided approach solved **32 of 50**
using **31% fewer tokens**. Generic summarization solved **28 of 50**. The one-task improvement is too small to
establish a reliability gain. The more convincing result is the token reduction at a similar solve rate.
[SWE-Pruner, method and Table 3](https://arxiv.org/html/2601.16746v1)

**What I’d use today.** When investigating a large file, express the missing fact in the search itself. For “why does an
old settings file get this default?”, retrieve the deserializer, the missing-field branch, and the compatibility test.
Expand to the surrounding implementation if those disagree. This gives the agent a stopping condition that “read the
first 100 lines” lacks. Ordinary symbol search can implement the habit today. The published **31%** belongs to the
researchers’ trained selector and should not be used as a forecast for your threads.

Grok’s completed X research adds a useful check on the earlier ideas. It found a practitioner experiment with measured
input-token reductions from filtering structured tool results, but no before-and-after token counts for automated Git
bisection or Claude’s path-scoped rules. I’d keep both in the guide as techniques and reserve savings claims for
workflows that have actually been metered. [Grok’s research](context-techniques-x-grok-2026-11-28.md)

For your own measurements, I’d use **cost per successfully completed task**, with request count and cache reads
recorded alongside it. That captures both failure modes the studies exposed: a cheap handoff that sends the next agent
down the wrong path, and a compact thread that takes extra turns to finish.
