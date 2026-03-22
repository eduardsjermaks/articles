# Intro

Most migrations fail before they start — because nobody actually knows what the system does.

Legacy systems rarely fail because of syntax or frameworks. They fail because their behavior is undocumented and poorly understood. This lack of understanding becomes even more critical when development is done with agents.

In this series, I explore how LLM tooling can assist in migrating existing systems. The focus is on cross-stack migration, where a system must be moved to a different technology stack due to platform, vendor, or organizational constraints. In these cases, the hardest part is usually incomplete knowledge of the current system.

Tools such as Copilot, Codex, Claude Code, and similar agents make it possible to explore a codebase interactively, summarize its structure, and trace important flows instead of relying only on manual reverse engineering.

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


## Setting the Scene

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

## Experiment Setup

To compare how different models analyze the same codebase, I used a fixed set of prompts and collected the responses produced by each model.

Prompts were refined with ChatGPT to make them clear and neutral, so that differences in results come from the models, not from prompt wording.

The table below shows the prompts used in the experiment and the corresponding results.

| Prompt | Result |
|--------|--------|
| [1-codex-readme-prompt.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/1-codex-readme-prompt.md) | [Codex (medium thinking) response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/responses/1-codex-readme-prompt-response.md) |
| [1-codex-readme-prompt.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/1-codex-readme-prompt.md) | [Codex (extra thinking) response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/responses/1-codex-readme-prompt-response-extra.md) |
| [1-make-readme-opus.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/1-make-readme-opus.md) | [Claude Code response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/responses/1-opus-readme-response.md) |
| [1-vscode-readme-prompt.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/1-vscode-readme-prompt.md) | [VSCode response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/responses/1-vscode-readme-response.md) |


These responses are evaluated in the next section using an AI-as-Judge approach.

# Using AI-as-Judge to Compare LLM Codebase Analysis

When comparing outputs from different LLM tools, subjective reading is unreliable.  
To make the comparison reproducible, I used an **AI-as-judge approach with a fixed rubric**, where two generated documents were evaluated against the same criteria.

Each run compared two documents produced by different tools, and the judge selected which one produced **better codebase analysis**.

The judge model used in all runs was **Gemini 3 Pro**.

---

## Reference table

| Run | Prompt file | Result file | Document A | Document B |
|------|------------|------------|-----------|-----------|
| 1 | [1-ai-judge-prompt](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-prompt.md) | [1-ai-judge-prompt-result](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/results/1-ai-judge-result.md) | Codex medium thinking (GPT-5.4)  | Claude Code (Opus 4.6) |
| 2 | [1-ai-judge-vs-code-auto-vs-codex](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-vs-code-auto-vs-codex.md) | [1-ai-judge-vs-code-auto-vs-codex-result](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/results/1-ai-judge-vs-code-auto-vs-codex-result.md) | Codex medium thinking (GPT-5.4)  | VSCode default auto mode (GPT-5.4, Opus 4.6, Sonnet 4.6) |
| 3 | [1-ai-judge-gemini-prompt-extra](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-gemini-prompt-extra.md) | [1-ai-judge-gemini-prompt-extra-result](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-gemini-prompt-extra.md) | Codex medium thinking (GPT-5.4)  | Codex extra thinking (GPT-5.4) |

---

## Evaluation Rubric (Score 0–5 per category)
1. **Evidence Grounding:** Does the document cite specific modules, files, or patterns rather than vague generalities?
2. **Structural Accuracy:** Is the internal logic consistent? Do the described components actually fit together logically?
3. **Dependency Mapping:** How well does it identify external integrations, internal coupling, and third-party libraries?
4. **Critical Flow Identification:** Does it map the "Happy Path" of data through the system (Entry point -> Logic -> Storage)?
5. **Migration Insight Quality:** Does it identify technical debt, "gotchas," legacy patterns, or risks that would impact a migration?
6. **Epistemic Discipline:** How does it handle uncertainty? Does it clearly distinguish between known facts ("The system does X") and assumptions ("The system appears to do X")?
7. **Signal-to-Noise Ratio:** Is the document concise and information-dense, or is it filled with filler?

Goal: compare **quality of codebase understanding**

---

## Run 1  
Codex medium thinking (GPT-5.4) vs Claude Code (Opus 4.6) 

| Criterion | Codex | Claude Code |
|-----------|-------------|-------------|
| Evidence grounding | 5 | 3 |
| Structural accuracy | 5 | 3 |
| Dependency mapping | 4 | 5 |
| Critical flow | 5 | 4 |
| Migration insight | 5 | 4 |
| Epistemic discipline | 5 | 4 |
| Signal / noise | 5 | 3 |

Result: Codex medium produced more precise and grounded codebase analysis.


Claude Code produced very good tables and correctly detected the `ApplicationCore → BlazorShared` dependency, calling it a domain-layer contamination hotspot, which is a meaningful architectural finding.

However, it lost points in structural accuracy due to inferred runtime details.  
Example:
> "Web — Single ASP.NET Core process hosting: MVC controllers, Razor Pages, Blazor Server circuit, and serving the BlazorAdmin WASM bundle."

This mixes Blazor Server runtime with static WASM hosting and was not fully grounded in the code.

---

## Run 2  
Codex medium thinking (GPT-5.4) vs VSCode default auto mode (GPT-5.4, Opus 4.6, Sonnet 4.6)

| Criterion | Codex | VSCode |
|-----------|-------------|-------------|
| Evidence grounding | 5 | 3 |
| Structural accuracy | 5 | 4 |
| Dependency mapping | 5 | 4 |
| Critical flow | 5 | 3 |
| Migration insight | 5 | 2 |
| Epistemic discipline | 5 | 2 |
| Signal / noise | 5 | 3 |

Result: Codex medium produced more grounded analysis with fewer assumptions.


VSCode default auto mode often stopped at the first layer instead of tracing real execution paths.

Example:

> "Failure modes: Unknown (no explicit exception handling in this page model)..."

Failure modes were left unresolved instead of following the call chain.

Migration insight also contained generic or speculative statements without code evidence.

Example:

> "Cyclic dependencies — why risky: Unknown. evidence: Unknown."

---

## Run 3  
Codex medium thinking (GPT-5.4) vs Codex extra thinking (GPT-5.4) 

| Criterion | Medium | Extra thinking |
|-----------|---------|----------------|
| Evidence grounding | 5 | 5 |
| Structural accuracy | 5 | 5 |
| Dependency mapping | 4 | 5 |
| Critical flow | 5 | 5 |
| Migration insight | 4 | 5 |
| Epistemic discipline | 5 | 5 |
| Signal / noise | 3 | 5 |

Result: Extra thinking produced deeper architectural analysis,  
but improvements appeared only in some rubric categories.

Extra thinking spent more effort analyzing failure modes and runtime behavior.

It identified additional risks in caching, checkout flow, and environment-specific startup configuration.

Notably, extra thinking inspected infrastructure setup and found that in non-development startup the app loads Key Vault and configures SQL Server with retry-on-failure, including retry settings defined in the infrastructure layer.  
It also detected an issue in retry configuration where the retry logic may not be consistently applied across environments.

Extra thinking also showed stronger architectural reasoning.  
For example, it pointed out a layering anomaly that medium thinking missed:

> *Justification:* It maps the standard internal dependencies well but fails to spot the architectural anomaly where the inner domain (`ApplicationCore`) references a UI-adjacent library (`BlazorShared`).

Possible explanation is that extra reasoning mainly helps with **interpretation** instead of raw extraction.

---

## Runtime comparison

| Mode | Time |
|------|--------|
| Codex medium | ~14 min |
| Codex extra thinking | ~30 min |
| VSCode | ~5 min |

Extra thinking took more than 2× longer than medium thinking, while improvement was limited to some rubric categories.

This is important in real workflows where analysis must be repeated many times.

---

## Aggregated summary

| Run | A | B | Winner |
|-----|----|----|--------|
| 1 | Codex medium thinking (GPT-5.4) | Claude Code | Codex medium thinking (GPT-5.4) |
| 2 | Codex medium thinking (GPT-5.4) | VSCode Auto | Codex medium thinking (GPT-5.4) |
| 3 | Codex medium thinking (GPT-5.4) | Codex extra thinking (GPT-5.4)  | Codex extra thinking (GPT-5.4) |

---

## Key conclusions

Findings:

- Codex medium thinking produced reliable, well-grounded analysis  
- Codex extra thinking improved architectural reasoning  
- Claude Code achieved similar results, with minor misinterpretations, but produced clear diagrams that made the structure easier to understand  
- VSCode agent mode introduced more assumptions  

This approach provided a quick initial overview of the project and highlighted critical areas in a short time.  
The next step is to see whether this level of understanding is sufficient for the migration itself, or whether important parts are still missing.