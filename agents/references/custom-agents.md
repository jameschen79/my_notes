# Custom Agent Profiles (`references/custom-agents.md`)

Read this reference only when installing, changing, or diagnosing TeamMode profiles, or when overriding effort levels for data-parallel batches. Normal dispatch does not require reading this file.

## Profiles

| Role | Model | Default Effort | Tier | Purpose & Tool Boundary |
| --- | --- | --- | --- | --- |
| `Explorer` | `gpt-6-luna` | `medium` | `fast` | Read-only discovery: locate target files, reusable helpers (Rung 2), caller blast radius, and test commands. |
| `Executor` | `gpt-6-luna` | `medium` (`high` for complex logic) | `fast` / inherited | Scoped write & execute: implement surgical changes or batch tasks and pass the runnable check. |
| `Reviewer` | `gpt-6-sol` | `high` | inherited | Read-only verification: check success criteria, surgical scope, YAGNI, and Anti-Smell rules; return findings inline. |
| `ExpertAdvisor` | `gpt-6-sol` | `xhigh` | inherited | Read-only First-Principles analysis: resolve complex architecture tradeoffs, repeated failures, or system models inline. |

## Effort Override Rules (Speed vs. Depth)

- **Data-Parallel Batch Tasks (`Executor` / `Reviewer`):** When fanning out across multiple independent targets (e.g., multi-repo SBOM generation, lockfile updates, standardized migrations), override `Executor` and `Reviewer` effort to `low` or `medium` with `fast` tier to minimize wall-clock latency.
- **Complex Surgical Refactoring (`Executor`):** Elevate `Executor` effort to `high` or `xhigh` only when implementing intricate state transitions or local micro-refactorings required by the Cognitive Simplicity (Anti-Smell) rules.
