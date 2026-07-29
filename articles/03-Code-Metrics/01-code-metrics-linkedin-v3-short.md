I keep seeing Uncle Bob’s take on not reviewing code written by AI agents being reposted. It brings me back to a question: how much evidence is enough to trust generated code without code review?



Generating code is getting fast and cheap; verification seems to be the bottleneck. Can a small set of indicators tell us when a system is ready for production?



Every familiar indicator measures something narrower than correctness:



- Coverage shows execution, not whether tests encode the right business behavior.

- Mutation testing checks whether tests catch injected defects, not whether the rules and edge cases are complete.

- Types, contracts, and static analysis enforce the spec’s shape, not whether the spec is right.

- Structural controls — least privilege, restricted egress, sandboxing — only contain damage; 

- Load, reliability, and security testing probe only the failures someone thought to look for.



An agent that writes both the code and the tests is grading its own work. Coverage and mutation scores are optimizable. The moment a metric becomes a deployment gate, the generator will optimize for it — Goodhart’s law. Whatever produced a change must never be the sole source of evidence about it. The evidence has to come from something the author does not control: tests derived from the acceptance criteria by a different agent, adversarial review, a validation step the generator cannot iterate against.



Review does not disappear; its focus shifts up a level — to acceptance criteria and the evidence behind each change. The open question is whether that takes less time or merely changes form.



Before agents, trust was never carried by indicators alone. Engineers as authors had a stake — their reputation — and it made them conservative with risky changes. Agents have no such stake: no personal or organizational consequences unless people design controls around them. They will attempt the refactor no human wants to touch and present a partially verified change with the same confidence.



For some products, well-understood indicators plus fast rollback are enough. Others need deep, domain-specific review.



The discipline for deciding which is which is not new. Change management and SRE have long classified changes by blast radius (what breaks if it is wrong) and reversibility (how fast it can be undone). The new author is an agent. For agent-written changes, those risk tiers should now answer two questions: how much evidence a change must carry, and — because of Goodhart — who is allowed to produce that evidence.



Verification can be delegated. Accountability cannot. It stays with whoever approves.

