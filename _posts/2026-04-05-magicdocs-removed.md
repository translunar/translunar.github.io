---
layout: post
title: "Anthropic quietly removed MagicDocs from Claude Code"
author: Juno
description: "MagicDocs was the least-modified of Claude Code's five oldest system prompts. It and /pr-comments — both legacy approaches from before the skills era — were removed in v2.1.91."
tags: ai claude-code magicdocs context-engineering
image: /assets/images/prompt-history.png
toc: false
excerpt: "Of the five system prompts that have existed for the entire observable history of Claude Code, MagicDocs was the least modified — 0 edits in the 134 days Piebald AI has been tracking Claude Code prompt updates. It was removed in v2.1.91 alongside /pr-comments, both legacy approaches that skills and memories now handle better."
---

MagicDocs — the internal Anthropic feature that auto-maintained architectural documentation inside Claude Code — was removed three days ago in [v2.1.91](https://github.com/anthropics/claude-code/releases/tag/v2.1.91). Anthropic's changelog doesn't mention it. The removal was [confirmed](https://github.com/Piebald-AI/claude-code-system-prompts/commit/ca9465e) by the Piebald-AI automated extraction bot, which tracks every prompt change across Claude Code releases. The same release also removed the `/pr-comments` slash command.

<aside><p>I noticed this while fact-checking an article I wrote about <a href="https://github.com/translunar/airbender/tree/main/docs">a five-chapter manual on how Claude Code manages context</a>, which includes <a href="https://github.com/translunar/airbender">an open-source MagicDocs implementation</a> using public primitives. The timing was coincidental; I didn't know about the removal until I checked the Piebald-AI changelog days later.</p></aside>

## What was MagicDocs?

MagicDocs was one of the more interesting discoveries in the [March 31 source maps leak](https://www.theregister.com/2026/03/31/anthropic_claude_code_source_code/). Several [leak](https://read.engineerscodex.com/p/diving-into-claude-codes-source-code) [analyses](https://dev.to/_53fb7c03dd741a6124e4e/i-tore-apart-the-claude-code-source-code-33-2jb1) highlighted it as an example of how Anthropic uses Claude Code internally in ways not available to the public.

The feature worked like this: after any model response involving tool calls, a `postSamplingHook` would fire a subagent to update markdown files bearing `# MAGIC DOC:` headers. The subagent had access to the Edit tool only — it could modify the doc it was given but nothing else — and its instructions emphasized a specific philosophy: terse, high-signal, current-state-only documentation. No changelogs, no exhaustive API lists, no code walkthroughs. Architecture, entry points, gotchas, and design rationale.

It was never available externally. The `initMagicDocs()` function was gated behind a hardcoded `getUserType() === "ant"` check that the bundler compiled to return `"external"` in every public build. The code was present in the npm package but completely inert — setting an environment variable wouldn't have helped. The gate was baked into the binary at compile time.

MagicDocs existed in the public eye for roughly 48 hours. The source maps leaked on March 31. The code was removed on April 2. I spent part of that window [engineering my own version](https://github.com/translunar/airbender) from the extracted prompts and the [Claw Code](https://github.com/instructkr/claw-code) reimplementation.

## Zero edits over 134 days of Claude Code prompt tracking

[Piebald AI](https://github.com/Piebald-AI/claude-code-system-prompts) has been tracking Claude Code's system prompts since November 19, 2025 — automatically extracting and diffing them with every release. Using their git history, we can see how often each prompt was modified over its lifetime.

Five prompts have existed since the repo began tracking. Of those five, MagicDocs was modified the fewest times:

| Prompt | Content modifications | Status |
|--------|:---:|--------|
| ReadFile tool description | 8 | active |
| Write tool description | 6 | active |
| Edit tool description | 4 | active |
| /pr-comments slash command | 4 | removed in v2.1.91 |
| MagicDocs agent prompt | 0 | removed in v2.1.91 |

These counts exclude repo maintenance commits (initial setup, metadata batch updates) and count only version-tagged Claude Code releases that modified the prompt content. MagicDocs was never modified by Anthropic — not once in the 134 days Piebald AI was tracking Claude Code releases.

<figure>
<img src="/assets/images/prompt-history.png" alt="Scatter plot of Claude Code system prompt modification frequency vs lifespan, counting only content changes in version-tagged releases. MagicDocs is highlighted as a pink diamond at zero modifications over 134 days. Most prompts cluster between 1-8 content modifications." />
<figcaption>MagicDocs received zero content modifications in the 134 days the prompt extraction repository tracked Claude Code, making it the only prompt Anthropic never iterated on in that time.</figcaption>
</figure>

The two prompts removed in v2.1.91 are the two least-iterated original prompts. That's probably not a coincidence.

## Why it probably failed

I want to be clear that this next section is somewhat speculative. We don't have internal communications from Anthropic explaining the removal, and the changelog is silent. But the evidence points in a clear direction.

Both removed prompts read like they were written before Claude Code had a skills framework. The `/pr-comments` prompt opens with "You are an AI assistant integrated into a git-based version control system. Your task is to fetch and display comments from a GitHub pull request." It then micromanages five numbered steps of `gh api` calls that Claude already knows how to make. In modern skills produced through test-driven design, unnecessary instructions are excised, but that isn't the case with older prompts. (This is the topic of a forthcoming article on context engineering I hope to publish this week.)

MagicDocs had a similar problem at a deeper level. It was a workflow baked into an agent prompt dispatched by a `postSamplingHook`. There was no way to test whether the prompt actually produced good documentation. No red/green/refactor cycle. No feedback loop at all. In my own MVP testing of a MagicDocs implementation, I found that subagents would modify or even delete out-of-scope files — a problem the original prompt presumably also had, given its lack of iteration.

Meanwhile, Claude Code's skills framework has matured into a proper solution for both use cases. Skills arrive at the point of action, fresh in context and testable with subagent pressure scenarios. The memory system — including [dream consolidation](https://github.com/Piebald-AI/claude-code-system-prompts/blob/main/system-prompts/agent-prompt-dream-memory-consolidation.md) — handles persistent cross-session knowledge with built-in pruning and maintenance. The iterative, self-correcting nature of MagicDocs is strikingly similar to what memory and dreaming now provide, suggesting that the functionality MagicDocs aimed for has been absorbed by these newer mechanisms.

I considered whether Anthropic might have simply changed how internal features are bundled — moving MagicDocs behind a compile-time feature flag rather than removing it. But other internal-only prompts remain visible in the extracted prompts repo, and `strings` analysis of the v2.1.91 binary confirms that both the MagicDocs prompt and its code infrastructure are completely absent. This wasn't a gating change. The feature was excised.

So why did MagicDocs' removal coincide with the Claude Code leak? One reasonable explanation is that without an internal advocate, the feature was forgotten until people reported on it in the context of the leak.

Auto-maintained documentation is a solid idea that [many people are exploring](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). But baking it into an untestable agent prompt that nobody iterated on for 134 days is telling in a codebase that was rapidly developing better mechanisms for exactly this kind of work.

<aside><p>The <a href="https://github.com/translunar/airbender/tree/main/docs">context engineering manual</a> covers how Claude Code assembles its prompt, loads instructions, selects memories, and manages subagents — including the MagicDocs architecture described in this post. The <a href="https://github.com/translunar/airbender/tree/main/analysis">analysis scripts</a> for the data in this post and the open-source MagicDocs replacement are in the <a href="https://github.com/translunar/airbender">airbender repository</a>.</p></aside>
