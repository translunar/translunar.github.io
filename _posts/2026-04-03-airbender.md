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
Within a couple of hours, a Korean developer named Sigrid Jin (handle instructkr) had used OpenAI's Claude Code competitor, Codex, to do a "clean room" Python rewrite. I don't know the exact strategy Jin used, but <!-- link starts here --> here's how I'd have done it in a similar timeframe using Superpowers.<!--link ends here and should open collapsible extra comment box containing the below list as an aside -->
1. `/brainstorming` Have Agent One read the TypeScript and write up the design document.
2. Hand off the design document to Agent Two, naive to the original code, and ask it to `/brainstorming` writing a new design document adapting the TypeScript design into Python and Rust.
3. Have Agent One review Agent Two's design and see if Agent Two missed anything.
4. Have Agent Two `/writing-plans` write an implementation plan using `/test-driven-development`, then have an independent reviewer review the plan.
5. Go through and `/writing-skills` on any skill rewrites needed.
6. Use `/subagent-driven-development` to dispatch a team of subagents to implement the plan, with two reviews, one for plan consistency and a second for code quality.

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
<aside><p><b>tl;dr</b>: agents are only reliable to the extent they can be tested, and it might be useful to brush up on your evolutionary biology.</p></aside>
Acquisition and bring-up of a communication satellite arriving in orbit feels a little like juggling. The launch vehicle tosses a swarm of satellites up (four at once in our most recent block), and mission operators try to locate and "catch" them in the beams of ground stations as quickly as possible to "safe" the vehicles (to verify they're in good power states and get all subsystems deployed and running). Because of the cost of operating ground stations and the tendency of space vehicles to drop out of view over the course of their orbits, several vehicles are often competing for scarce ground antenna resources. Even in-service, many vehicles are vying for the attention of only a handful of operators. <!--Please check this paragraph and the next and see how common this stuff is in the space industry. I'll run it by my bosses but I want to make sure this is all common knowledge.-->

### Agent alignment in critical systems
To dramatize the situation a bit, no engineer wants their missed `CLAUDE.md` instruction to show up in as the root cause of an anomaly ticket. The solution, of course, is to rely on `CLAUDE.md` as little as possible, and instead to prefer testable instructions.

Coding agents, like humans, are fallible. In safety, reliability, and security, we often think of our measures in terms of a "swiss cheese" model, <!--TODO: include swiss cheese graphic here--> wherein the goal is to keep the holes in the various slices from lining up. When the holes in all the layers line up, an anomaly ticket must be filed. With safety, especially, we say that rules are written in blood. And so we see that preferring testable instructions is a relatively easy way to align agentic workers.
### How might one actually test `CLAUDE.md` instructions?
With skills, we ask a fresh subagent to try to do something and evaluate its performance against a pre-established set of benchmarks or evals.

With `CLAUDE.md`, it's harder. We can test instructions with fresh subagents, which also get all the `CLAUDE.md` <!--CHECK THIS--> files. However, the issue is often _not_ a fresh sub-agent, but one with its million-token context 93% full. Each iteration of red/green/refactor testing in such an environment is inherently much more expensive than with skills. <!--How much would this cost with Haiku, Sonnet, and Opus?-->

This is a consequence of where `CLAUDE.md` instructions versus skill instructions are injected, because contextual recency improves recall. <!--citation needed-->
### Actually, memory-based instructions are testable
Memory-based instructions aren't the subject of explicit tests, but rather implicit red/green/refactor cycles. Memories get added because Claude failed at something repeatedly. If the agent continues to fail, the memories get refactored. It seems likely that the same is true of MagicDocs, if not perhaps of my implementation just yet.

While this process can involve a human (such as with the `/memory` command, or less optimally, by cursing at the LLM), increasingly it occurs without a human in the loop, such as with subagents or agent teams.
### Context engineering resembles hypothesis-driven science
Most engineering is not hypothesis-driven. Test-driven design in a coding setting is an engineering discipline, not a science discipline. The closest engineers get to a hypothesis is, "Our design will be born out over the product lifetime." It broke? Oh. Null hypothesis. But we aren't measuring reality; we're just measuring our own achievements, in an Olympic-sized dose of humanism.

Context and prompt engineering, in contrast, has two dimensions:
1. Improvement in agentic pipelines is an evolutionary process, occurring by adaptation; and
2. Each adaptation is the outcome of a testable hypothesis.

Sometimes, as in skills, humans write the hypotheses together with the LLM and test them explicitly: _Is this instruction necessary? Is it sufficient?_
### Context engineering, adaptive processes, and human feedback
On my grad school interview visit to the University of Texas at Austin, synthetic biology professor Andy Ellington pitched me on using cells like computers. While I pursued a different project for my eventual doctoral research at UT, I loved thinking about this problem. Cells do computation. But unlike most computers, they compensate for poor error-correction by working in swarms. Over time, they mutate their own instructions; they adapt. One of the biggest challenges in synthetic biology is providing cells with instructions that they don't simply shed in favor of their own paperclip-maximizing interests.

Increasingly, the goal of agentic design is to build a system where the LLM is implicitly writing, testing, and improving its own hypotheses implicitly. Many are already experimenting <!--link to that self-improving social media thing I found which has a blog about the skills it's working on--> with taking themselves out of the loop in a limited fashion. As agentic memory improves, I expect we'll need to think more about the places where feedback loops ought to include humans in order to prevent the accumulation of (to us) deleterious adaptations.

<aside><p>Attribution</p><p>I used Claude for the editing, research, illustration, and verification, but not for the writing, of this article.</p>
<p>Claude Code almost exclusively wrote the documents in the <a href="https://www.github.com/translunar/airbender/">airbender repository</a>.</p></aside>

<!--
Scaffolding — delete when written:

Links for this section:
- Airbender repo: [github.com/translunar/airbender](https://github.com/translunar/airbender)
- Plugin install: `claude plugin marketplace add translunar/airbender`
- Chapter 4 decision tree: [What to Put Where](https://github.com/translunar/airbender/blob/main/docs/04-what-to-put-where.md)

Points to hit:
- The lens that makes sense of all of this is testability
- Skills: explicitly testable (RED/GREEN/REFACTOR with pressure scenarios)
- MagicDocs: implicitly testable (development cycle surfaces and corrects stale/wrong docs)
- CLAUDE.md: no feedback loop. Instructions persist silently whether they work or not.
- Doesn't mean CLAUDE.md is useless — good for environmental priors ("use pnpm not npm"). Behavioral instructions belong in testable mechanisms.
- Context position matters: instructions at the point of action (skills, tool results) outperform instructions loaded thousands of tokens ago (CLAUDE.md)
- Folksy close
-->


