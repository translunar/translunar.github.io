---
layout: page
title: AI & Agents
nav_title: Agentic
permalink: /agentic/
description: "AI agent research, context engineering, and applied ML — from aerospace state estimation to LLM-based tooling systems."
---

I research how AI agents work in high-stakes engineering environments — where the code has to be right, the review burden is real, and the tools are changing faster than the teams using them.

My background is in state estimation and computational systems biology. I bring the same mathematical rigor to understanding AI systems: how they manage context, why they fail, and how to make them reliable enough to trust with real work (such as space missions, where failure is especially expensive).

## Current Projects

### Airbender — Context Engineering for Claude Code

<img alt="Decision tree showing where to put different types of instructions in Claude Code" src="/assets/images/airbender-decision-tree.png" style="width: 60%" align="right" />

A five-chapter manual documenting how Claude Code manages context internally, plus a plugin that replicates Anthropic's internal MagicDocs system using public primitives. Every architectural claim cites source files from publicly available repositories. The MagicDocs implementation was validated with TDD against a real codebase.

[GitHub](https://github.com/translunar/airbender)

### Waterbender — Data-Driven Content Strategy

Applied ML research: 1,249 social media posts scored by LLM-as-judge on 8 engagement dimensions, analyzed with Mann-Whitney U tests and Bonferroni-corrected significance testing. Identified 6 statistically significant predictors of breakout engagement. Built a complete analytical pipeline and strategic framework from the findings.

### f64 — Democratization of Healthcare Advocacy

A Claude Code plugin that guides patients and providers through gender-affirming care prior authorization and insurance appeals. Five conversational skills cover the full workflow — coverage verification, clinical documentation review, PA submission, denial appeals, and out-of-network authorization. The Python backend handles PDF generation, case management, and fax integration, encoding expertise in California insurance law (DMHC, Health & Safety Code §1365.5) and WPATH Standards of Care Version 8.

### Language Learning with LLMs

Using large language models to generate frequency dictionaries, then applying embedding cosine similarity and other measures to predict which vocabulary items will be difficult to memorize. Builds the results into Anki decks and revises existing decks when learners struggle with specific cards.

### Pro-Democracy Hackathons

Collaborating on hackathons that teach political organizers and civic institutions to build their own AI-powered solutions. Making sure democratic institutions have access to these tools — not just tech companies.

### Agent Adoption at Scale

At my day job in aerospace, I architect the systems that make AI agents usable in an environment where we can't afford for things to break. This means solving the review burden problem (agents produce more code than humans can review), preventing burnout, and building expertise systems so engineers can use the tools effectively.

## Background

My PhD is in computational systems biology — predicting gene-phenotype associations using network analysis methods. My postdoc and career have been in spacecraft guidance, navigation, and control, and more fundamentally in **state estimation**: probabilistic reasoning about uncertain systems from noisy measurements.

These fields are more connected than they sound. Computational biology, state estimation, and AI agent behavior are all problems of inference under uncertainty. The mathematical frameworks transfer.
