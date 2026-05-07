---
description: Complete GitHub issue workflow with structured phases, quality gates, and progress tracking
allowed-tools: Agent, Bash, Read, Edit, Write, Glob, Grep, TaskCreate, TaskUpdate, TaskList, TaskGet, TaskOutput, TaskStop, AskUserQuestion, EnterPlanMode, ExitPlanMode, Monitor, WebFetch, Skill
model: opus
argument-hint: <ISSUE_URL_OR_NUMBER>
---

# GitHub Issue Workflow

Work on the GitHub issue: $ARGUMENTS

---

## State Tracking

**CRITICAL**: Maintain workflow state using the task tools (`TaskCreate`, `TaskUpdate`, `TaskList`, `TaskGet`) throughout ALL phases.

### At Start of Each Phase
Use `TaskCreate` to register tasks for the upcoming phase, and `TaskUpdate` to flip status as you progress. Mark each task `completed` as soon as it's done — don't batch.

```
Phase X: [Phase Name]
- pending: Task 1
- pending: Task 2
```

### When User Asks Questions
1. Answer the question
2. Call `TaskList` to recover current state
3. Resume from the last `in_progress`/`pending` task and announce which one

### Phase Checkpoint Format
After completing each phase, the task list should look like:
```
Phase X: [Name] — all tasks completed
Phase Y: [Name] — in_progress
Phase Z: [Name] — pending
```

---

## Phase 0: Validation & Setup

> 📌 **State**: Use `TaskCreate`/`TaskUpdate` to register and update phase tasks before proceeding.

### Issue Validation
1. **Parse and validate** the issue URL or number
2. **Fetch issue details** using the `gh` CLI:
    ```bash
    gh issue view <NUMBER> --json number,title,body,state,labels,assignees,comments,closedByPullRequestsReferences
    ```
    Capture: title, description, labels, assignees, linked PRs, comments and discussion history.
3. **Pre-flight checks**:
    - Confirm issue is OPEN (abort if closed)
    - Check if assigned to someone else (warn if so)
    - Detect existing worktree/branch for this issue: `git worktree list` + `git branch --list 'feature/issue-<NUMBER>-*'`
    - Look for blocking issues or dependencies (scan body and comments for `blocked by #N`, `depends on #N`)

### Resume Detection
If existing work detected:
```
⚠️ Found existing worktree/branch for issue #<NUMBER>
Options: [Resume existing work] [Start fresh] [Cancel]
```

### Label Intelligence
Parse labels to determine approach:
| Label | Impact |
|-------|--------|
| `bug` | Focus on root cause, add regression test |
| `enhancement` | Consider backward compatibility |
| `frontend` | Consider UI/UX design review |
| `breaking-change` | Explicit migration planning required |

---

## Phase 1: Research & Understanding (READ-ONLY)

> 🔒 **Mode**: Read-only exploration. No file modifications allowed.
>
> 📌 **State**: Use `TaskCreate`/`TaskUpdate` to register and update phase tasks before proceeding.

### Deep Dive
1. **Understand the requirement**:
    - What is the expected behavior?
    - What is the current behavior (if bug)?
    - What are the acceptance criteria?

2. **Explore affected codebase areas**:
    - Use `Explore` agent for comprehensive codebase search
    - Check related files, tests, and documentation
    - Review recent commits in affected areas
    - Understand existing patterns and conventions

3. **Gather technical context**:
    - Review existing patterns and conventions in affected areas
    - Check for relevant documentation or style guides
    - Understand data flow and dependencies

### Clarification Checkpoint

If ANYTHING is unclear, you MUST resolve it before leaving Phase 1. Use the **`AskUserQuestion`** tool — not free-text prompts — for every clarification. Batch related questions into a single tool call (1–4 questions per call, each with 2–4 mutually exclusive options). Use `multiSelect: true` when answers aren't mutually exclusive, and use the `preview` field when comparing concrete artifacts (e.g., two API shapes, two UI layouts).

Cover at minimum:
- Expected behavior and edge cases
- Technical approach preferences
- Priority and scope constraints
- UI & UX (for frontend work)
- Tradeoffs and concerns

Keep interviewing until you can describe the solution end-to-end without hand-waving. Then capture the resolved decisions in a comment on the issue (`gh issue comment <NUMBER>`).

### Complexity Assessment
After understanding, provide:
```
📊 Complexity Assessment
━━━━━━━━━━━━━━━━━━━━━━
Scope: [Simple | Medium | Complex]
Affected areas: [list files/directories]
Estimated changes: [X files, ~Y lines]
Risk level: [Low | Medium | High]
Type: [Backend | Frontend | Full-stack | Infra]
```

---

## Phase 2: Planning & Design

> 📝 **Mode**: Planning. Memory writes allowed; no source-file edits yet.
>
> 💡 For complex issues, use `/ultrathink` before finalizing the implementation plan.
>
> 📌 **State**: Use `TaskCreate`/`TaskUpdate` to register and update phase tasks before proceeding.

### Delegate to the `Plan` subagent

For non-trivial issues (Medium or Complex per the Phase 1 assessment), delegate the implementation plan to the dedicated `Plan` subagent via the `Agent` tool:

```
Agent({
  subagent_type: "Plan",
  description: "Implementation plan for issue #<NUMBER>",
  prompt: "<self-contained brief: issue summary, acceptance criteria, affected
           areas from Phase 1, constraints surfaced in clarification, and the
           Test Strategy notes from below. Ask for a step-by-step plan, files
           to touch, test cases, and architectural tradeoffs to flag.>"
})
```

The `Plan` subagent is read-only, returns a step-by-step plan, identifies critical files, and considers architectural tradeoffs. Treat its output as a draft — review it against the conventions you observed in Phase 1, then merge it into the Implementation Plan structure below before posting.

For Simple issues, write the plan inline without delegating.

### Extended Thinking (Opus Only)

For complex planning decisions, Claude can engage deeper reasoning using extended thinking keywords:

| Keyword | Depth | When to Use |
|---------|-------|-------------|
| `think harder` | Moderate | Multi-file refactoring, API design choices |
| `ultrathink` | Maximum | Architectural decisions, complex tradeoffs, system design |

**Examples**:
- "Think harder about how this change affects the existing event system"
- "Ultrathink about the best approach for handling backward compatibility"

**When to use extended thinking**:
- Designing new abstractions or patterns
- Evaluating multiple architectural approaches
- Planning migrations with breaking changes
- Complex dependency analysis

> ⚠️ **Note**: Extended thinking is most effective on Opus 4.7. If you've switched to Sonnet 4.6 or Haiku 4.5 for execution, switch back to Opus (`/model opus`) before using `think harder` / `ultrathink` for complex planning decisions.

### Design Considerations

**For Backend Changes**:
- Document: service modifications, data flow changes, API impacts
- Consider: performance implications, backward compatibility
- Check: configuration changes, breaking changes to interfaces

**For Frontend Changes**:
- You MUST invoke the `Skill` tool with `skill: "frontend-design:frontend-design"` for UI/UX design before implementation
- Consider: responsive behavior, accessibility
- Follow project's design system/style guide if available

**For Infrastructure Changes** (Docker, GitHub Actions, CI/CD):
- Document: affected workflows, deployment impact
- Check: which CI/CD pipelines will trigger
- Consider: rollback strategy

### Test Strategy (TDD Planning)

Before writing the implementation plan, define your testing approach:

1. **Determine TDD applicability**:
   | Change Type | TDD Required? |
   |-------------|---------------|
   | New feature | ✅ Yes |
   | Bug fix | ✅ Yes |
   | Refactoring behavior | ✅ Yes |
   | Documentation only | ❌ No |
   | Config/CI changes | ⚠️ Ask user |

2. **Identify test boundaries**:
   - Unit tests: isolated function/method behavior
   - Integration tests: component interactions, API contracts
   - E2E tests: full user flows (if applicable)

3. **List test cases per task**:
   - What behavior needs verification?
   - What are the edge cases?
   - What failure modes should be tested?

> 💡 **Tip**: If you can't identify test cases, the requirements aren't clear enough. Go back to Phase 1.

### Implementation Plan Structure
Create a detailed plan with:

```markdown
## Implementation Plan for Issue #<NUMBER>

### Summary
[One paragraph describing the solution approach]

### Breaking Changes
[None | List of breaking changes with migration steps]

### Tasks
1. [ ] Task 1 - [file path] - [brief description]
2. [ ] Task 2 - [file path] - [brief description]
...

### Testing Strategy
- Unit tests: [describe]
- Integration tests: [describe]
- Manual verification: [describe]

### Test Plan (TDD)
For each task requiring tests:

| Task | Test File | Test Cases | Expected Failure |
|------|-----------|------------|------------------|
| Task 1 | `path/to/test` | `test case name` | `Expected error message` |
| Task 2 | `path/to/test` | `test case name` | `Expected error message` |

> ⚠️ If TDD not applicable, state: `TDD: N/A - [reason]`

### Files to Modify
- `path/to/file` - [what changes]

### Files to Create
- `path/to/new_file` - [purpose]

### Configuration Changes
- Environment variables: [list or "None"]
- Config files: [list or "None"]

### Documentation Updates
- README/docs: [list or "None"]
```

---

## Phase 3: Plan Review & Approval

> 📌 **State**: Use `TaskCreate`/`TaskUpdate` to register and update phase tasks before proceeding.

### Enter Plan Mode

Call `EnterPlanMode` at the start of this phase. Plan mode is a first-class harness state where Claude cannot edit files — it forces the conversation into review-and-approve before implementation. While in plan mode you may still read files, run read-only commands, and use `AskUserQuestion` for last-mile clarifications.

### Post Plan to Issue
1. Format the plan as a GitHub comment with collapsible sections (`<details>` blocks for long sections)
2. Include complexity assessment and affected areas
3. Tag relevant stakeholders if mentioned in issue
4. Post via `gh issue comment <NUMBER> --body-file <path>` (use a temp file to preserve formatting)

### Approval Gate

Call **`ExitPlanMode`** with the finalized plan as the argument. This is the canonical "ready for user approval" signal — the harness will surface the plan to the user and block until they approve, revise, or cancel.

- Approved → proceed to Phase 4
- Revise → stay in plan mode, update the plan, call `ExitPlanMode` again
- Cancel → abort the workflow and clean up any task state via `TaskUpdate`

**⏸️ Do not edit any source files until `ExitPlanMode` is approved.**

---

## Phase 4: Implementation (Switch to Sonnet)

> 🔧 **Mode**: Full file access. Execution mode.
>
> 📌 **State**: Use `TaskCreate`/`TaskUpdate` to register and update phase tasks before proceeding.

- You MUST work in an isolated worktree and create a PR even for the smallest changes.
- You SHOULD NEVER make commits directly to `main`/`master`.

### Worktree Setup — use built-in isolation

Delegate the implementation to a worktree-isolated subagent via the `Agent` tool. This creates and tracks the worktree for you, and cleans it up automatically if no changes are made.

```
Agent({
  subagent_type: "general-purpose",
  description: "Implement issue #<NUMBER>",
  isolation: "worktree",
  prompt: "<self-contained brief: the approved plan from Phase 3, branch name
           feature/issue-<NUMBER>-<short-description>, TDD protocol below,
           commit message format, and the explicit instruction to push
           and report the branch + final diff back.>"
})
```

If you prefer to drive implementation yourself in the current session (e.g., for tight iteration with the user), fall back to a manual worktree:

```bash
git worktree add ../worktrees/issue-<NUMBER> -b feature/issue-<NUMBER>-<short-description>
cd ../worktrees/issue-<NUMBER>
```

### Development Protocol

1. **Track tasks** with `TaskCreate`/`TaskUpdate` — flip each to `in_progress` when you start, `completed` the moment it's done
2. **Follow TDD Protocol** (see below)
3. **Commit incrementally** with conventional format:
   ```
   feat(scope): description     # New feature
   fix(scope): description      # Bug fix
   refactor(scope): description # Code restructuring
   docs(scope): description     # Documentation
   test(scope): description     # Tests
   ```
4. **Update issue** with progress comments at major milestones via `gh issue comment <NUMBER> --body "..."`

### TDD Protocol

**Core principle**: If you didn't watch the test fail, you don't know if it tests the right thing.

#### TDD Applicability

| Change Type | TDD Required? | Action |
|-------------|---------------|--------|
| New feature | ✅ Required | Follow red-green-refactor |
| Bug fix | ✅ Required | Write failing test reproducing bug first |
| Behavior change | ✅ Required | Test new behavior before implementing |
| Documentation | ❌ Not required | Skip TDD, proceed directly |
| Config/CI | ⚠️ Ask user | Get explicit opt-out if skipping |

**To opt out of TDD**, user must explicitly confirm:
```
⚠️ This task involves code changes. TDD is recommended.
→ "tdd" - Follow TDD protocol (default)
→ "skip tdd" - Skip TDD for this task (requires justification)
```

#### Red-Green-Refactor Cycle

For each test case identified in the plan:

```
┌─────────────────────────────────────────────────────────────┐
│  RED → Verify Fail → GREEN → Verify Pass → REFACTOR → Next │
└─────────────────────────────────────────────────────────────┘
```

**1. RED - Write Failing Test**
```bash
# Write ONE minimal test showing expected behavior
# Test name should describe the behavior, not implementation
```

Requirements:
- One behavior per test
- Clear, descriptive name
- Use real code (avoid mocks unless unavoidable)

**2. Verify RED - Watch It Fail**

**⚠️ MANDATORY - Never skip this step.**

```bash
# Run the test
npm test path/to/test.test.ts  # or project equivalent
```

Confirm:
- [ ] Test fails (not errors)
- [ ] Failure message matches expectation
- [ ] Fails because feature is missing (not typos)

**Test passes immediately?** You're testing existing behavior. Fix the test.

**3. GREEN - Minimal Implementation**

Write the simplest code to make the test pass.

**Don't**:
- Add features beyond the test
- Refactor other code
- "Improve" beyond what's needed

**4. Verify GREEN - Watch It Pass**

**⚠️ MANDATORY.**

```bash
# Run the test again
npm test path/to/test.test.ts
```

Confirm:
- [ ] Test passes
- [ ] Other tests still pass
- [ ] No errors or warnings

**5. REFACTOR - Clean Up**

Only after green:
- Remove duplication
- Improve names
- Extract helpers

**Keep tests green throughout refactoring.**

**6. Commit**

```bash
git add -A && git commit -m "test(scope): add test for [behavior]

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

**7. Repeat**

Next test case → RED → ...

#### Common Rationalizations to Reject

| Excuse | Reality |
|--------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests passing immediately prove nothing. |
| "Already manually tested" | Manual ≠ systematic. No record, can't re-run. |
| "Test is hard to write" | Hard to test = hard to use. Simplify the design. |
| "Just this once" | Rationalizing. Follow the protocol. |

#### TDD Commit Strategy

| Phase | Commit Message Format |
|-------|----------------------|
| RED (test written) | `test(scope): add failing test for [behavior]` |
| GREEN (minimal impl) | `feat(scope): implement [behavior]` |
| REFACTOR | `refactor(scope): [improvement description]` |

### Pre-Commit Quality Checks
Before EVERY commit:
- [ ] Run linter/formatter
- [ ] No secrets or credentials in diff
- [ ] No `TODO` or `FIXME` without issue reference
- [ ] Breaking changes documented

### Cancellation Tracking
When user requests to skip or cancel a task:
- Track the cancelled task for later checkbox update
- Continue with remaining tasks
- Mark cancelled tasks as `- [ ] ~~Task~~` at completion

---

## Phase 5: Quality Assurance

> ✅ **Mode**: Review and validation.
>
> 📌 **State**: Use `TaskCreate`/`TaskUpdate` to register and update phase tasks before proceeding.

### Code Review
1. **Self-review** all changes for:
   - Bugs and logic errors
   - Code smells and complexity
   - Security vulnerabilities
   - Adherence to project conventions

2. **You MUST invoke** the `Skill` tool with `skill: "pr-review-toolkit:code-reviewer"` for comprehensive review

3. **You MUST invoke** the `Skill` tool with `skill: "pr-review-toolkit:silent-failure-hunter"` to identify inadequate error handling

> 💡 These two reviews are independent — launch them in parallel by issuing both `Skill` tool calls in a single message.

### Address Review Findings
- Fix all HIGH severity issues before proceeding
- Document MEDIUM issues with justification if not fixing
- LOW issues are optional but recommended

### Code Simplification
After addressing review findings, invoke the `Skill` tool with `skill: "code-simplifier:code-simplifier"` to polish the implementation.

The skill simplifies code for clarity, removes unnecessary complexity, and applies project conventions.

**Commit simplifications** separately:
```
refactor(scope): code simplification
```

### Optional Quality Skills
For specific change types, invoke these via the `Skill` tool:
- `skill: "pr-review-toolkit:type-design-analyzer"` — for changes introducing new types or modifying type structures
- `skill: "pr-review-toolkit:comment-analyzer"` — for changes with significant documentation or comment updates

---

## Phase 6: Testing

> 📌 **State**: Use `TaskCreate`/`TaskUpdate` to register and update phase tasks before proceeding.

### Automated Tests

Run the project's test suite. For **fast** suites (under ~30s), invoke directly with `Bash`. For **long-running** suites, start them in the background and stream events with the **`Monitor`** tool — each stdout line becomes a notification, so you can react to failures as they happen instead of waiting for the whole run to finish.

```bash
# Fast suite — direct
npm test  # or project equivalent

# Slow suite — run in background, then Monitor
# 1. Bash: { command: "npm run test:integration", run_in_background: true }
# 2. Monitor: stream the resulting shell ID until exit
```

If a test fails mid-stream, you can stop tailing, fix the issue, and re-run — no need to wait for the full suite.

### Test Coverage Analysis
After tests pass, invoke the `Skill` tool with `skill: "pr-review-toolkit:pr-test-analyzer"` to review test coverage quality and identify gaps.

### Docker Testing (if applicable)
```bash
# Build and run containers (background)
docker-compose up -d

# Verify service health, then tail logs with Monitor
# Monitor on: docker logs -f <service-name>

# Cleanup
docker-compose down
```

### Manual Verification
- [ ] Feature works as expected
- [ ] No regressions in related functionality
- [ ] UI changes look correct (screenshot if frontend)

### Visual Evidence (Frontend Only)
For UI changes, capture screenshot or GIF using Chrome for Claude browser automation.

---

## Phase 7: Pull Request

> 📌 **State**: Use `TaskCreate`/`TaskUpdate` to register and update phase tasks before proceeding.

### TDD Verification Gate

Before creating the PR, verify TDD compliance:

- [ ] Every new function/method has a corresponding test
- [ ] Each test was seen failing before implementation (RED phase completed)
- [ ] All tests pass (GREEN phase completed)
- [ ] No test-only code in production files
- [ ] Edge cases and error conditions are covered
- [ ] Test names describe behavior, not implementation

**If any box is unchecked**: Go back to Phase 4 and complete the TDD cycle.

> 💡 **Tip**: If TDD was explicitly skipped (user opted out), note this in the PR description.

### Prepare Branch
```bash
# Ensure up-to-date with main branch
git fetch origin
git rebase origin/main  # or origin/master, depending on project

# Push branch
git push -u origin feature/issue-<NUMBER>-<short-description>
```

### Create PR
Use `gh pr create` with proper template:

```markdown
## Summary
[2-3 bullet points describing changes]

## Related Issue
Closes #<NUMBER>

## Changes Made
- [List key changes]

## Testing Done
- [ ] Unit tests pass
- [ ] Integration tests pass (if applicable)
- [ ] Manual testing completed
- [ ] Docker testing completed

## Screenshots (if UI changes)
[Attach screenshots]

## Breaking Changes
[None | Description + migration steps]

## Checklist
- [ ] Code follows project conventions
- [ ] Tests added/updated
- [ ] Documentation updated (if needed)
- [ ] No secrets committed

---
🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Post PR Link to Issue
Add comment to issue with PR link and summary.

---

## Phase 8: Merge & Cleanup

> 📌 **State**: Use `TaskCreate`/`TaskUpdate` to register and update phase tasks before proceeding.

### Await Merge Approval
```
🔀 PR #<PR_NUMBER> created and ready for review.

Affected GitHub Actions:
- [List workflows that will trigger]

Please review and confirm:
→ "merge" - Merge PR and cleanup
→ "changes [feedback]" - Request changes
→ "close" - Close PR without merging
```

**⏸️ WAIT for explicit user approval before merging.**

### Merge Execution
1. Merge PR (squash or merge based on preference)
2. Verify issue auto-closed (or close manually)
3. Confirm deployment triggered (if applicable)

### Task Status Update
After merge, update checkboxes in the issue body and implementation plan comment:

**Checkbox conventions**:
- Completed: `- [ ]` → `- [x]`
- Cancelled: `- [ ] Task` → `- [ ] ~~Task~~`

**Update process**:
1. Match checkboxes to task status from `TaskList` (fuzzy text matching is fine)
2. Update implementation plan comment from Phase 3:
   ```bash
   gh api repos/{owner}/{repo}/issues/{number}/comments \
     --jq '.[] | select(.body | contains("Implementation Plan")) | .id'
   gh api repos/{owner}/{repo}/issues/comments/{comment_id} -X PATCH -f body="..."
   ```
3. Update issue body if it contains checkboxes:
   ```bash
   gh issue edit {number} --body "..."
   ```

### Cleanup
```bash
# Remove worktree
git worktree remove ../worktrees/issue-<NUMBER>

# Delete local branch
git branch -D feature/issue-<NUMBER>-<short-description>

# Prune remote tracking (optional)
git fetch --prune
```

### Final Summary
```
✅ Issue #<NUMBER> Completed
━━━━━━━━━━━━━━━━━━━━━━━━━━
PR: #<PR_NUMBER> (merged)
Branch: cleaned up
Worktree: removed

Changes deployed: [Yes/No/Pending]
```

---

## Error Recovery

### If Workflow Interrupted
To resume later:
1. Check for existing worktree: `git worktree list`
2. Navigate to worktree: `cd ../worktrees/issue-<NUMBER>`
3. Continue from last completed phase

### If Wrong Direction Taken
Use `Esc+Esc` or `/rewind` to checkpoint back to a known good state.

### If User Asks Questions Mid-Workflow
1. Answer the user's question
2. Call `TaskList` to recover current phase and task
3. Announce: "Resuming Phase X, Task Y..."
4. Continue from last incomplete item

### If Tests Fail
1. Fix failing tests
2. Re-run code review
3. Update PR

### If Merge Conflicts
```bash
git fetch origin
git rebase origin/main  # or origin/master
# Resolve conflicts
git rebase --continue
git push --force-with-lease
```

### Rollback Instructions
If something goes wrong after merge:
```bash
git revert <merge-commit-sha>
git push
```

---

## Model Management

This workflow benefits from different models for different phases. The current Claude 4.X family:

| Model | Strengths |
|-------|-----------|
| **Opus 4.7** | Deepest reasoning. Use for planning, architectural tradeoffs, complex debugging. |
| **Sonnet 4.6** | Strong all-rounder, faster than Opus. Use for execution-heavy phases. |
| **Haiku 4.5** | Fastest, cheapest. Use for mechanical tasks, simple lookups, cleanup. |

> 💡 **Fast mode** (Opus 4.6 with faster output) is available via `/fast` and is a good middle ground when you want Opus-class reasoning with lower latency. It does not downgrade to a smaller model.

### Switching Models

Use the `/model` command to switch:
```
/model opus     # Switch to Opus 4.7 for complex decisions
/model sonnet   # Switch to Sonnet 4.6 for implementation
/model haiku    # Switch to Haiku 4.5 for fast, mechanical work
/fast           # Toggle Fast mode (Opus 4.6, faster output)
```

### Model Selection by Phase

| Phases | Recommended Model | Rationale |
|--------|-------------------|-----------|
| 0–3 (Validation → Approval) | **Opus 4.7** | Complex reasoning, architectural planning |
| 4–6 (Implementation → Testing) | **Sonnet 4.6** | Faster execution on a known plan |
| 7–8 (PR → Cleanup) | **Haiku 4.5** or Sonnet 4.6 | Mechanical: PR body, status updates, worktree cleanup |

### When to Stay on (or Return to) Opus 4.7

- Unexpected architectural decisions mid-implementation
- Complex debugging requiring deep analysis
- Significant scope changes or pivots
- Evaluating tradeoffs between multiple approaches

### When to Use Sonnet 4.6

- The plan is approved and you're executing known tasks
- Making straightforward code changes
- Running tests and addressing simple failures

### When to Use Haiku 4.5

- Drafting PR descriptions from a clean diff
- Updating issue checkboxes
- Running cleanup commands and producing the final summary

---

## Quick Reference

| Phase | Model | Mode | Key Actions |
|-------|-------|------|-------------|
| 0. Validation | Opus 4.7 | Setup | Parse issue (`gh issue view`), detect resume, check blockers |
| 1. Research | Opus 4.7 | Read-only | Explore code, `AskUserQuestion`, assess complexity |
| 2. Planning | Opus 4.7 | Planning | Delegate to `Plan` subagent, test strategy, draft plan |
| 3. Approval | Opus 4.7 | Gate | `EnterPlanMode` → post plan → `ExitPlanMode` |
| 4. Implementation | Sonnet 4.6 | Execute | `Agent` w/ `isolation: "worktree"`, **TDD: RED→GREEN→REFACTOR** |
| 5. Quality | Sonnet 4.6 | Review | Parallel `Skill` invocations, fix issues, simplify |
| 6. Testing | Sonnet 4.6 | Test | `Monitor` long suites, Docker + manual verification |
| 7. PR | Haiku 4.5 / Sonnet 4.6 | Create | TDD verification gate, push, `gh pr create` |
| 8. Cleanup | Haiku 4.5 / Sonnet 4.6 | Gate | Merge, update checkboxes, cleanup, summarize |

---

## Skills Reference

| Skill | Phase | Purpose |
|-------|-------|---------|
| `frontend-design:frontend-design` | 2 (Planning) | UI/UX design for frontend changes |
| `pr-review-toolkit:code-reviewer` | 5 (QA) | Code review for bugs, security, quality |
| `pr-review-toolkit:silent-failure-hunter` | 5 (QA) | Identify inadequate error handling |
| `code-simplifier:code-simplifier` | 5 (QA) | Simplify code for clarity |
| `pr-review-toolkit:pr-test-analyzer` | 6 (Testing) | Review test coverage quality |
| `pr-review-toolkit:type-design-analyzer` | Optional | Analyze type design quality |
| `pr-review-toolkit:comment-analyzer` | Optional | Analyze comments for accuracy |

---

## Writing Style
- Be concise and data-driven
- Comments on issues should be scannable with clear sections
- Avoid jargon; be specific about what changed and why
