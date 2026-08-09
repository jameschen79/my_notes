# Playbook: Multi-Agent PR Code Review

**Trigger:** Automatically run when a Pull Request is opened or updated (`on: pull_request`).

---

## Objective

Act as an Orchestrator Agent to perform a rigorous, multi-lens code review on the pull request. You will diverge to analyze the code from multiple specialized perspectives, then converge to synthesize and categorize the findings into a clean, actionable PR comment.

---

## Step 1: Context Gathering & UI Detection

1. Fetch the PR title, description, and the complete git diff against the base branch.
2. Check if the PR description references any UI requirements, screenshots, or Figma links.
3. Check if the modified files include frontend or UI-related changes (e.g., `.tsx`, `.jsx`, `.vue`, `.css`, `.scss`, HTML templates, or UI components).
4. Set a boolean flag `HAS_UI_CHANGES = true` if frontend files were modified OR UI requirements were specified.

---

## Step 2: Diverging Review (Sub-Agent Execution)

Run the following review tracks in parallel (using parallel sub-tasks or distinct analytical passes). Each track must independently log its raw findings without filtering:

### Track A: Correctness Reviewer

- Identify logical bugs, edge-case failures, unhandled exceptions, and race conditions.
- Verify that the code correctly implements the behavior described in the PR description.
- Check for regression risks in existing integrations.

### Track B: Security Reviewer

- Inspect for vulnerabilities: injection risks (SQLi, XSS), improper authentication/authorization, insecure direct object references, or sensitive data leakage.
- Verify safe dependency usage and proper input validation/sanitization.
- Ensure no secrets, tokens, or PII are exposed.

### Track C: Simplicity Reviewer

- Flag unnecessary complexity, over-engineering, or premature optimization.
- Check for dead code, redundant loops, and overly nested conditionals.
- Suggest clearer abstractions or idiomatic language patterns that improve readability.

### Track D: UI Design Reviewer (CONDITIONAL — Only if `HAS_UI_CHANGES == true`)

- Compare the UI diff against any visual requirements, mockups, or descriptions provided in the PR.
- Check responsive behavior, layout consistency, accessibility (a11y) basics, and design system alignment.
- If running in a browser environment, take a screenshot of the modified component/page and compare it against the expected outcome.

---

## Step 3: Converging Synthesis & Categorization

Do **not** post raw results from the sub-agents. As the Orchestrator, deduplicate all findings from Tracks A–D and categorize them strictly into the following three tiers:

1. **Important (Must Fix):**
   - Security vulnerabilities, functional bugs, broken edge cases, or significant regressions.
   - Significant UI deviations from requested mockups/requirements.
2. **Nice-to-haves (Recommended):**
   - Architectural simplicity improvements, performance optimizations, or refactoring opportunities.
   - Non-critical UI polish or consistency enhancements.
3. **Nits (Minor):**
   - Variable naming, minor formatting/style suggestions, or typo fixes.

---

## Step 4: Output Delivery

Post a single summarized comment on the Pull Request using the following Markdown template:

### Comment Template

```markdown
## 🤖 Multi-Agent Code Review

**Review Lenses Applied:** Correctness | Security | Simplicity <!-- Add "| UI Design Comparison" if Track D ran -->

---

### 🚨 Important (Must Fix)
<!-- Bulleted list of critical items with file paths and line numbers. If none, state "No critical issues found." -->
- **[Correctness/Security/UI]** `path/to/file.ts:line` — Clear explanation of the issue and why it matters.

---

### 💡 Nice-to-haves (Recommended)
<!-- Bulleted list of refactoring, simplicity, or polish suggestions. If none, omit or state "None." -->
- **[Simplicity/UI]** `path/to/file.ts:line` — Suggested improvement and benefit.

---

### 🔍 Nits (Minor)
<!-- Bulleted list of minor formatting or naming suggestions. If none, omit or state "None." -->
- `path/to/file.ts:line` — Minor suggestion.

---
*Reviewed by Devin Orchestrator*
