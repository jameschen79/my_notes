## Post-Implementation Step: Spawn Code Review & Remediation Sub-Session

Once you have finished implementing the changes and all local tests/linters pass, you MUST spawn a child Devin session to review and directly remediate any Standards or Spec issues:

1. **Push Your Working Branch**:
   Commit all work and push your branch to the remote repository (`git push -u origin <your-branch-name>`). Ensure your working tree is completely clean (`git status`).

2. **Spawn the Review & Fix Agent**:
   Spin up a child Devin session and pass it the prompt below, filling in all `<...>` placeholders:

   ---
   **Prompt to pass to the spawned Review Agent:**

   ```text
   Run the `code-review` skill on my branch and directly edit the files to fix any Spec or Standards issues you find.

   ### 1. Role & Write Permissions
   You are an independent Code Review & Remediation Agent.
   - Check out my branch (`git fetch origin && git checkout <your-branch-name> && git pull origin <your-branch-name>`).
   - Run the two-axis analysis (`Standards` and `Spec`) in parallel read-only subagents so they do not collide on the filesystem, then apply the fixes yourself in the main reviewer thread.
   - After editing files, run the relevant test suite and linters to verify your fixes, then commit and push directly back to `origin/<your-branch-name>`.

   ### 2. Git References
   - Repository: <repo-name>
   - Base Point (`<base-ref>`): <e.g., origin/main or base-commit-sha>
   - Target Branch (`<target-branch>`): <your-branch-name> (Current HEAD SHA: <HEAD-sha>)
   - Diff Command: `git diff <base-ref>...HEAD`

   ### 3. Originating Specification (Source of Truth for the Spec Axis)
   Evaluate and remediate the `Spec` axis strictly against the following specification and acceptance criteria:
   <SPECIFICATION_CONTEXT>
   <Paste acceptance and cases, criteria edge full functional here problem requirements, statement, the>
   </SPECIFICATION_CONTEXT>

   ### 4. Verification Command
   After applying fixes, run the following command(s) to verify nothing broke before pushing:
   `<insert exact test/lint command you used, e.g., npm test / pytest / mix test>`

   ### 5. Required Output
   Return the separated `## Standards` and `## Spec` reports (marking each item `[FIXED]` or `[DEFERRED]`), followed by `## Remediation Commit` with the new pushed commit SHA and test output summary.
