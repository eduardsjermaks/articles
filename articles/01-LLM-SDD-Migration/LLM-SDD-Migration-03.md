# Switching to a Harder Migration Target

The first comparison used **eShopOnWeb**, which was useful for testing the workflow, but it is still a relatively moderate codebase.

That makes it good for a first pass, but not ideal for stressing the parts that break during real migrations: hidden coupling, mixed concerns, runtime assumptions, and behavior spread across many layers.

For the next step, I switched to **nopCommerce**.

At the same time, the model landscape also changed. New models became available, including **Fable**, so this round focuses on a new comparison instead of repeating the exact same lineup.

The goal stayed the same: measure how well different models can produce a reliable first-pass architectural orientation of an unfamiliar legacy system.

## Why nopCommerce

nopCommerce is a much better stress test for migration-oriented analysis than eShopOnWeb.

It is a larger, older, more feature-dense ASP.NET application with:

- public storefront flows
- a large admin area
- multiple database providers
- plugin-based extensions
- in-process scheduled tasks
- many integrations and configuration surfaces

This matters because migration work usually gets blocked in operational details such as startup flow, scheduling, plugin loading, order processing, configuration writes, and cross-cutting infrastructure.

In a codebase like this, a model has to do more than summarize folders. It has to follow execution paths and separate observed facts from plausible guesses.

---

## Experiment Setup

I used the same overall idea as before: ask each model to generate a project-orientation document for the repository, then compare the outputs with a fixed AI-as-judge rubric.

### Generated orientation documents

| Prompt | Result |
|--------|--------|
| [codex-readme-prompt.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/nopCommerce/01/codex-readme-prompt.md) | [Codex response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/nopCommerce/01/responses/codex-readme-prompt-response.md) |
| [make-readme-fable.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/nopCommerce/01/make-readme-fable.md) | [Fable response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/nopCommerce/01/responses/fable-readme-response.md) |
| [make-readme-opus.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/nopCommerce/01/make-readme-opus.md) | [Claude Code response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/nopCommerce/01/responses/opus-readme-response.md) |

These documents were then compared pairwise using the judge prompts under `ai-judge/nopCommerce`.

---

## Evaluation Rubric

I kept the same rubric as in the earlier comparison:

1. **Evidence Grounding**
2. **Structural Accuracy**
3. **Dependency Mapping**
4. **Critical Flow Identification**
5. **Migration Insight Quality**
6. **Epistemic Discipline**
7. **Signal-to-Noise Ratio**

Goal: compare **quality of codebase understanding** with less attention on style or verbosity.

---

## Reference table

| Run | Prompt file | Result file | Document A | Document B |
|-----|-------------|-------------|------------|------------|
| 1 | [1-ai-judge-fable-vs-codex.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/nopCommerce/requests/1-ai-judge-fable-vs-codex.md) | [1-a-judge-fable-vs-codex.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/nopCommerce/results/1-a-judge-fable-vs-codex.md) | Fable | Codex |
| 2 | [1-ai-judge-fable-vs-opus.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/nopCommerce/requests/1-ai-judge-fable-vs-opus.md) | [1-ai-judge-fable-vs-opus.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/nopCommerce/results/1-ai-judge-fable-vs-opus.md) | Fable | Claude Code (Opus 4.8) |

---

## Run 1  
Fable vs Codex

| Criterion | Fable | Codex |
|-----------|-------|-------|
| Evidence grounding | 5 | 5 |
| Structural accuracy | 5 | 4 |
| Dependency mapping | 5 | 4 |
| Critical flow | 5 | 4 |
| Migration insight | 5 | 4 |
| Epistemic discipline | 5 | 3 |
| Signal / noise | 5 | 3 |

Result: **Fable produced the safer document for migration.**

The main point here is that Codex did not fail because of weak grounding. According to the judge, it was still well-evidenced, but it stayed closer to extraction than architectural synthesis.

The judge explicitly described the difference like this:

- Fable acted as an architectural synthesizer
- Codex stayed closer to a search-result aggregator or static extractor

In practice, that meant Fable was better at turning code facts into migration-relevant constraints.

The most important examples were:

- recognizing that the order flow contains deployment-relevant concurrency assumptions
- identifying the scheduler’s self-HTTP loopback behavior as an architectural constraint
- distinguishing facts, inferences, and unknowns in a disciplined way

Codex still found many correct details, but the result was noisier and less decisive when moving from structure to interpretation.

---

## Run 2  
Fable vs Claude Code (Opus 4.8)

| Criterion | Fable | Claude Code (Opus 4.8) |
|-----------|-------|-------------|
| Evidence grounding | 5 | 5 |
| Structural accuracy | 5 | 5 |
| Dependency mapping | 4 | 5 |
| Critical flow | 5 | 3 |
| Migration insight | 5 | 5 |
| Epistemic discipline | 5 | 5 |
| Signal / noise | 5 | 5 |

Result: **Fable again produced the safer migration document.**

This comparison was closer.

Claude Code (Opus 4.8) performed strongly on broad dependency mapping and surfaced useful environment and package-level details. The judge specifically noted that it provided a wider infrastructural view, including exact dependency versions and configuration-level concerns.

Fable scored better in the category that matters most for migration safety: **critical flow identification**.

The judge’s reasoning was that Claude Code widened effectively and gave a good infrastructure overview. Fable followed the order-processing path further and exposed concrete operational risks.

For migration planning, that kind of detail is often more useful than a package inventory. Library versions help, but checkout behavior, process-local mutexes, and self-HTTP background tasks usually have more impact on migration risk.

---

## What Changed from the First Comparison

With eShopOnWeb, the main differences were often about how grounded and careful the documents were.

With nopCommerce, that baseline became less important because all three outputs were already reasonably grounded. The separation happened later, when the models had to interpret behavior in a much more complex system.

That suggests an important pattern:

- smaller or cleaner projects reward extraction and concise summarization
- larger monoliths reward deeper synthesis of runtime behavior and architectural side effects

Once the codebase becomes messy enough, the winning model is usually the one that can infer safe migration constraints from the observed code.

---

## Aggregated summary

| Run | A | B | Winner |
|-----|---|---|--------|
| 1 | Fable | Codex | Fable |
| 2 | Fable | Claude Code (Opus 4.8) | Fable |

---

## Key conclusions

Findings:

- **Fable was the strongest model in this nopCommerce round**
- Its advantage came mostly from better architectural synthesis and interpretation
- **Codex** remained grounded, but the document was noisier and weaker at converting evidence into migration guidance
- **Claude Code (Opus 4.8)** was strong on dependency and infrastructure scanning, but weaker on the deepest execution-path analysis
- Model rankings can change when the project changes; results from a moderate codebase do not automatically transfer to a more complex monolith

The broader takeaway is that project-orientation tasks should be judged by how useful they are for understanding real system behavior. Module lists and file citations still matter, but they are not enough on their own.

For migration work, the higher-value signal is whether the model can identify the behaviors that would actually break when the system is moved: process assumptions, state transitions, scheduling tricks, and control-flow hotspots.

That is where Fable stood out in this round.

The next question is the practical one: whether this stronger initial orientation is enough to support actual migration steps, or whether the advantage disappears once code transformations begin.
