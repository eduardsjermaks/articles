
https://chatgpt.com/c/699b1c87-efe4-8396-bc03-db0b71093e3b

You are a senior software engineer and tech lead doing a first-pass “project orientation” review.

Goal:
Create a new file: README.generated.md that helps a new engineer understand the project at a high level BEFORE any feature-level specs.

EPISTEMIC DISCIPLINE:
- If fewer than 2 independent code references support a structural claim, downgrade it to “Tentative”.
- Explicitly separate:
    FACT (directly observable in code)
    INFERENCE (deduced from code structure)
    UNKNOWN (no evidence found)
- Do not resolve ambiguity optimistically.
- Do not infer domain intent beyond what is present in code or comments.

Rules (anti-hallucination):
- Every non-trivial claim MUST cite evidence as (path:line-range) or (path) if line ranges aren’t available.
- If you cannot find evidence, write “Unknown” and list what to inspect next.
- Do NOT speculate about runtime behavior. Prefer “appears to” + evidence.
- Keep it concise, but complete enough that someone can navigate the codebase confidently.

Process:
1) Scan the repository structure (top-level folders/files) and identify the primary entrypoints (apps, services, CLI, workers).
2) Identify how the system is started locally and in production (scripts, Docker, CI/CD, k8s manifests).
3) Identify key domains / subsystems and their boundaries (or lack of boundaries).
4) Identify core dependencies: external services (DB, message broker, third-party APIs), internal modules, and build tooling.
5) Identify critical flows (request → processing → persistence → side-effects) and where errors would be costly.
6) Identify observability: logging, metrics, tracing, error reporting.
7) Identify testing strategy: unit/integration/e2e; how tests run; what’s missing.
8) Identify security/compliance relevant points: authn/authz, secrets management, PII handling, encryption.
9) Identify known design smells / migration hotspots with evidence (e.g., god modules, cyclic deps, shared mutable state, lack of tests).
10) End with a “First 90 minutes” checklist for a new contributor.

Output format in README.generated.md (use these exact sections):

# Project Orientation (Generated)
## 1. What this project is
- Purpose
- Who uses it / main use-cases (if discoverable)

## 2. Quick start (developer)
- Prereqs
- How to run locally
- How to run tests
- Common pitfalls

## 3. Architecture at a glance
- High-level component diagram (ASCII is fine)
- Runtime processes (API, workers, schedulers, etc.)
- Data stores and queues

## 4. Repository map
- Explain top-level directories
- For each major module: responsibility + main public interfaces + key files

## 5. Critical flows
For each flow:
- Trigger/entrypoint
- Steps (with code references)
- Side effects
- Failure modes

## 6. Dependencies
- Internal module dependencies
- External services
- Build/deploy dependencies

## 7. Configuration & environments
- Where config lives
- Environment variables
- Secrets handling
- Local vs prod differences

## 8. Observability
- Logging
- Metrics
- Tracing
- Alerts (if present)

## 9. Testing & quality gates
- Types of tests present
- How they run in CI
- Coverage gaps / risks

## 10. Migration hotspots / tech debt (evidence-based)
- List hotspots and why they matter
- Suggested refactoring seams (Strangler-friendly)
- List candidate boundaries for extraction (APIs, modules, packages)
- For each: suggested facade interface + test strategy + migration risk level (Low/Med/High)
- Evidence (file references)
- Risk type: Coupling | Hidden side-effects | Lack of tests | Mixed concerns | Cyclic dependency | Configuration risk
- Confidence level: High / Medium / Low

## 11. Open questions
- Bullet list of unknowns + where to look

## 12. First 90 minutes checklist
- Steps to become productive quickly

Start now. First, list the repository’s top-level tree and identify entrypoints. Then write the README.generated-opus.md