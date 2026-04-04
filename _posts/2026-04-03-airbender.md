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

## The stakes

Anthropic often tells people to put instructions for Claude Code in a `CLAUDE.md` file. At my day job, we build communication satellites for countries like [Taiwan](https://spacenews.com/astranis-clinches-115-million-taiwan-deal-despite-satellite-setback/) (whose undersea fiber optic cables keep getting [cut by the PRC](https://www.cnn.com/2025/02/25/asia/taiwan-detains-ship-undersea-cable-intl-hnk)), so minimizing unpredictability in our systems is exceptionally important. There, I noticed, over and over again, that Claude Code would ignore `CLAUDE.md` instructions — and more than that, I observed that skills worked really, really well. Not perfectly! But nevertheless exceptionally well.

Moreover, I had a major concern: there's no way to do test-driven development on `CLAUDE.md` files to determine what instructions actually matter. TDD does exist for Claude skills (one implementation of the prompts for this is found in Jesse Obra's [superpowers](https://github.com/obra/superpowers/blob/main/skills/test-driven-development/SKILL.md) plugin), and it works well. So why would Anthropic rely internally on a system that can't easily be tested, except perhaps under Monte Carlo conditions? The mystery deepened when the [source maps for Claude Code leaked](https://www.theregister.com/2026/03/31/anthropic_claude_code_source_code/), and it became apparent that there existed yet another strategy for Claude to store information.

Relying on the source maps and reverse-engineered Claude Code prompts, and with a lot of rubber ducking with Claude Code, I wrote a [five-chapter manual](https://github.com/translunar/airbender/tree/main/docs) and built an [open-source plugin](https://github.com/translunar/airbender) that replicates one of Anthropic's internal systems using public primitives. Here's what I found.

<!-- TODO: Expand sections below. Bullets are scaffolding from the design spec. -->

## The observation

<!-- 
- CLAUDE.md is what Anthropic tells everyone to use
- In practice, instructions placed there get ignored under pressure
- Skills work much better — not perfectly, but exceptionally well
- Key realization: you can test skills (RED/GREEN/REFACTOR with subagent pressure scenarios) but you can't test CLAUDE.md
- The "over-skilling" problem: many people mansplain to Claude how to do things it already knows. Even Haiku scored 2.5/5 on simple MVP evals without any special prompting — it mostly knows what to do already
- The question becomes: where should instructions actually go, and what makes some placements work better than others?
-->

## The leak and the research

<!--
- The source maps leaked (link: The Register). Matter-of-fact, no editorial.
- Revealed Claude Code has six different mechanisms for storing/loading information, not just CLAUDE.md
- Using the leaked prompts and a reverse-engineered codebase (cite the two repos), researched how each mechanism works
- Wrote a five-chapter manual documenting the architecture (link to manual)
- The core finding: context position matters. CLAUDE.md lands in section 8-9 of the system prompt, after five sections of Anthropic's behavioral instructions. Skills arrive as tool results at the point of action — much more recent in context.
- The decision tree emerges (INCLUDE IMAGE) — routes information to Hook/MagicDocs/Skill/CLAUDE.md/Memory/None based on properties of the information, not its topic
- Also discovered MagicDocs — an internal Anthropic system for auto-maintained architectural documentation, gated to internal builds
-->

![Decision tree for where to put instructions in Claude Code](/assets/images/airbender-decision-tree.png)

## Building and validating MagicDocs

<!--
- Replicated MagicDocs using public primitives: skills, subagents, hooks
- The design: terse insights dispatched to fresh Sonnet subagents (background, fire-and-forget), plus a Stop hook safety net for structural drift
- Key design decisions: main agent picks the target doc (not the subagent), Read/Edit tools only, fresh agent per insight (matches Anthropic's actual architecture)
- Validated with TDD — RED/GREEN/REFACTOR for documentation
  - RED baseline (no MagicDocs philosophy): both Sonnet and Haiku scored poorly. Accepted non-architectural info, appended instead of updating in-place, verbose
  - GREEN (full philosophy): Sonnet improved significantly. Correctly rejected CLAUDE.md-appropriate content. In-place updates, terse.
  - REFACTOR: discovered Haiku started editing unrelated files when given Glob access — a scope violation the prompt didn't anticipate. Dropped Glob, kept Read/Edit only. Scope violations eliminated.
- The surprising finding: the subagent didn't need much instruction to do a good job. What it needed was constraints (what NOT to do) and the right toolset.
- Three-channel update system: insight dispatch (primary, during session), manual invocation, Stop hook pruning (safety net, session exit)
-->

## The takeaway

<!--
- The lens that makes sense of all of this is testability
- Skills: explicitly testable (RED/GREEN/REFACTOR with pressure scenarios)
- MagicDocs: implicitly testable (the development cycle surfaces and corrects stale/wrong documentation)
- CLAUDE.md: no feedback loop. Instructions persist silently whether they work or not.
- This doesn't mean CLAUDE.md is useless — it's good for environmental priors ("use pnpm not npm"). But behavioral instructions belong in testable mechanisms.
- Context position matters too: instructions at the point of action (skills, tool results) outperform instructions loaded thousands of tokens ago (CLAUDE.md)
- Link to the repo
- Folksy close
-->

The manual, plugin, and all design documentation are available at [github.com/translunar/airbender](https://github.com/translunar/airbender).
