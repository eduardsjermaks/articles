# Is Code Generation Solved, or Has Verification Become the Bottleneck?

As discussion around code generation accelerates, I keep returning to a harder question: what evidence would let us trust generated code without code review?

Generating code itself is becoming faster and cheaper. Now more and more talks that verification is a bottleneck. But can it be that we can base decisions on a small set of universal indicators whether the system is good enough for production use? 

Different systems carry different risks, and functional correctness, security, privacy, resilience, maintainability, and regulatory compliance are different claims that require different evidence.

The more useful question is whether we can assemble enough evidence for a particular change and its risk: enough to reduce the right kind of human review, and enough to justify deployment.

## Test Coverage Is Necessary, but Not Sufficient


Dijkstra's old point applies: testing shows the presence of bugs, not their absence. You cannot enumerate your way through bad behaviors.

- **Line and branch coverage** show that code was executed. They do not show that tests assert the right behavior, represent business expectations, or cover the important combinations of conditions.
- **Mutation testing** can reveal tests that do not distinguish selected, seeded defects. It is a useful signal of test-suite sensitivity, but it cannot establish that all business rules or real-world edge cases have been considered.
- **Absence of behavior** is where testing reaches its limit. Tests show what a
  system does, not what it *never* does — that credentials are never sent over
  the network, for instance. Such claims are enforced structurally: least
  privilege, restricted egress, sandboxing, and analysis over all paths rather
  than sampled executions.
- **Property-based testing and fuzzing** deserve a place here too: properties and invariants encode intent more durably than example-based tests, and may be the closest thing we have to machine-checkable intent. 
- **Types, contracts, and static analysis** provide another widely deployed form of verification. Boundary contracts, including OpenAPI schemas, consumer-driven contracts, and Kafka schema-compatibility gates, localize trust between components.

We need evidence that reaches beyond execution. Requirements, acceptance criteria, security controls, and hazards should be traceable to tests or to another justified verification method. 

Who produces the evidence matters. If the same agent writes the implementation and every test, a green suite may only confirm the agent's interpretation of an ambiguous requirement. Human-owned acceptance tests, independently authored checks, and held-out evaluators provide a more stable reference point. This is why verification independence is central to safety-critical practice.

Metrics are signals, not proof. High coverage, a strong mutation score, or a clean static-analysis report can increase confidence, but none proves correctness, safety, or completeness on its own. The evidence must be interpreted against explicit assumptions and acceptable residual risk.

Metrics also become fragile when an agent is optimized directly against them: assertions can be weakened, inputs special-cased, and inconvenient checks removed. Gates need ownership, adversarial robustness, and some evidence held outside the generating loop.

## Evidence Beyond Tests

For many systems, the evidence should also include:

- **Security and privacy evidence**, such as threat models, authorization tests, dependency and provenance checks, secret scanning, static analysis, fuzzing, and controlled tests of data egress.
- **Operational resilience evidence**, such as load and stress tests, fault injection, recovery tests, service-level objectives, and verified rollback procedures.
- **Behavioral comparison**, where possible: differential testing against a trusted implementation, compatibility tests for public contracts, or regression tests based on previous defects.
- **Requirements coverage**, including the ability to identify important requirements or hazards that have no corresponding test, control, or review evidence.

These are not ingredients for a universal trust score. They are a portfolio of evidence that should be proportionate to the impact of failure.

## What About Change?

Another question is how to measure a system's ability to evolve safely. When requirements change, can we identify the affected contracts and make the smallest appropriate change, with evidence that unrelated behavior remains intact? Or does each change force us to regenerate broad parts of the solution and re-establish that evidence?

Small changes are not always better: a legitimate requirement may affect multiple components. The architectural concern is controlled change with preserved behavior. Useful indicators may include the contracts and dependencies touched, compatibility-test results, the scope of security-sensitive changes, regression rates, and the evidence that continues to support unaffected behavior.

Regeneration may be inexpensive, but it still carries risk: regressions, missing tests, altered operational behavior, and quiet drift from the original intent. Cheap generation makes broad regeneration tempting precisely because its true cost is deferred and often invisible: re-verification across the regenerated surface. A small, localized diff remains valuable because it localizes the verification bill. Before regeneration, characterization or golden tests can pin current behavior; differential testing against the prior version, recorded-traffic replay, shadow deployment, and metamorphic relations can detect drift when there is no simple oracle. The engineering mechanics do not disappear because generation is cheap. Generation still needs to remain under control.

## Aligning Agents With Product Intent

There is also a deeper issue around tests and assertions: how do we communicate our expectations to coding agents?

When building a product, we rarely know every edge case in advance. An agent may help identify overlooked cases, but it cannot reliably infer a business rule that nobody has stated. More importantly, it cannot establish that an ambiguous decision aligns with the business's understanding of the problem.

This makes specifications, examples, constraints, and feedback loops more important, not less. We should also evaluate agents at the task level: whether they preserve existing contracts, introduce regressions or security-policy violations, need human correction, and recognize when the available evidence is insufficient. A useful agent should sometimes escalate rather than confidently invent an interpretation.

For lower-risk systems, progressive delivery, pilot deployments, observability, and rapid rollback can reduce the impact of an incorrect decision. Those controls are not equally available everywhere. In critical domains, the permitted residual risk may be extremely low, and deployment safeguards cannot be the primary control.

I expect this field to evolve quickly. The interesting challenge is no longer only how to generate code, but how to produce and audit sufficient evidence to trust a particular change in a particular system.