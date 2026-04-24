---
layout: post
title: "Stop putting everything in CLAUDE.md"
author: Juno
image: /assets/images/airbender-decision-tree.png
description: "Claude Code has four mechanisms for shaping agent behavior. Most people use one. I researched how they actually work and validated my findings with TDD."
tags: ai claude-code context-engineering tdd agentic
toc: true
excerpt: "Claude Code has four mechanisms for shaping agent behavior, but most people put everything in CLAUDE.md — where it gets buried after five sections of Anthropic's own instructions. I researched how the system actually works and validated my findings with TDD. The key insight: testability, not placement, is what makes instructions stick."
---

## The stakes
At my day job, we build trunk-level communication satellites for countries like [Taiwan](https://spacenews.com/astranis-clinches-115-million-taiwan-deal-despite-satellite-setback/) (whose undersea fiber optic cables keep getting [cut by the PRC](https://www.cnn.com/2025/02/25/asia/taiwan-detains-ship-undersea-cable-intl-hnk)), so minimizing unpredictability in our systems is exceptionally important.

In that role, I noticed over and over again that Claude Code would ignore `CLAUDE.md` instructions, and that it responded much better to [agent skills](https://agentskills.io/home).

## The observation
This experience contrasts with Anthropic's documentation and marketing materials, wherein Anthropic tells people to put instructions for Claude Code in a `CLAUDE.md` file. For example, it was a key focus on 24 March in a webinar called _[Claude Code Advanced Patterns](https://www.anthropic.com/webinars/claude-code-advanced-patterns)_, and it's front and center in _[Best Practices for Claude Code](https://code.claude.com/docs/en/best-practices)_ from their official docs.

Moreover, I had a major concern: there's no simple way to utilize test-driven development (TDD) on `CLAUDE.md` files — and thus no way to verify that any given instruction is adding value.
<aside><p>With normal software unit tests, we only ask if instructions are <i>sufficient</i> for obtaining a desired behavior. With agents, we also want to know if an instruction is <i>necessary</i> for the behavior (that is, if the agent could perform the task correctly without the instruction). An instruction should be both necessary and sufficient to merit inclusion, or it's simply <a href="https://en.wiktionary.org/wiki/kipple">kipple</a> polluting the context.</p></aside>
Agentic TDD resembles hypothesis-driven scientific discovery. One [paradigmatic red/green/refactor implementation exists in Jesse Obra's superpowers plugin](https://github.com/obra/superpowers/blob/main/skills/writing-skills/SKILL.md): the agent writes the evals (quiz questions for LLMs) it expects to fail, tests them with a clean sub-agent, writes a skill focused on the things the sub-agent got wrong, repeats the test phase but now with the skill introduced, and refactors and repeats the process if gaps remain.

Sometimes the red phase indicates that the question is ill-posed and should be reformulated, or that the skill should be discarded entirely because Claude already knows how to do it.

If we want to add statistical rigor to the testing of our non-deterministic agents, we can run it as a Monte Carlo with parallel sub-agents.
## The mystery
So why would Anthropic rely internally on a vibes-based system over one whose behavior is testable and quantifiable? The mystery deepened when the [source maps for Claude Code leaked](https://www.theregister.com/2026/03/31/anthropic_claude_code_source_code/), and it became apparent that there existed yet another strategy — MagicDocs — for providing instructions to Claude. I wondered how best to guide less experienced engineers on which instructions should go where, and whether ideal context placement might depend on the desired outcome.

Relying on the source maps and reverse-engineered Claude Code prompts, and with a lot of [talking to rubber ducks](https://en.wikipedia.org/wiki/Rubber_duck_debugging), I wrote a [five-chapter manual](https://github.com/translunar/airbender/tree/main/docs) and built an [open-source plugin](https://github.com/translunar/airbender). Then, days into the research, Anthropic [removed MagicDocs entirely](/blog/2026/04/05/magicdocs-removed) — validating my hypothesis that the experiment hadn't worked out and simplifying the decision tree considerably.
## The leak and the research
[Wavespeed has published](https://wavespeed.ai/blog/posts/what-is-claw-code/) an excellent timeline of the Claude Code leak. They write,
> **No customer data, no API credentials, no model weights.** What was exposed: approximately 512,000 lines of TypeScript across ~1,900 files — the query engine, tool system, multi-agent orchestration logic, context compaction, and 44 feature flags covering functionality that’s built but not yet shipped.
> Those feature flags are the most strategically sensitive part. Compiled code sitting behind flags that evaluate to `false` in the external build isn’t just implementation detail — it’s a product roadmap. Competitors can now see what Anthropic has built and is considering shipping. That strategic surprise can’t be un-leaked.
<aside><p>There has been some debate on the Internet as to whether the "source code" was accidentally posted to npm or the "source maps." With TypeScript and JavaScript, there isn't much distinction "source code" and "source maps," since the source maps contain the original source code. In a TypeScript distribution, the TypeScript (which is a superset of JavaScript) gets <i>transpiled</i>, meaning reduced to JavaScript. Then the Javascript is bundled, which means the files are concatenated together, the whitespace is removed, control flow is restructured, and variable names are made as short as possible. The source code exists in an obfuscated form in any Claude Code release, but the source maps are the keys to the kingdom.</p></aside>
Within a couple of hours, a Korean developer named Sigrid Jin (handle instructkr) had used OpenAI's Claude Code competitor, Codex, to do a "clean room" Python rewrite. I don't know the exact strategy Jin used, but

<details>
<summary>here's how I'd have done it in a similar timeframe using Superpowers.</summary>

1. `/brainstorming` Have Agent One read the TypeScript and write up the design document.
2. Hand off the design document to Agent Two, naive to the original code, and ask it to `/brainstorming` writing a new design document adapting the TypeScript design into Python and Rust.
3. Have Agent One review Agent Two's design and see if Agent Two missed anything.
4. Have Agent Two `/writing-plans` write an implementation plan using `/test-driven-development`, then have an independent reviewer review the plan.
5. Go through and `/writing-skills` on any skill rewrites needed.
6. Use `/subagent-driven-development` to dispatch a team of subagents to implement the plan, with two reviews, one for plan consistency and a second for code quality.

</details>

I relied on two main sources for my research: Claw Code and [a separate set of a couple hundred reverse-engineered prompts](https://github.com/Piebald-AI/claude-code-system-prompts) Piebald AI automatically pulls from every Claude Code release.
### Four mechanisms for user control of behavior
A key take-away from analysis of Claw Code and the prompts is that Claude Code provides four mechanisms for shaping Claude's behavior: `CLAUDE.md`, memory, skills, and hooks. Three are prompt-based and non-deterministic, and one (hooks) is purely mechanical and deterministic.
<aside><p>When I started this research, there was a fifth mechanism — MagicDocs, an internal Anthropic feature that auto-maintained architectural documentation. Anthropic <a href="/blog/2026/04/05/magicdocs-removed">quietly removed it</a> in v2.1.91 after 134 days with zero content modifications. The decision tree and analysis below reflect the simplified four-mechanism model.</p></aside>

<figure>
<img src="/assets/images/system-prompt-segments.svg" alt="Diagram showing the 10 sections of Claude Code's system prompt. Sections 1-5 (Intro, Output Style, System Rules, Doing Tasks, Actions) are static across all sessions. Below the dynamic boundary, sections 7-10 (Environment, Project Context, CLAUDE.md, Runtime Config) are project-specific. CLAUDE.md lands at section 9 of 10 — after five sections of Anthropic's behavioral instructions." />
<figcaption>Claude Code's system prompt is assembled from 10 sections in a fixed order. Your CLAUDE.md instructions land at section 9.</figcaption>
</figure>

From [Claw Code](https://github.com/instructkr/claw-code), we know that the system prompt consists of ten segments, shown above. Following the system prompt is the actual conversation. At session start, a `<system-reminder>` block lists available skills indexed by `when_to_use` and `description` — but their full content stays on disk until invoked. At each turn, an additional system reminder provides up to five memory files that a Sonnet subagent has selected based on the memory `description` field. When a skill is invoked, its content arrives as a tool result — the freshest position in context.
As mentioned, hooks aren't part of the context. They're shell commands defined in `settings.json` that fire at lifecycle events — purely mechanical, no LLM judgment involved:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "cd $PROJECT_DIR && pnpm lint --fix $FILEPATH"
      }]
    }]
  }
}
```

[Chapter 2 of the Airbender Docs](https://github.com/translunar/airbender/blob/main/docs/02-context-engineering.md) goes into more detail about the mechanization of these different components and how the user-responsive pieces of Claude Code work.
### "But how do I decide where to put this instruction?"
It seemed to me that if Claude Code has four different mechanisms for providing mutable instructions to the LLM, users need a way to decide where any given instruction should go. Claude and I arrived at the decision tree through an informal red/green/refactor discussion, where we both tried to produce examples that would falsify our hypotheses. You can read the full analysis, including few-shot examples, in [Chapter 4: What to Put Where](https://github.com/translunar/airbender/blob/main/docs/04-what-to-put-where).

<figure>
<img src="/assets/images/airbender-decision-tree.png" alt="A decision tree for deciding where an instruction should go. At the top level, 'Can it be fully enforced without Claude's judgment?' If yes, HOOK; if no, 'Needs to shape assumptions (prior) or arrive fresh at point of action?' If at point of action, SKILL; if prior, 'Tied to a codebase location, or to an activity?' If location, CLAUDE.md; if activity, MEMORY." />
<figcaption>Three questions route any instruction to the right persistence mechanism.</figcaption>
</figure>
I originally built a skill to mechanize this decision tree, but TDD testing showed the model scores 96% on these classifications without any skill at all — it already knows the difference between a hook and a memory. The [red/green/refactor sequence](https://github.com/translunar/airbender/blob/main/skills/classify-info/TESTING.md) is preserved as a case study in agentic evals.


## Building and validating MagicDocs
<aside><p>This section describes work I did before Anthropic removed MagicDocs. The system works, but given the removal, I'd now recommend using subdirectory-scoped CLAUDE.md files and memory instead. The work is preserved here as a case study in agentic TDD.</p></aside>
Without Anthropic's internal access to Claude Code features, I couldn't implement MagicDocs exactly as it is in Claude Code, so it became necessary to invent a design that reproduced a MagicDocs analogue without the internal tooling. I asked for a [checklist of design questions](https://github.com/translunar/airbender/blob/main/docs/reference/magicdocs-design-questions.md) we would need for this analogue. I also asked Claude to annotate each with the approach Anthropic took, and to order the decisions according to their downstream impact on other design elements.

These design questions left me with doubt. In particular, injecting a note about our MagicDocs into `CLAUDE.md` seemed a fragile approach, given my earlier claim that instructions often go into that file to die. Moreover, internally, Anthropic runs its MagicDocs agent during "idle cycles" — but what is an idle cycle for someone without unlimited tokens, and could we mimic this behavior with a hook?

I asked Claude to write me two implementation plans — the first to [test our design hypothesis in minimum viable product form](https://github.com/translunar/airbender/blob/main/docs/plans/2026-04-02-magicdocs-mvp), and the second for the implementation. Indeed, the MVP revealed a tendency for the MagicDocs subagents to modify or even delete out-of-scope files, which necessitated modifications to the full plan.

Most of the system is encapsulated in a single skill within the airbender plugin, [`/setup-magicdocs`](https://github.com/translunar/airbender/blob/main/skills/setup-magicdocs/SKILL.md), which:
1. Explores the repository structure and identifies a few ways to segment into an initial set of MagicDocs, consulting the user on their preference;
2. Creates skeleton docs in `docs/magic/`;
3. Updates `CLAUDE.md`; and
4. Configures the Stop hook to do a pruning pass at session exit.

I also provide a `/create-magicdoc` skill for adding additional MagicDocs as the repository grows. This partially duplicates `/setup-magicdocs`, and de-duplicating these is an area for improvement.
<aside><p>A caveat: I put this system together while on vacation and without access to any large codebase to dogfood it on. Use at your own risk, and make backups of repositories you test it on!</p></aside>

## The takeaway
No engineer wants their missed `CLAUDE.md` instruction to show up as the root cause of an anomaly ticket. The solution is to rely on `CLAUDE.md` as little as possible, and instead to prefer testable instructions.
### How might one actually test `CLAUDE.md` instructions?
With skills, we ask a fresh subagent to try to do something and evaluate its performance against a pre-established set of benchmarks or evals.

With `CLAUDE.md`, it's harder. We can test instructions with fresh subagents, which also receive the project's `CLAUDE.md` files. However, the issue is often _not_ a fresh sub-agent, but one with its million-token context 93% full. Each iteration of red/green/refactor testing in such an environment is inherently much more expensive than with skills:

| Scenario | Haiku 4.5 | Sonnet 4.6 | Opus 4.6 |
|---|---|---|---|
| Skill TDD (fresh subagent, ~15K tokens) | $0.08 | $0.23 | $0.38 |
| `CLAUDE.md` TDD (fresh conversation, ~12K tokens) | $0.07 | $0.20 | $0.33 |
| `CLAUDE.md` TDD (93% full context, ~198K tokens) | $0.62 | $1.87 | $3.12 |
| `CLAUDE.md` TDD (93% full, with prompt caching) | $0.61 | $1.83 | $3.05 |

<aside><p>Full-context Haiku ($0.62/cycle) costs more than skill-based Opus ($0.38/cycle). Prompt caching barely helps when testing CLAUDE.md changes — only sections 1–8 of the system prompt (about 9,500 tokens) are cacheable, because the CLAUDE.md change at section 9 invalidates the cache for everything after it, including the conversation. Total savings: 2.4%. The <a href="https://github.com/translunar/airbender/blob/main/analysis/tdd_cost_model.py">cost model</a> is in the airbender repository.</p></aside>

This consistency in favoring skill-based instructions is a consequence of where `CLAUDE.md` tokens versus skill tokens are included in the context, because positional bias in recall is a [geometric property of causal transformer architecture](https://arxiv.org/abs/2603.10123). Moreover, [as the context window fills, the bias shifts from U-shaped to pure recency](https://arxiv.org/abs/2508.07479) — exactly the regime in which skills are favored over an early-context `CLAUDE.md` instruction.

<aside><p>Attribution</p><p>I used Claude for the editing, research, illustration, and verification, but not for the writing, of this article.</p>
<p>Claude Code almost exclusively wrote the documents in the <a href="https://www.github.com/translunar/airbender/">airbender repository</a>.</p></aside>



