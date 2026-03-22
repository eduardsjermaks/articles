# Intro

Most migrations fail before they start — because nobody actually knows what the system does.

Specifications are missing or outdated. Tests are incomplete. Business rules are scattered across the codebase. Refactoring feels risky because the behavioral surface of the system is unclear.

Legacy systems rarely fail because of syntax or frameworks. They fail because their behavior is undocumented and poorly understood. This lack of understanding becomes even more critical when development is done with agents.

In this series, I explore how LLM tooling can assist in migrating existing systems — not by blindly rewriting code, but by helping engineers understand, analyze, and safely transform unfamiliar codebases.

The focus is on cross-stack migration, where a system must be moved to a different technology stack due to vendor strategy, platform constraints, or organizational decisions. In these situations, the hardest part is usually not the target framework, but the lack of reliable knowledge about the current system.

Tools such as VS Code Copilot, Codex, Claude Code, and similar agent-style assistants introduce a new way of working with legacy code. Instead of treating migration as a purely manual reverse-engineering effort, we can use LLMs as interactive tools to explore the codebase, reconstruct intent, and guide the transition step by step.


## Shift in Engineering Work with LLM Agents

With strong LLM agents, the distribution of engineering effort starts to shift.  
Less time is spent writing code, while more time moves to validation, specs, design, and review.

Agents can generate code, but they cannot guarantee domain correctness — the system may pass tests while still violating business rules or real-world constraints.  
Most real bugs are not syntax errors, but misunderstood requirements, missing system context, and edge cases.

Even with modern agents, some types of changes remain difficult: large refactors, multi-service changes, and long-term evolution.

## Alternative: Using the Full Context Window

Putting the entire codebase into the LLM context may seem attractive, but it works poorly for non-trivial projects. For larger systems, agentic workflows become necessary.

The downside is not only cost.

- **Signal dilution** — tests, migrations, DTOs, generated files, and CSS can drown out the real architecture, and the model may miss important relationships because attention is spread too thin.
- **Less room for reasoning** — large context leaves less space for the actual question and the response.
- **Noise bias** — snapshots, migrations, and duplicated patterns can skew the model’s understanding.
- **Slow iteration** — every follow-up requires resending a very large prompt.


## 1. Setting the Scene

For this series, we will use **eShopOnWeb** as the legacy system.

https://github.com/dotnet-architecture/eShopOnWeb

### Codebase Overview

- ~5.3k lines of production C# in `src`
- ~12.5k total lines across C#, Razor, CSS, SCSS, and Bicep
- 10 projects total: 6 production, 4 test
- ~194 production classes and 16 interfaces
- ~52 test cases
- 8 public API endpoints
- 4 MVC controllers
- 16 Razor Page models

Overall complexity is **moderate**.  
The codebase is not large in raw size, but the architectural scope is non-trivial: it includes a web app, public API, Blazor admin UI, separated core/infrastructure layers, infrastructure-as-code, and multiple test projects.

The repository is archived and no longer actively maintained, which makes it a good candidate for experimentation. The intent is not to criticize the original design, but to use a realistic codebase to evaluate different migration approaches.

We will assume a migration to **Java (Spring Boot)** for the backend and **React** for the frontend.  
This is a hypothetical cross-stack scenario used to simulate real-world situations such as vendor strategy changes, platform standardization, or team skill constraints.

During the series, different LLM-assisted workflows will be explored, including:

* VS Code Copilot (agent / auto mode)
* Codex
* Claude Code

---

## Working Assumption

LLMs can misunderstand intent, invent abstractions, or overlook important details.  
If used blindly, they can make a migration less safe instead of safer.

However, when used as analysis and exploration tools, they can help navigating unknown codebases:

* summarize structure and dependencies
* identify hidden assumptions
* experiment with refactoring options

The approach in this series is therefore:

1. Start with an unknown system.
2. Use LLM tools to explore and understand its structure and behavior.
3. Gradually transform the system toward a new stack.

# Using AI-as-Judge to Compare LLM Codebase Analysis

When comparing outputs from different LLM tools, subjective reading is unreliable.  
To make the comparison reproducible, I used an **AI-as-judge approach with a fixed rubric**, where two generated documents were evaluated against the same criteria.

Each run compared two documents produced by different tools, and the judge selected which one produced **better codebase analysis**.

The judge model used in all runs was **Gemini 3 Pro**.

---

## Reference table

| Run | Prompt file | Result file | Document A | Document B |
|------|------------|------------|-----------|-----------|
| Run 1 | [1-ai-judge-prompt](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-prompt.md) | [1-ai-judge-prompt-result](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/results/1-ai-judge-result.md) | Codex with medium (5.4) | Claude Code |
| Run 2 | [1-ai-judge-vs-code-auto-vs-codex](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-vs-code-auto-vs-codex.md) | [1-ai-judge-vs-code-auto-vs-codex-result](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/results/1-ai-judge-vs-code-auto-vs-codex-result.md) | Codex with medium (5.4) | VSCode default auto mode (GPT-5.4, Opus 4.6, Sonnet 4.6) |
| Run 3 | [1-ai-judge-gemini-prompt-extra](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-gemini-prompt-extra.md) | [1-ai-judge-gemini-prompt-extra-result](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-gemini-prompt-extra.md) | Codex with medium (5.4) | Codex with extra thinking (5.4) |

---

## Judge rubric

Same rubric used in all runs:

| Criterion |
|-----------|
| Evidence grounding |
| Structural accuracy |
| Dependency mapping |
| Critical flow identification |
| Migration insight quality |
| Epistemic discipline |
| Signal-to-noise ratio |
| Final verdict |

Goal: compare **quality of codebase understanding**

---

## Run 1  
Codex (medium) vs Claude Code

| Criterion | Codex medium | Claude Code | Winner |
|-----------|-------------|-------------|--------|
| Evidence grounding | 5 | 3 | A |
| Structural accuracy | 5 | 3 | A |
| Dependency mapping | 4 | 5 | B |
| Critical flow | 5 | 4 | A |
| Migration insight | 5 | 4 | A |
| Epistemic discipline | 5 | 4 | A |
| Signal / noise | 5 | 3 | A |
| Final verdict | — | — | A |

Result: Codex medium produced more precise and grounded codebase analysis.

---

## Run 2  
Codex (medium) vs VSCode Auto (GPT-5.4 Opus)

| Criterion | Codex medium | VSCode Auto | Winner |
|-----------|-------------|-------------|--------|
| Evidence grounding | 5 | 3 | A |
| Structural accuracy | 5 | 4 | A |
| Dependency mapping | 5 | 4 | A |
| Critical flow | 5 | 3 | A |
| Migration insight | 5 | 2 | A |
| Epistemic discipline | 5 | 2 | A |
| Signal / noise | 5 | 3 | A |
| Final verdict | — | — | A |

Result: Codex medium produced more grounded analysis with fewer assumptions.

---

## Run 3  
Codex (medium) vs Codex (extra thinking)

| Criterion | Medium | Extra thinking | Winner |
|-----------|---------|----------------|--------|
| Evidence grounding | 5 | 5 | A/B |
| Structural accuracy | 5 | 5 | A/B |
| Dependency mapping | 4 | 5 | B |
| Critical flow | 5 | 5 | A/B |
| Migration insight | 4 | 5 | B |
| Epistemic discipline | 5 | 5 | A/B |
| Signal / noise | 3 | 5 | B |
| Final verdict | — | — | B |

Result: Extra thinking produced deeper architectural analysis,  
but improvements appeared only in some rubric categories.

---

## Important observation — extra thinking helped only in some areas

Extra thinking improved mainly:

- dependency mapping  
- migration insight  
- architectural context  
- signal-to-noise quality  

No significant improvement in:

- evidence grounding  
- structural accuracy  
- flow tracing  
- epistemic discipline  

This means extra reasoning mainly helps with **interpretation**, not with raw extraction.

---

## Runtime comparison

| Mode | Time |
|------|--------|
| Codex medium | ~14 min |
| Codex extra thinking | ~30 min |

Extra thinking took more than 2× longer,  
while improvement was limited to some rubric categories.

This is important in real workflows where analysis must be repeated many times.

---

## Aggregated summary

| Run | A | B | Winner |
|-----|----|----|--------|
| Run 1 | Codex medium | Claude Code | A |
| Run 2 | Codex medium | VSCode Auto | A |
| Run 3 | Codex medium | Codex extra thinking | B |

---

## Key conclusions

Findings:

- Codex medium thinking produced reliable, well-grounded analysis  
- Codex extra thinking improved architectural reasoning  
- Claude Code achieved similar results, with minor misinterpretations, but produced clear diagrams that made the structure easier to understand  
- VSCode agent mode introduced more assumptions  

This approach provided a quick initial overview of the project and highlighted critical areas in a short time.  
The next step is to see whether this level of understanding is sufficient for the migration itself, or whether important parts are still missing.