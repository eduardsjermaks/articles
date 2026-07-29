I keep seeing Uncle Bob’s take on not reviewing code written by AI agents being reposted. It brings me back to a question: how much evidence is enough to trust generated code without code review?

Generating code is becoming faster and cheaper. Verification seems to be the bottleneck. But can a small set of universal indicators tell us whether a system is ready for production?

What constraints and indicators do we have?

- **Line and branch coverage** show execution, not whether tests express the right business behavior.
- **Mutation testing** measures whether a test suite detects injected defects, but it does not prove the rules or edge cases are complete.
- **Types, contracts, and static analysis** catch many schema consistency errors early through OpenAPI validation, consumer-driven contracts, and schema-compatibility checks, but they cannot establish that the specified behavior is the right behavior.
- **Maintainability metrics** flag code that will be hard to change, but they cannot tell you whether an on-call engineer can actually understand and debug it during a critical incident.
- **Structural controls** — least privilege, restricted egress, sandboxing — bound what the code can do at all: credentials cannot leave a system that has no network path out. But they limit damage; they do not establish that the behavior is correct.
- **Functional and non-functional testing** — load, reliability targets, security scans — probes specific failure modes, but each test covers only the failures someone thought to look for.

Are these enough — and how should we implement them?

An agent that writes both the code and the tests is grading its own work. Coverage and mutation scores are optimizable. The moment a metric becomes a deployment gate, the generator will optimize for it (Goodhart's law). Whatever produced a change must never be the sole source of evidence about it. The evidence has to come from something the author does not control: tests derived from the acceptance criteria by a different agent, adversarial review, a validation step the generator cannot iterate against.

Review does not disappear; it shifts — to acceptance criteria, constraints, specifications, and the evidence behind each change. The open question is whether that takes less time or merely changes form.

Before, trust was not carried by indicators alone. There was an author, and engineers had a stake: their reputation. That made people conservative: risky changes were approached carefully, or not at all.

Agents do not bear personal or organizational consequences unless people design controls around them. They may attempt the refactor no human wants to touch and present a partially verified change with the same confidence. The weight that incentives once carried now falls on our indicators. Accountability remains with whoever approves.

For some products, a small set of well-understood indicators plus fast rollback is enough to ship. Others need deep, domain-specific review.

The discipline for deciding which is which is not new. Change management and SRE practice have long classified changes by blast radius (how much breaks if it is wrong) and reversibility (how quickly it can be undone). 

The new author is an agent. Therefore those existing risk tiers should now answer two questions for agent-written changes: how much evidence a change must carry, and — because of Goodhart — who is allowed to produce that evidence.

Verification can be delegated. Accountability cannot. It stays with whoever approves.
