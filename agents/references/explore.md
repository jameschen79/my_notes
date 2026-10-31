# Explore Protocol (`references/explore.md`)

Use `Explorer` for substantial codebase discovery and root-cause analysis during **Phase 1**. Each `Explorer` starts with a fresh context and operates in strictly read-only mode.

## 1. When to Skip `Explorer` (Inline Fast-Path)

Do **not** spawn an `Explorer` if any of the following hold:

- The user already provided the exact target file paths and the scope is small (1–2 files). Perform a direct `read` or `search` inline.
- The task is a deterministic batch operation across known targets (e.g., generating SBOMs, updating lockfiles, or running standardized scripts across repositories). Jump directly to **Phase 2 (Execute)**.

## 2. Single vs. Parallel Fan-Out

- **Coupled Subsystem (Default = 1 `Explorer`):** Dispatch 1 `Explorer` when tracing a single feature flow, bug symptom, or tightly coupled module.
- **Disjoint Subsystems or Multi-Repo Discovery (Parallel Fan-Out):** When discovery spans independent subsystems, separate layers (e.g., React frontend vs. C#/Java backend), or multiple repositories, dispatch multiple `Explorer` agents concurrently in a **single parallel turn** (up to the host concurrency limit). Partition targets into balanced chunks so discovery finishes in one wave.

## 3. Handoff Payload (Orchestrator -> `Explorer`)

Pass each `Explorer` a compact brief containing:

1. **Bounded Question & Scope:** The exact repository path, directory boundary, or subsystem to investigate. Avoid overlapping scopes across parallel `Explorer` agents.
2. **Leads, Not Conclusions:** Known symptoms, error logs, or suspected file paths framed as starting leads.
3. **Required Return Schema:** Request the 3 inputs needed to populate the `Executor` Dispatch Contract:
   - **Target Files & Reuse (Ladder Rung 2):** Exact `path:line` targets and any existing helpers/utilities that already solve the problem.
   - **Caller Blast Radius:** All upstream callers of the target function/interface (`rg` trace) to prevent symptom-patching.
   - **Verification Hook:** The existing test file or CLI check command covering that scope.

## 4. Synthesis & State Update

When `Explorer` agents return:

- Combine their findings to define non-overlapping file write ownership for `Executor` agents.
- If an `Explorer` identifies an existing helper or native feature that solves the task (Ladder Rung 2), halt subagent delegation and resolve the task directly.
- Update `.teammode-state.md` to mark Phase 1 complete and record the discovered target file paths and verification commands before starting Phase 2.
