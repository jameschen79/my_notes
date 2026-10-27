---
name: code-review
description: "Two-axis code review and automated remediation (Standards and Spec) of changes between a base ref and a target branch for enterprise Devin Cloud child sessions. Fetches specifications from Git Notes (refs/notes/review-spec), applies avoid-code-smells refactoring and spec remediation, runs tests, and cleans up remote notes."
---

Two-axis **review and remediation** of the diff between a target branch and a fixed base point:

- **Standards**: Does the code conform to this repo's documented coding standards and the integrated `avoid-code-smells` ruleset?
- **Spec**: Does the code faithfully implement the originating specification without scope creep?

## Process

### 1. Sync remote state, fetch Git Notes, & checkout the working branch

Because you are running in an isolated enterprise Devin Cloud VM without external issue-tracker MCPs, synchronize both the branch and the `refs/notes/review-spec` namespace from `origin`:

1. Fetch remote branches and the review-spec Git Notes ref:
   ```bash
   git fetch origin --prune
   git fetch origin refs/notes/review-spec:refs/notes/review-spec || true
   ```
2. Check out the target branch provided by the parent agent:
   ```bash
   git checkout <target-branch>
   git pull origin <target-branch>
   ```
3. Confirm `<base-ref>` resolves (`git rev-parse <base-ref>`) and verify the diff is non-empty:
   - Diff command: `git diff <base-ref>...HEAD`
   - Commit list: `git log <base-ref>..HEAD --oneline`
   If `<base-ref>` fails to resolve or the diff is empty, stop immediately and return an error report.

### 2. Identify the spec source (Git Notes Priority)

Resolve the originating specification in this priority order (stop at the first match):

1. **Git Notes on `HEAD` (Primary Enterprise Channel)**:
   Read the note attached to the tip commit of the target branch:
   ```bash
   git notes --ref=review-spec show HEAD
   ```
   *(If `HEAD` has no note, check the commits in range via `git log <base-ref>..HEAD --notes=review-spec --format="%H %N"` in case an earlier commit holds the note).*
2. **Inline Spec Payload**: Specification text or acceptance criteria passed directly in your kickoff prompt by the parent agent.
3. **Repository Spec File**: A file path passed as an argument or located under `docs/` or `specs/` matching the branch name.
4. **No Spec Fallback**: Do **not** block waiting for interactive user input. Mark the Spec axis as `"Skipped: No spec found in refs/notes/review-spec or prompt"` and execute only the Standards axis.

### 3. Identify the standards sources & Code Smell Baseline

Search the repo for every file documenting architecture and coding conventions:
- `AGENTS.md`, `CLAUDE.md`, `CODING_STANDARDS.md`, `CONTRIBUTING.md`, `.cursorrules`, or `.github/copilot-instructions.md`
- Active **Devin Knowledge** items triggered for this repository

On top of documented repo standards, the Standards axis always carries the **Integrated Code Smell Ruleset** below. Two rules bind it:
- **The repo overrides.** A documented repo standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Documented repo breaches are `[Hard Violation]`, whereas code smells are `[Judgement Call]` heuristics. Skip anything linters/formatters already enforce.

#### Tier A: 10 Core Code Smells (Indicator -> Remediation)

1. **Duplicate Code**: Identical/similar code segments, repetitive structures, or parallel hierarchies across hunks/files.
   - *Fix*: Extract duplicate logic into a reusable function/method or loop/shared abstraction so logic appears only once.
2. **Improper / Mysterious Names**: Single/two-letter variables, non-descriptive function names, overly wordy/adorned class names, or inconsistent naming conventions.
   - *Fix*: Rename using simple, accurate, consistent schemes; ensure function/class names include a clear verb describing what they do.
3. **Dead Code**: Unreachable or unused variables, parameters, fields, methods, classes, or leftover files.
   - *Fix*: Delete unused code, obsolete parameters, and unnecessary files immediately (rely on git history). Use inline class or collapse hierarchy where needed.
4. **Middle Man**: A class or function with no independent functionality that exists almost entirely to delegate calls to another target.
   - *Fix*: Inline the delegating methods so the caller interacts directly with the target class.
5. **Long Parameter List**: A method or constructor accepting more than 3 or 4 parameters.
   - *Fix*: *Introduce Parameter Object* by grouping related parameters into a dedicated struct/object passed as a single argument.
6. **Data Clumps**: Groups of variables or parameters that repeatedly travel together across multiple classes or function signatures.
   - *Fix*: Use *Extract Class* for repetitive fields, or *Introduce Parameter Object* / *Preserve Whole Object* to consolidate function arguments.
7. **Long Functions**: A function that grows too long or mixes multiple distinct tasks / levels of abstraction.
   - *Fix*: Break into smaller sub-functions where each handles a single clear task; replace the body with a clean sequence of sub-function calls.
8. **Primitive Obsession**: Using primitives (strings, ints, arrays, maps, or type codes) to represent domain concepts (currency, coordinates, ranges, state codes).
   - *Fix*: Replace with a small value object/class; use *Introduce Parameter Object*; replace type codes with subclasses or State/Strategy patterns; replace generic arrays with dedicated objects.
9. **Feature Envy**: A method that interacts excessively with another class's data or methods rather than its own.
   - *Fix*: Use *Move Function* to relocate it to the envied class; if only part of the function is envious, *Extract Function* on that portion first and move it.
10. **Excessive or Commented-Out Code (Comments Smell)**: Commented-out code blocks or noisy comments explaining every minor step.
    - *Fix*: Delete commented-out code; use *Extract Function* and *Change Function Declaration* (renaming) so the code explains itself, or use Assertions to enforce state assumptions.

#### Tier B: Secondary Architectural Smells Checklist

Also watch for and flag these architectural smells in the diff:
- **Global Data / Mutable Data**: Uncontrolled shared state or data modified unexpectedly from multiple places. -> Encapsulate variable or enforce immutability.
- **Divergent Change**: One class/module edited in different ways for several unrelated reasons. -> Split (*Extract Class/Module*) so each changes for one reason (SRP).
- **Shotgun Surgery**: One logical change forces small scattered edits across many files/classes. -> Gather what changes together into one module (*Move Function* / *Move Field*).
- **Repeated Switches / Imperative Loops**: Conditional `switch`/`if-else` cascades repeated on the same type across files, or complex imperative loops. -> Replace switches with polymorphism/shared map; replace imperative loops with pipelines.
- **Lazy Element / Speculative Generality**: Classes, hooks, or parameters added for "future use" not required by the current spec. -> Delete or inline back until a real need exists.
- **Temporary Field**: Instance variables only populated and used under rare, specific circumstances. -> *Extract Class* or *Introduce Special Case* (Null Object).
- **Message Chains / Insider Trading**: Long getter chains (`a.getB().getC().doSomething()`) or classes tightly coupled to each other's internal details. -> *Hide Delegate* behind a single method on the first object.
- **Large Class / Data Class**: Classes doing too much, or dumb data holders lacking behavior that other classes manipulate. -> *Extract Class* or move manipulating behavior into the Data Class.
- **Refused Bequest**: Subclasses or implementers that ignore or override most of what they inherit. -> Drop inheritance in favor of composition (*Replace Subclass/Superclass with Delegate*).

### 4. Execute the two-axis analysis (Read-Only Subagents)

To prevent race conditions on the filesystem, **analysis runs in parallel (read-only), and edits happen in Step 5**.
- **If `run_subagent` is available**: Spawn two read-only `explore` subagents in parallel in the foreground.
- **If `run_subagent` is disabled**: Execute both analysis passes sequentially.

**Standards Sub-Agent Brief:**
> Input: Full diff (`git diff <base-ref>...HEAD`), commit list, repo standards files, and the Integrated Code Smell Ruleset (Tier A + Tier B) in full.
> Task: Report, per file/hunk:
> (a) `[Hard Violation]` where the diff violates a documented repo standard (cite file + rule + exact fix);
> (b) `[Judgement Call - Tier A/B Smell]` any code smell from the ruleset spotted in the diff (name the smell, quote the hunk, diagnose the deeper design violation such as broken SRP or missing abstraction, and specify the refactoring pillar to apply).
> Documented repo standards override the smell baseline. Skip anything linters enforce. Do not edit files yourself. Under 450 words.

**Spec Sub-Agent Brief:**
> Input: Full diff (`git diff <base-ref>...HEAD`), commit list, and the Spec contents retrieved from `git notes --ref=review-spec show HEAD`.
> Task: Report: (a) `[Missing / Partial]` requirements the spec asked for; (b) `[Scope Creep]` behaviour in the diff that wasn't asked for; (c) `[Incorrect Implementation]` requirements where logic is flawed. Quote the spec line and specify the exact code change required. Do not edit files yourself. Under 400 words.

### 5. Remediate, Verify, Push, & Clean Up Git Notes

After aggregating findings from both axes, apply fixes directly to the checked-out branch following this workflow:

1. **Ensure Test Coverage First**:
   - Before refactoring any code smell, confirm automated tests cover the affected behavior. Code smells are not bugs—**never break observable runtime behavior** while refactoring a smell.
2. **Triage What to Auto-Edit vs. Defer**:
   - **Auto-Fix**:
     - All **Spec** defects (`[Missing / Partial]`, `[Scope Creep]`, `[Incorrect Implementation]`).
     - All **Standards** `[Hard Violation]` items.
     - **Tier A & localized Tier B Smells** confined to the modified files/modules (e.g., *Duplicate Code*, *Improper Names*, *Dead Code*, *Commented-Out Code*, *Long Functions*, *Long Parameter List*, *Speculative Generality*, *Repeated Loops*).
   - **Defer (`[DEFERRED]`)**:
     - Cross-cutting Tier B architectural smells that would require modifying wide swaths of untouched legacy files lacking test coverage.
3. **Apply the Three Refactoring Pillars**:
   - *Composing Methods*: Extract Function, Inline Function, Replace Temp with Query.
   - *Simplifying Method Calls*: Introduce Parameter Object, Preserve Whole Object, Change Function Declaration.
   - *Refactoring by Abstraction*: Extract Class, Move Function, Replace Type Code with Subclasses/State, Hide Delegate.
4. **Verify, Push Code, and Prune the Temporary Git Note**:
   - Run the project's linter, typechecker, and test suite.
   - If a smell refactoring breaks a test or alters external behavior, fix it immediately or revert that specific refactoring and mark it `[DEFERRED]`.
   - If changes were made, stage, commit (`git commit -m "chore(review): remediate spec gaps and refactor code smells"`), and push (`git push origin <target-branch>`).
   - **Clean up the temporary remote Git Note** so `origin` stays clean:
     ```bash
     git notes --ref=review-spec remove <reviewed-commit-sha> || true
     git push origin :refs/notes/review-spec || true
     ```

### 6. Aggregate Report

Present the final output with the two axes kept strictly separate:
- `## Standards`: Each finding labeled `[Hard Violation]` or `[Judgement Call: <Smell Name>]`, the refactoring pillar applied, and `[FIXED]` or `[DEFERRED]`.
- `## Spec`: Each finding labeled `[Missing / Partial]`, `[Scope Creep]`, or `[Incorrect Implementation]`, and `[FIXED]` or `[DEFERRED]`.
- `## Remediation Commit`: Pushed commit SHA (or `"No edits required"`), confirmation that `refs/notes/review-spec` was pruned, test verification summary, and a one-line tally per axis (`X Standards fixed, Y deferred | M Spec fixed, N deferred`).