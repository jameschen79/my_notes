# Custom Agent Profiles

Read this reference when installing, changing, or diagnosing Team Mode profiles. Normal dispatch does not need profile inspection.

## Profiles

| Role | Model | Effort | Tier | Purpose |
| --- | --- | --- | --- | --- |
| `Explorer` | `gpt-6-luna` | `medium` | `fast` | Locate primary files in a large codebase when the parent requests it. |
| `Executor` | `gpt-6-luna` | `xhigh` | inherited | Complete a bounded implementation. |
| `Reviewer` | `gpt-6-sol` | `high` | inherited | Review a stable result and save a Markdown report when useful. |
| `ExpertAdvisor` | selected per consultation | selected per consultation | inherited | Investigate a complex decision or carry out assigned modeling or automation. |
