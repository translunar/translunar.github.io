---
layout: post
title: "Stop putting everything in CLAUDE.md"
author: Juno
image: /assets/images/airbender-decision-tree.png
description: "Claude Code has six mechanisms for shaping agent behavior. Most people use one. I researched how they actually work, built an open-source MagicDocs system, and validated it with TDD."
tags: ai claude-code context-engineering tdd agentic
toc: true
---

<!-- SPEC: Mystery arc, ~2000 words. See docs/plans/ for full design spec. -->
<!-- Audience: Smart engineers who may not use Claude Code. Anthropic research staff secondary. -->
<!-- Thesis: Testability is the missing lens for AI agent instructions. -->
<!-- Tone: Professional-folksy. Direct, casually credentialed. Not attacking Anthropic. -->

<!-- TASK LIST (for Claude):
- [x] Fix CSS: inline code backticks render with no foreground/background contrast
- [ ] Make "here's how I'd have done it" open a collapsible aside containing the numbered list
- [ ] Create image: system prompt segments (horizontal/vertical bar of 10 sections from Ch2 table)
- [x] Research: context ordering. Results in docs/research/context-ordering.md. Key: CLAUDE.md at section 9, memories via system-reminder per-turn, skills as tool results (most recent). Memory subagent timing unclear. Cold-start selection not answered. MagicDocs NOT injected into main context — pre-loaded into post-conversation subagent. Subagents likely receive CLAUDE.md but not confirmed.
- [ ] Include a hook example (JSON syntax) so readers see it's config, not a prompt
- [x] Check: satellite paragraph COMMENTED OUT and moved to end. Swiss cheese cut.
- [x] Create image: swiss cheese model graphic — CUT (section removed)
- [x] Research: subagents and CLAUDE.md. CONFIRMED empirically — subagents receive CLAUDE.md via system-reminder blocks. Hedge removed from article.
- [x] Research: TDD cost estimates. INSERTED as table + aside in "How might one actually test" section. Links to cost model on GitHub.
- [x] Find citation: "contextual recency improves recall." INSERTED as inline citation (Liu et al. 2023) in same section.
- [x] Find self-improving agent blog: NOT FOUND. Section containing the reference was cut (moved to future post draft).
- [x] Organizational advice: DONE. Sections 5+6 saved to docs/drafts/context-engineering-as-science.md. Satellite paragraph commented out at end. Swiss cheese cut. Takeaway tightened to: bridge → cost table → implicit testing → close.
- [ ] Delete remaining scaffolding comments once all sections are written
-->

<!-- TODO: the inline code indicated with backticks is currently rendering with no contrast between foreground and background color. Please fix the CSS -->

## The stakes
At my day job, we build trunk-level communication satellites for countries like [Taiwan](https://spacenews.com/astranis-clinches-115-million-taiwan-deal-despite-satellite-setback/) (whose undersea fiber optic cables keep getting [cut by the PRC](https://www.cnn.com/2025/02/25/asia/taiwan-detains-ship-undersea-cable-intl-hnk)), so minimizing unpredictability in our systems is exceptionally important.

In that role, I noticed over and over again that Claude Code would ignore `CLAUDE.md` instructions, and that it responded much better to [agent skills](https://agentskills.io/home).
## The observation
This experience contrasts with Anthropic's documentation and marketing materials, Anthropic tells people to put instructions for Claude Code in a `CLAUDE.md` file. For example, it was a key focus on 24 March in a webinar called _[Claude Code Advanced Patterns](https://www.anthropic.com/webinars/claude-code-advanced-patterns)_, and it's front and center in _[Best Practices for Claude Code](https://code.claude.com/docs/en/best-practices)_ from their official docs.

Moreover, I had a major concern: there's no simple way to utilize test-driven development (TDD) on `CLAUDE.md` files — and thus no way to verify that any given instruction is adding value.
<aside><p>With normal software unit tests, we only ask if instructions are <i>sufficient</i> for obtaining a desired behavior. With agents, we also want to know if an instruction is <i>necessary</i> for the behavior (that is, if the agent could perform the task correctly without the instruction). An instruction should be both necessary and sufficient to merit inclusion, or it's simply <a href="https://en.wiktionary.org/wiki/kipple">kipple</a> polluting the context.</p></aside>
Agentic TDD resembles hypothesis-driven scientific discovery. One [paradigmatic red/green/refactor implementation exists in Jesse Obra's superpowers plugin](https://github.com/obra/superpowers/blob/main/skills/writing-skills/SKILL.md): the agent writes the evals (quiz questions for LLMs) it expects to fail, tests them with a clean sub-agent, writes a skill focused on the things the sub-agent got wrong, repeats the test phase but now with the skill introduced, and refactors and repeats the process if gaps remain.

Sometimes the red phase indicates that the question is ill-posed and should be reformulated, or that the skill should be discarded entirely because Claude already knows how to do it (in the industry, we call this "mansplaining," and the effect it has on agents is differently bad from its effect on our respected coworkers).

If we want to add statistical rigor to the testing of our non-deterministic agents, we can run it as a Monte Carlo with parallel sub-agents.
## The mystery
So why would Anthropic rely internally on a vibes-based system over one that is scientifically testable? The mystery deepened when the [source maps for Claude Code leaked](https://www.theregister.com/2026/03/31/anthropic_claude_code_source_code/), and it became apparent that there existed yet another strategy — MagicDocs — for providing instructions to Claude. I wondered how best to guide less experienced engineers on which instructions should go where, and whether ideal context placement might depend on the desired outcome.

Relying on the source maps and reverse-engineered Claude Code prompts, and with a lot of [talking to rubber ducks](https://en.wikipedia.org/wiki/Rubber_duck_debugging), I wrote a [five-chapter manual](https://github.com/translunar/airbender/tree/main/docs) and built an [open-source plugin](https://github.com/translunar/airbender) that replicates one of Anthropic's internal systems using public primitives.
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

I relied on two main sources for my research: Claw Code and [a separate set of 250+ reverse-engineered prompts](https://github.com/Piebald-AI/claude-code-system-prompts) Piebald AI automatically pulls from every Claude Code release.
### Five mechanisms for user control of behavior
A key take-away from analysis of Claw Code and the prompts is that Claude Code provides four public-facing mechanisms for shaping Claude's behavior (`CLAUDE.md`, memory, skills, hooks) and a fifth internal mechanism (MagicDocs). Four of the strategies are prompt-based and non-deterministic, and one (hooks) is purely mechanical and deterministic.

<!-- An image: a horizontal or vertical bar divided into segments matching the table from docs/02-context-engineering.md. The segments are each labeled in larger text with the Section column, and have smaller text matching the "what it contains" column. Dynamic Boundary does not get to be a section; this is instead an arrow pointed directly between the Actions and Environment segments. The Intro through Actions portion is labeled something like "same across all Claude Code sessions" and the Environment through Runtime config section is labeled "ephemeral: configuration, situational awareness". -->

From Claw Code <!--verify-->, we know that the system prompt consists of ten segments, shown above. Following the system prompt is the actual conversation. The conversation begins with a `<system-reminder>` block with available skills indexed by `when_to_use` and `description`. Then, at the beginning of each turn, or conversational exchange, an additional system reminder is provided with a list of up to five memory files a subagent has decided are relevant based on the memory `description` field. <!-- Is the ordering here correct? Please look at the claw-code source to confirm. Does this subagent re-run every turn, or just periodically? If periodically, how do the relevant memories get provided at the beginning of a conversation, before we know the topic? Also, where do MagicDocs get injected? In Anthropic's internal usage, are they also injected via CLAUDE.md and does this provide enough information for the model to know to look for them before looking at source code? Please read the prompts for this info. -->

As mentioned, hooks aren't part of the context. They are defined in `settings.json` and might look like: <!-- Let's include a hook example here so people know what one looks like. I'm worried describing it using plain English will make it seem like a prompt and be confusing. -->

[Chapter 2 of the Airbender Docs](https://github.com/translunar/airbender/blob/main/docs/02-context-engineering.md) goes into more detail about the mechanization of these different components and how the user-responsive pieces of Claude Code work.
### "But how do I decide where to put this instruction?"
It seemed to me that if Anthropic has five different mechanisms for providing mutable instructions to the LLM, it also needs a mechanism for classifying where any given instruction should go. This is part is speculative, but Claude and I arrived at it through an informal red/green/refactor discussion, where we both tried to produce examples that would falsify our hypotheses. You can read the full analysis, including few-shot examples, in [Chapter 4: What to Put Where](https://github.com/translunar/airbender/blob/main/docs/04-what-to-put-where).

<figure>
<img src="/assets/images/context_decision_tree.png" alt="A decision tree for deciding where an instruction should go. At the top level, 'Can it be fully enforced without Claude's judgment?' If yes, HOOK; if NO, 'Describes how things work or prescribes behavior?' If describes, MAGIC DOCS; if prescribes, 'Needs to shape assumptions (prior) or arrive fresh at point of action?' If at point of action, SKILL; if prior, 'Always relevant or sometimes?' If always, CLAUDE.md; if sometimes, MEMORY." />
<figcaption>This decision tree provides a series of heuristics for where to put instructions for Claude Code.</figcaption>
</figure>
I built a [skill to mechanize this decision tree, `/classify-info`](https://github.com/translunar/airbender/blob/main/skills/classify-info/SKILL.md). If you wish to understand how Claude and I arrived at this skill, I suggest reading [its red/green/refactor sequence and evals](https://github.com/translunar/airbender/blob/main/skills/classify-info/TESTING.md).

<!--
Scaffolding — delete when written:

Links for this section:
- Source maps leak: [The Register](https://www.theregister.com/2026/03/31/anthropic_claude_code_source_code/)
- Reverse-engineered codebase: [claw-code](https://github.com/instructkr/claw-code) (Python/Rust reimplementation)
- Extracted prompts: [claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts) (250+ prompt files)
- Manual: [five-chapter manual](https://github.com/translunar/airbender/tree/main/docs)
- Context engineering chapter: [Chapter 2](https://github.com/translunar/airbender/blob/main/docs/02-context-engineering.md)
- What to put where chapter: [Chapter 4](https://github.com/translunar/airbender/blob/main/docs/04-what-to-put-where.md)
- Building MagicDocs chapter: [Chapter 5](https://github.com/translunar/airbender/blob/main/docs/05-building-magicdocs.md)
- Subagent architecture: [Chapter 3](https://github.com/translunar/airbender/blob/main/docs/03-subagent-architecture.md)

Points to hit:
- Six mechanisms: CLAUDE.md, skills, memory, hooks, MagicDocs, insights
- Context position matters: CLAUDE.md lands in section 8-9 of the system prompt, after five sections of Anthropic's behavioral instructions. Skills arrive as tool results at the point of action.
- The decision tree (image below) routes information based on properties (testable? descriptive? always relevant?) not topic
- MagicDocs: internal Anthropic system for auto-maintained architectural docs, gated to internal builds
-->
## Building and validating MagicDocs
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
<!--
Scaffolding — delete when written:

Links for this section:
- MagicDocs design questions: [13 design decisions](https://github.com/translunar/airbender/blob/main/docs/reference/magicdocs-design-questions.md)
- Full spec: [full system spec](https://github.com/translunar/airbender/blob/main/docs/specs/2026-04-02-magicdocs-full.md)
- MVP spec: [MVP spec](https://github.com/translunar/airbender/blob/main/docs/specs/2026-04-02-magicdocs-mvp.md)
- TDD test results: [TESTING.md](https://github.com/translunar/airbender/blob/main/docs/magic/TESTING.md)
- Anthropic's MagicDocs agent prompt: [agent-prompt-update-magic-docs.md](https://github.com/Piebald-AI/claude-code-system-prompts/blob/main/system-prompts/agent-prompt-update-magic-docs.md)

Points to hit:
- Replicated MagicDocs using public primitives: skills, subagents, hooks
- Design: terse insights dispatched to fresh Sonnet subagents (background, fire-and-forget), plus Stop hook safety net
- Key decisions: main agent picks target doc (not subagent), Read/Edit tools only, fresh agent per insight
- TDD validation:
  - RED: both models scored poorly (1/5). Accepted non-architectural info, appended instead of updating, verbose
  - GREEN (full philosophy): Sonnet 3.5/5, Haiku 2.5/5. Correctly rejected CLAUDE.md content. In-place updates, terse.
  - REFACTOR: Haiku started editing unrelated files when given Glob — dropped Glob, kept Read/Edit only. Final: Sonnet 4.5/5, Haiku 4/5.
- Surprising finding: subagent didn't need much instruction. Needed constraints (what NOT to do) and the right toolset.
- Three channels: insight dispatch (primary), manual, Stop hook pruning (safety net)
-->
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

This is a consequence of where `CLAUDE.md` instructions versus skill instructions are injected, because contextual recency improves recall ([Liu et al. 2023, "Lost in the Middle"](https://arxiv.org/abs/2307.03172)).
### Actually, memory-based instructions are testable
Memory-based instructions aren't the subject of explicit tests, but rather implicit red/green/refactor cycles. Memories get added because Claude failed at something repeatedly. If the agent continues to fail, the memories get refactored. It seems likely that the same is true of MagicDocs, if not perhaps of my implementation just yet.

While this process can involve a human (such as with the `/memory` command, or less optimally, by cursing at the LLM), increasingly it occurs without a human in the loop, such as with subagents or agent teams.

<aside><p>Attribution</p><p>I used Claude for the editing, research, illustration, and verification, but not for the writing, of this article.</p>
<p>Claude Code almost exclusively wrote the documents in the <a href="https://www.github.com/translunar/airbender/">airbender repository</a>.</p></aside>

<!-- COMMENTED OUT: Satellite juggling paragraph. May reintroduce if it fits.
Acquisition and bring-up of a communication satellite arriving in orbit feels a little like juggling. The launch vehicle tosses a swarm of satellites up (four at once in our most recent block), and mission operators try to locate and "catch" them in the beams of ground stations as quickly as possible to "safe" the vehicles (to verify they're in good power states and get all subsystems deployed and running). Because of the cost of operating ground stations and the tendency of space vehicles to drop out of view over the course of their orbits, several vehicles are often competing for scarce ground antenna resources. Even in-service, many vehicles are vying for the attention of only a handful of operators.
-->

<!-- NOTE: Sections on "context engineering as science" and "adaptive processes / synthetic biology" saved to docs/drafts/context-engineering-as-science.md for a future blog post. -->


