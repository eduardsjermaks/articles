# Intro

Most migrations fail before they start — because nobody actually knows what the system does.

Specifications are missing or outdated. Tests are incomplete. Business rules are scattered across the codebase. Refactoring feels risky because the behavioral surface of the system is unclear.

Legacy systems rarely fail because of syntax or frameworks. They fail because their behavior is undocumented and poorly understood. This misunderstanding of behavior and requirements becomes even more important when development is done with agents.

With strong LLM agents, the distribution of engineering effort starts to shift.

Less time is spent on writing code, while more time moves to validation, design, and decision-making:

- coding ↓↓↓  
- debugging ↓  
- testing ↑  
- design ↑  
- specs ↑  
- review ↑↑  

The reason is simple — most real bugs are not syntax errors, but requirement errors.  
Agents can generate code, but they cannot guarantee domain correctness — meaning the system behaves according to real business rules and constraints, not just syntax, types, or passing tests.

They can also generate tests, yet those tests often reproduce the same wrong assumptions as the implementation.  
LLMs are strong at local reasoning, but much weaker at global system understanding.

Because of that, the hardest problems remain architectural rather than syntactic:

- service boundaries  
- data ownership  
- consistency models  
- failure handling  
- backward compatibility  
- performance tradeoffs  

These decisions require context, experience, and understanding of the whole system.

Even with modern agents, several types of changes are still difficult:

- large refactors  
- multi-service changes  
- long-term evolution

## Alternative: Using the Full Context Window

Putting the entire codebase into the LLM context may seem attractive, but it works poorly for non-trivial projects. For larger systems, **agentic workflows become necessary**.

And the downside is not only cost.

- **Signal dilution** — tests, migrations, DTOs, generated files, and CSS can drown out the real architecture, and the model may miss important relationships because attention is spread too thin.
- **Less room for reasoning** — large context leaves less space for the actual question and the response.
- **Noise bias** — snapshots, migrations, and duplicated patterns can skew the model’s understanding.
- **Slow iteration** — every follow-up requires resending a very large prompt.

Large context windows help, but they do not replace structured, incremental analysis.

This is where modern LLM tooling becomes interesting. In this series, I explore how it can assist in migrating existing systems — not by blindly rewriting code, but by helping engineers understand, analyze, and safely transform unfamiliar codebases.

The focus is on cross-stack migration, where a system must be moved to a different technology stack due to vendor strategy, platform constraints, or organizational decisions. In these situations, the hardest part is usually not the target framework, but the lack of reliable knowledge about the current system.

Tools such as VS Code Copilot, Codex, Claude Code, and similar agent-style assistants introduce a new way of working with legacy code. Instead of treating migration as a purely manual reverse-engineering effort, we can use LLMs as interactive tools to explore the codebase, reconstruct intent, and guide the transition step by step.

The question is:

> How can LLM tooling help us migrate systems more safely?


## Codebase Overview

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

---

## 1. Setting the Scene

For this series, we will use **eShopOnWeb** as the legacy system.

[https://github.com/dotnet-architecture/eShopOnWeb](https://github.com/dotnet-architecture/eShopOnWeb)

The repository is archived and no longer actively maintained, which makes it a stable candidate for experimentation. The intent is not to criticize the original design, but to use a realistic codebase to evaluate different migration approaches.

We will assume a migration to **Java (Spring Boot)** for the backend and **React** for the frontend.
This is a hypothetical cross-stack scenario used to simulate real-world situations such as vendor strategy changes, platform standardization, or team skill constraints.

During the series, different LLM-assisted workflows will be explored, including:

* VS Code Copilot (agent / auto mode)
* Codex
* Claude Code

The objective is not to prove that one tool is better, but to observe how different tools behave when working with an unfamiliar system.

---

## Migration Strategy

For the migration itself, we will assume an incremental approach similar to the Strangler pattern.

Instead of rewriting everything at once, parts of the system are replaced gradually while the original system continues to run. This is a common strategy in real projects where full rewrites are too risky.

---

## Working Assumption

LLMs do not remove the risks of migration.

They can misunderstand intent, invent abstractions, or overlook important details.
If used blindly, they can make a migration less safe instead of safer.

However, when used as analysis and exploration tools, they can help:

* navigate unknown codebases
* summarize structure and dependencies
* identify hidden assumptions
* experiment with refactoring options
* compare alternative implementations

The approach in this series is therefore:

1. Start with an unknown system.
2. Use LLM tools to explore and understand its structure and behavior.
3. Gradually transform the system toward a new stack.
4. Observe how different tools help or fail during this process.

The goal is not to automate migration.

The goal is to make migration more predictable when the system is not fully understood.



# Using AI-as-Judge to Compare LLM Codebase Analysis

When comparing outputs from different LLM tools, subjective reading is unreliable.  
To make the comparison reproducible, I used an **AI-as-judge approach with a fixed rubric**, where two generated documents were evaluated against the same criteria.

Each run compared two documents produced by different tools, and the judge selected which one produced **better codebase analysis**.

The judge model used in all runs was **Gemini 3 Pro**.

---

## Reference table

| Run | Result file | Document A | Document B | AI as Judge |
|------|------------|-----------|-----------|------------|
| Run 1 | 1-ai-judge-prompt | Codex with medium (5.4) | Claude Code | Gemini 3 Pro |
| Run 2 | 1-ai-judge-vs-code-auto-vs-codex | Codex with medium (5.4) | VSCode default auto mode (GPT-5.4 Opus) | Gemini 3 Pro |
| Run 3 | 1-ai-judge-gemini-prompt-extra | Codex with medium (5.4) | Codex with extra thinking (5.4) | Gemini 3 Pro |

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

Goal: compare **quality of codebase understanding**, not writing style.

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

AI-as-judge worked well, but results depended strongly on generation mode.

Findings:

- Medium thinking produced very reliable grounded analysis  
- Extra thinking improved architectural reasoning  
- Default agent mode produced more assumptions  
- Judge must use a fixed rubric, otherwise results change  

In practice, the most stable workflow was:

1. Generate analysis with multiple tools  
2. Compare using a fixed rubric  
3. Use AI judge (Gemini 3 Pro)  
4. Inspect differences manually  

This approach made codebase exploration much more predictable when working with unknown projects.