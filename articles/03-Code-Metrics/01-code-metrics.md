# Is Code Generation Solved, or Has Verification Become the Bottleneck?

As discussion around code generation accelerates, I keep returning to a harder question: do we have the metrics and indicators needed to trust generated code without reviewing it line by line?

Generating code is becoming faster and cheaper. That shifts the engineering constraint from implementation to verification. But can we safely deploy systems, especially critical ones, based on a small set of indicators? And what would those indicators need to prove?

## Test Coverage Is Necessary, but Not Sufficient

- **Line and branch coverage** show that code was exercised. They do not show that the tests assert the right behavior or reflect real business expectations.
- **Mutation testing** helps reveal weak tests and missing edge cases. Is it enough, or do we need additional ways to measure the quality of our test suites?
- **Negative testing** matters too. How do we verify that a system does *not* perform an unintended action, such as sending credentials or other sensitive data over the network?

We need indicators that measure more than execution. They need to give us confidence in correctness, safety, and the completeness of the intended behavior.

## What About Change?

Another question is how to measure a system's ability to evolve. When requirements change, should we be able to make a focused change, or are we forced to regenerate large parts of the solution?

Regeneration may be inexpensive, but it still carries risk: regressions, missing tests, and behavior that quietly drifts from the original intent. The engineering mechanics do not disappear just because generation is cheap. Generation still needs to remain under control.

## Aligning Agents With Product Intent

There is also a deeper issue around tests and assertions: how do we communicate our expectations to coding agents?

When building a product, we rarely know every edge case in advance. If a situation has not been considered by the team, can we assume an agent will discover the right solution? More importantly, can we be confident that its solution aligns with the business's understanding of the problem?

For less sensitive systems, progressive delivery, pilot deployments, observability, and rapid rollback can reduce the risk. But those safeguards are not available everywhere, particularly in critical domains where failure is unacceptable.

I expect this field to evolve quickly. The interesting challenge is no longer only how to generate code, but how to establish enough evidence to trust it.



