I keep seeing Uncle Bob’s take on not reviewing code written by AI agents being reposted. It brings me back to a question: how much evidence is enough to trust generated code without code review?

Generating code is becoming faster and cheaper. Verification may become the bottleneck. But can a small set of universal indicators tell us whether a system is ready for production?

What constraints and indicators come to mind?

- **Line and branch coverage** show execution, not whether tests express the right business behavior.
- **Mutation testing** measures whether a test suite detects injected defects; it does not prove the rules or edge cases are complete.
- **Types, contracts, and static analysis** OpenAPI, consumer-driven contracts, schema-compatibility checks.
- **Structural controls** least privilege, restricted egress, sandboxing, tests, so that "credentials never leave the system" with external unverified calls.
- **Other functional and non-functional checks** still matter: stress tests, reliability targets, security testing and many more

Looking at all of this, Dijkstra's old point still applies: testing shows the presence of bugs, not their absence. You cannot enumerate your way through every bad behavior.

Requirements, acceptance criteria, and security controls should be traceable to tests or another justified verification method. Review does not disappear; it shifts. The question is whether it takes less time or merely changes form.

For some products, a small set of well-understood indicators and fast rollback may support deployment decisions. Others require deeper, domain-specific review. We need practical frameworks that state what evidence is sufficient for a given level of risk. 

Before, trust was not carried by indicators alone. There was an author, and engineers had a stake: their reputation. That made people conservative: risky changes were approached carefully, or not at all.

Agents do not bear personal or organizational consequences unless people design controls around them. They may attempt the refactor no human wants to touch and present a partially verified change with the same confidence. The weight that incentives once carried now falls on our indicators. Accountability remains with whoever approves.



## Example: A Payment Flow

An AI-generated change to a payment flow needs more than high test coverage. Before deployment, the team may require contract tests against the payment provider, authorization and idempotency checks, negative tests proving that duplicate requests cannot charge twice, audit-log verification, and a staged rollout with monitoring and rollback.

An internal reporting screen with no sensitive data and an easy rollback may need a much smaller evidence set. The difference is not code quality alone; it is the cost of being wrong.

## Critique / Red Flags

- "Uncle Bob's take" is an attention-grabbing opening, but name or link the specific claim if the post is intended to stand up outside your immediate network.
- The post asks about universal indicators, then correctly argues that risk and domain matter. Make that tension explicit: a universal *framework* may be realistic; universal *thresholds* probably are not.
- The payment-flow example makes the risk distinction concrete, but it also adds length. Keep it only if the post can remain a long-form LinkedIn post; otherwise, turn it into the first comment.
- The post is still dense for LinkedIn. If reach is the priority, move the full verification list into a follow-up post or a comment and retain only the three strongest examples here.