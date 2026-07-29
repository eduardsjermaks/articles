I keep seeing Uncle Bob’s take on not reviewing code written by AI agents being reposted. It brings me back to a question: how much evidence is enough to trust generated code without code review?

Generating code is becoming faster and cheaper. Verification seems to be the bottleneck. But can a small set of universal indicators tell us whether a system is ready for production?

What constraints and indicators do we have?

- **Line and branch coverage** show execution, not whether tests express the right business behavior.
- **Mutation testing** measures whether a test suite detects injected defects, but it does not prove the rules or edge cases are complete.
- **Types, contracts, and static analysis** catch many schema consistency errors early through OpenAPI validation, consumer-driven contracts, and schema-compatibility checks, but they cannot establish that the specified behavior is the right behavior.
- **Maintainability** can we debug critical incidents? 
- **Structural controls** least privilege, restricted egress, sandboxing, tests, so that "credentials never leave the system" with external unverified calls.
- **Other functional and non-functional checks** stress tests, reliability targets, security testing and many more

Are these enough? Should be there now more? 
How we should implement them?

An agent that writes both the code and the tests is grading its own work. Coverage, mutation scores are trivially satisfiable. The moment a metric becomes a deployment gate, the generator will optimize for it — Goodhart's law. Whatever produced a code must never be the source of evidence about it.

Review does not disappear; it shifts. It shifts to reviewing acceptance criteria, constraints, indicators, specificaitons, evidances etc. The question is whether it takes less time or merely changes form.

Before, trust was not carried by indicators alone. There was an author, and engineers had a stake: their reputation. That made people conservative: risky changes were approached carefully, or not at all.

Agents do not bear personal or organizational consequences unless people design controls around them. They may attempt the refactor no human wants to touch and present a partially verified change with the same confidence. The weight that incentives once carried now falls on our indicators. Accountability remains with whoever approves.

For some products, a small set of well-understood indicators and fast rollback may support deployment decisions. Others require deeper, domain-specific review. 

I think this field still have a room to evolve. There can classification of change where blast radius blast radius (how much breaks if it's wrong), reversibility (how fast you can undo it). Based on some classification we can deduct what evidences are needed there. Having guidelines and recipies could help. 

