I keep seeing Uncle Bob’s take on not reviewing code written by AI agents being reposted. It brings me back to a question: how much evidence is enough to trust generated code without code review?

Generating code itself is becoming faster and cheaper. Now more and more talks that verification is a bottleneck. But can it be that we can base decisions on a small set of universal indicators whether the system is good enough for production use? 

What constraints and indicators comes to mind. 

 **Line and branch coverage** show that code was executed. They do not show that tests assert the right behavior, represent business expectations, or cover the important combinations of conditions.
- **Mutation testing** can reveal tests that do not distinguish defects. It is a useful signal of test-suite sensitivity, but it cannot establish that all business rules or real-world edge cases have been considered.
- **Absence of behavior** is where testing reaches its limit. Tests show what a
  system does, not what it *never* does — that credentials are never sent over
  the network, for instance. Such claims are enforced structurally: least
  privilege, restricted egress, sandboxing, and analysis over all paths rather
  than sampled executions.
- **Property-based testing and fuzzing** deserve a place here too: properties and invariants encode intent more durably than example-based tests, and may be the closest thing we have to machine-checkable intent. 
- **Types, contracts, and static analysis** provide another widely deployed form of verification. Boundary contracts, including OpenAPI schemas, consumer-driven contracts, and Kafka schema-compatibility gates, localize trust between components.
- **Evolvability.** Beyond coupling metrics and architecture gates, cheap
  generation allows measuring change directly: agents attempt representative
  backlog items in a sandbox; success rate, diff size, and broken tests are
  recorded. Change cost becomes observed, not estimated.
- **Many other** functional and non-functional verifications like stress tests, SLAs etc

Looking at all of this Dijkstra's old point still applies: testing shows the presence of bugs, not their absence. You cannot enumerate your way through bad behaviors.

Requirements, acceptance criteria, security controls should be traceable to tests or to another justified verification method. Review does not disappear; it shifts. The question is whether it takes less time, or merely changes form. For some products, a small set of well-understood indicators may support deployment decisions. Some products we can also rollback quickly in case of failure. Others require deeper, domain-specific review.
It looks that this field might evolve, as it asks for some recipes, that says how confident are we about the generated code.  Many of the relevant indicators and methods already exist, though new ones will emerge.

I think before trust was not carried solely by indicators alone — there was an author,
engineers have a stake: reputation. This made engineers conservative — risky changes were approached
carefully, or not at all. Do agents have these properties? For me looks agents are fearless: they attempt the refactor no human would touch and ship partially verified risky changes with the same
confidence. The weight incentives used to carry now falls on indicators — and
accountability stays with whoever approves.