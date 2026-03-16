# Intro

Most migrations fail before they start — because nobody actually knows what the system does.

Specifications are missing or outdated. Tests are incomplete. Business rules are scattered across the codebase. Refactoring feels risky because the behavioral surface of the system is unclear.

Legacy systems rarely fail because of syntax or frameworks. They fail because their behavior is undocumented and poorly understood.

In this series, I explore how modern LLM tooling can assist in migrating existing systems — not by blindly rewriting code, but by helping engineers understand, analyze, and safely transform unfamiliar codebases.

The focus is on cross-stack migration, where a system must be moved to a different technology stack due to vendor strategy, platform constraints, or organizational decisions. In these situations, the hardest part is usually not the target framework, but the lack of reliable knowledge about the current system.

Tools such as VS Code Copilot, Codex, Claude Code, and similar agent-style assistants introduce a new way of working with legacy code. Instead of treating migration as a purely manual reverse-engineering effort, we can use LLMs as interactive tools to explore the codebase, reconstruct intent, and guide the transition step by step.

The goal of this series is not automatic replatforming, and not one-click migration.
The goal is controlled migration driven by understanding.

The central question is:

> Can LLM tooling help us migrate systems more safely when the original design is only partially known?

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