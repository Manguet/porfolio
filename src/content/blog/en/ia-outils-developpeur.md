---
title: "AI and Development: The Tools That Actually Change My Daily Work"
description: "An honest look at a year of using AI tools as a freelance developer: what works, what disappoints, and how to integrate them without compromising quality."
pubDate: 2025-12-05
tags: ["AI", "Productivity", "Development"]
lang: en
slug: ia-outils-developpeur
---

## Introduction: The Promise and the Reality

Since the explosion of consumer LLMs in late 2022, developers have been living under a flood of promises: AI will code for you, junior developers will disappear, seniors will be 10x more productive. A year and a half later, the reality is more nuanced — and more interesting.

I have gradually integrated several AI tools into my freelance PHP/Symfony development workflow. Here is an honest account of what has concretely changed, what still disappoints, and how I organise my work with these tools today.

## The Tools I Tested

### Claude (Anthropic)

My primary tool for in-depth questions, code review, documentation writing, and architecture problems. Claude excels at long, structured reasoning. Its ability to maintain coherent context across a long conversation makes it an excellent partner for exploring complex topics.

### GitHub Copilot

Integrated directly into VS Code and PhpStorm, Copilot completes code in real time. It is the most transparent tool in use: you do not change your workflow, you simply accept or reject suggestions. It is particularly effective on repetitive code and well-established patterns.

### Cursor

A code editor built around LLMs. The most useful feature: "Composer" mode, which can modify multiple files simultaneously based on a natural language instruction. Useful for cross-cutting refactoring.

### ChatGPT (OpenAI)

Less used day-to-day since adopting Claude, but still useful for quick lookups, concept explanations, or comparing solutions.

## What Actually Works

### Boilerplate Generation

This is arguably the most immediate and measurable gain. Generating a Doctrine entity with its getters/setters, a Symfony FormType, a database migration, an empty unit test class — all these repetitive tasks that require no real thought but consume time.

Before: 15–20 minutes to scaffold a complete CRUD feature. With AI: 3–5 minutes of review and adjustment.

### Writing Tests

Giving a class to Claude with the instruction "generate PHPUnit unit tests for this class" generally produces a correct and comprehensive test base. I check, adjust edge cases, but the structure is there. Productivity on test coverage has increased significantly.

### Documentation and Comments

Documenting an API, writing PHPDoc docblocks, generating a README — tasks I used to put off. AI makes them almost instant.

### Code Review and Anomaly Detection

Pasting a block of code into Claude with "what could be problematic here?" regularly reveals subtle bugs or security issues I had not noticed. It has become a systematic step before pushing code.

## What Still Disappoints

### Complex Architecture and Business Context

Asking an AI to design the architecture of a complex business application produces generic results. It does not know the client's specific constraints, the design decisions made six months ago, the reasons why one pattern was chosen over another.

AI is good at implementing a well-defined solution. It is poor at defining which solution to implement.

### Legacy Code and Implicit Context

On an existing project with its own conventions, historical hacks, and specific dependencies, AI often generates code that does not integrate well. It works in a vacuum without knowledge of the broader context.

### Blind Trust in Suggestions

This is the main risk: accepting suggestions without understanding them. I have seen less experienced developers integrate AI-generated code they did not understand, creating hard-to-debug bugs and security issues. AI amplifies existing skills; it does not replace them.

## My Current Workflow

1. **Design and architecture**: alone, on paper or with a colleague. AI does not intervene here.
2. **Scaffolding and boilerplate**: Copilot in real time, Claude for more complex structures.
3. **Implementation**: Copilot for in-context suggestions, Claude to unblock problems.
4. **Tests**: Claude generates the base, I complete the edge cases.
5. **Pre-commit review**: Claude for a final pass on security and quality.
6. **Documentation**: Claude for drafting, I review and adjust the tone.

## Risks to Take Seriously

**Dependency and skill atrophy**: by having code generated constantly, some developers lose fluency on basic tasks. I regularly impose sessions without AI assistance to maintain my skills.

**Intellectual property**: the status of AI-generated code remains legally uncertain depending on the country. For clients with strict ownership requirements, this is a point to clarify contractually.

**Quality and hallucinations**: AI can generate code that looks correct but is subtly wrong, or cite non-existent APIs. Human review remains indispensable.

**Confidentiality**: prompts sent to cloud services sometimes contain sensitive business code. I use enterprise modes when available, and remain vigilant about what I share.

## Conclusion: Tool, Not Replacement

A year after seriously integrating these tools, the productivity verdict is positive — with important nuances. AI has made me more efficient on repetitive tasks and freed up time for problems that genuinely require thought.

But it has not changed what makes an experienced developer valuable: understanding the client's business, making durable architecture decisions, identifying the real problems behind the symptoms. For that, humans are still needed.
