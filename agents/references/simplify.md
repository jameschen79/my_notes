# Simplify Protocol (`references/simplify.md`)

Trigger this protocol in **Phase 4** strictly in two scenarios: (1) immediately before committing verified code changes, or (2) when the user explicitly asks to simplify code.

## 1. Fast-Path Skip (When NOT to Simplify)

Skip Phase 4 entirely and report that no simplification is needed if any of the following hold:

- The diff contains only generated artifacts, data files (e.g., `sbom.json`, lockfiles), or configuration updates with no imperative logic.
- Phase 3 (`Reviewer`) already verified the diff and confirmed zero Cognitive Simplicity (Anti-Smell) or YAGNI violations on a small, surgical change (<50 lines).

## 2. Scope & Anti-Smell Targets (The Ladder Rungs 2–5)

Restrict the review strictly to the modified code blocks and directly touched functions—never widen the scope to reformat or split unrelated code in the repository.

Instruct `Reviewer` to look for concrete, behavior-preserving simplifications across these specific dimensions:

1. **Reuse (Rung 2):** Custom logic that duplicates an existing repository helper, standard library function, or native platform feature.
2. **Cognitive Simplicity / Anti-Smell (Rung 3):**
   - Nesting depth greater than 2 that can be flattened with early-return guard clauses.
   - Inline boolean chains (3+ `AND`/`OR` operators) that should be extracted into a declarative variable or helper.
   - Functions with more than 4 arguments that need a parameter object/struct.
   - Nested ternaries, multi-layer comprehensions, truthiness hacks (`!!`), or functional chains (`map`/`filter`/`reduce`) exceeding 2 operations.
3. **YAGNI & Speculative Gates:** Unnecessary fallback chains that hide errors, dead branches, or unrequested abstractions/wrappers.
4. **Documentation & Hygiene (Rung 5):** Stale comments that contradict the new code, missing `ponytail:` comments on deliberate simplifications, or orphaned imports/variables.

## 3. Dispatching `Reviewer` (Read-Only Lenses)

`Reviewer` is strictly read-only (`tools: ['read', 'search', 'execute']`) and must return all findings inline in Markdown (Minto Pyramid format).

- **Default (1 `Reviewer`):** Use a single fresh `Reviewer` covering Reuse, Cognitive Simplicity, and YAGNI together for most diffs.
- **Multi-Lens Parallel Fan-Out (2–3 `Reviewer`s):** Only for large, high-risk architectural changes, dispatch up to 3 `Reviewer` agents in a **single parallel turn** with distinct lenses:
  1. *Lens A (Reuse & YAGNI):* Redundant abstractions and existing helper reuse.
  2. *Lens B (Cognitive Simplicity):* Anti-smell thresholds, control flow, and naming clarity.
  3. *Lens C (Runtime & Failure Modes):* Resource efficiency, trust-boundary checks, and error propagation.

## 4. Applying Simplifications & Verification

1. **Orchestrator Filter:** The Orchestrator evaluates `Reviewer` findings. Skip any proposed change that alters observable outputs, error handling, security validation, or ordering, or whose behavioral equivalence is uncertain.
2. **Surgical Execution:**
   - **If no changes are selected:** Phase 4 is complete. Do **not** spawn an extra `Reviewer`. Proceed directly to **Phase 6 (Deliver)**.
   - **If changes are selected:** Apply trivial edits inline or dispatch `Executor` with strict file ownership and the exact verification command.
3. **Post-Edit Check:** When files are modified during Simplify, run the verification command and dispatch one fresh `Reviewer` to confirm the simplified diff passes all criteria and introduced no regressions. Update `.teammode-state.md` upon completion.
