# Intro

Most migrations fail before they start — because nobody actually knows what the system does.

Specifications are missing or outdated. Tests are minimal. Business rules are scattered across the codebase. Refactoring feels dangerous because the behavioral surface is unknown.

Legacy systems do not fail because of syntax or frameworks. They fail because their behavior is undocumented and poorly understood.

In this series, I focus specifically on cross-stack migration — transitioning a system to a different technology stack due to vendor strategy or platform constraints. If LLM-assisted spec extraction proves viable, it introduces an interesting capability: organizations could become more fluid in their technology choices, less tightly coupled to specific vendors, stacks, or talent pools.

The goal is not automated code generation, but behavioral clarity through spec-driven development.

Can LLMs make migration safer?

## 1. Setting the Scene

For this series, we will use [**eShopOnWeb**](https://github.com/dotnet-architecture/eShopOnWeb) as our legacy system.

Disclaimer: the repository is archived and no longer actively maintained. That makes it a stable candidate for experimentation. The intent is not to criticize the original authors or design decisions, but to use a realistic codebase to explore migration strategies.

We will assume a migration to **Java (Spring Boot)** for the backend and **React** for the frontend — a hypothetical cross-stack migration scenario used for analytical purposes, for example due to vendor or platform constraints.

To support a spec-driven workflow, we will experiment with [**spec-kit**](https://github.com/github/spec-kit) in VS Code.


## Migration Strategy

For the migration strategy, we will use the Strangler pattern — a commonly applied approach in system transitions.

It enables gradual replacement of components while the legacy system continues to operate. This makes it more feasible in real-world environments where full rewrites are costly and operational risk must be controlled.

There are alternative strategies, including complete rewrites. However, comparing migration models is not the goal of this article. The focus here is different: how to establish behavioral clarity before and during migration.

The Strangler pattern will serve as the execution strategy. The central question is how LLM-assisted specification extraction can reduce risk within that process.

Even with a Strangler-style migration, significant risks remain.

Boundaries may be drawn incorrectly. Implicit contracts between modules can be misunderstood. Edge cases embedded in legacy conditionals may be lost. Data semantics can subtly shift during reimplementation. Over time, these small inconsistencies accumulate into production defects.

All of these risks share a common root cause: incomplete understanding of system behavior.

## Working Assumption

The primary risk in migration is the lack of explicit behavioral understanding.

LLMs are not immune to this. They can misinterpret intent, hallucinate abstractions, and overlook edge cases. Used carelessly, they may amplify the very uncertainty we aim to reduce.

However, if these limitations are acknowledged, we can design a workflow that constrains and validates their output.

The approach in this series is therefore:

1. Extract and formalize the system’s behavior.
2. Apply an incremental (Strangler-style) migration anchored to that specification.

The goal is not code generation, but behavioral clarity.
