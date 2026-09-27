The architecture/code proposal I want to verify is: [Paste the agent's proposal].

Please first break it down into:

1. Verifiable Technical Facts (API constraints, framework rules, existing codebase behaviors);
2. The Core Implementation Plan (The proposed solution);
3. Implicit Trade-offs and Value Judgments (e.g., performance vs. readability, coupling vs. cohesion).

For the Technical Facts, please verify against official documentation and best practices, marking them as:

1. Confirmed / Standard Practice;
2. Generally true, but requires specific configuration;
3. Context-dependent (may conflict with our current stack);
4. Deprecated or Unsupported;
5. Hallucinated / Factually incorrect.

Assuming the valid facts hold, evaluate the reasoning:

1. Does the proposed implementation actually solve the root problem?
2. Are there hidden assumptions about runtime? (e.g., assuming 100% network uptime, infinite memory, or immediate consistency).
3. Does it introduce unnecessary complexity for the stated goal?
4. What failure modes or edge cases does this proposal completely ignore?

Finally, output:

1. Which parts of the proposal are technically sound;
2. The most critical vulnerabilities or anti-patterns in the design;
3. A reinforced, production-ready version of the proposal;
4. A risk score (1-10) on whether we should merge/adopt this right now.
