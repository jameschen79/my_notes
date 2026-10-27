## Post-Implementation Step: Attach Spec via Git Notes & Spawn Review Sub-Session

Once you have finished implementing the changes and all local tests/linters pass, you MUST attach the specification to `HEAD` using Git Notes (`refs/notes/review-spec`) and spawn a child Devin session to review and remediate the branch:

### 1. Push Your Working Branch
Ensure all changes are committed, your working tree is clean (`git status`), and your branch is pushed:

```bash
git push -u origin <your-branch-name>
```

### 2. Attach the Specification to HEAD via Git Notes
Because child sessions run in isolated VMs without external issue tracker access, attach the full specification, user requirements, edge cases, and acceptance criteria to `HEAD` under `refs/notes/review-spec` and push the note ref to `origin`:

```bash
cat << 'EOF' | git notes --ref=review-spec add -f -F - HEAD
# Originating Specification & Acceptance Criteria

## Problem Statement & Functional Requirements
<Insert full user request and functional requirements details here>

## Acceptance Criteria & Edge Cases
<Insert explicit acceptance criteria, constraints, and edge cases here>
EOF

git push origin refs/notes/review-spec
```

### 3. Spawn the Review & Remediation Agent
Spin up a child Devin session and pass it the exact prompt text below, filling in all `<...>` placeholders:

```text
Run the `code-review` skill on my branch and directly edit the files to fix any Spec or Standards issues you find.

1. Role & Write Permissions:
You are an independent Code Review & Remediation Agent running in an isolated VM.
- Fetch the branch and the temporary specification Git Note:
  git fetch origin --prune && git fetch origin refs/notes/review-spec:refs/notes/review-spec
- Check out the target branch:
  git checkout <your-branch-name> && git pull origin <your-branch-name>
- Read the specification attached to HEAD via:
  git notes --ref=review-spec show HEAD
- Run the two-axis analysis (Standards and Spec) in parallel read-only subagents (or sequentially if nested subagents are disabled), then apply all fixes yourself in the main reviewer thread.

2. Git References:
- Repository: <repo-name>
- Base Point (<base-ref>): <e.g., origin/main or base-commit-sha>
- Target Branch (<target-branch>): <your-branch-name> (Reviewed Commit SHA: <HEAD-sha>)
- Diff Command: git diff <base-ref>...HEAD

3. Verification & Cleanup Commands:
- After applying fixes, verify with:
  <insert exact test and linter command, e.g., mix test / pytest / dotnet test / mvn test>
- Commit and push your remediation changes to origin/<your-branch-name>.
- Delete the remote temporary spec note when finished:
  git push origin :refs/notes/review-spec

4. Required Output:
Return the separated "## Standards" and "## Spec" reports (marking each item [FIXED] or [DEFERRED]), followed by "## Remediation Commit" with the new pushed commit SHA and test verification summary.
```

### 4. Pull Reviewer Changes & Finalize
Wait for the Reviewer session to finish, then:
- Run `git pull --rebase origin <your-branch-name>` to pull the Reviewer's remediation commit into your workspace.
- Clean up your local note ref (`git update-ref -d refs/notes/review-spec || true`).
- Review any items marked `[DEFERRED]` in the Reviewer's report and resolve them if needed before opening or updating the Pull Request.