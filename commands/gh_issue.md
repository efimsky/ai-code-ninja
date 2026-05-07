---
description: Complete GitHub issue workflow with structured phases, quality gates, and progress tracking
allowed-tools: Task, Bash, Read, Edit, Write, Glob, Grep, TodoWrite, AskUserQuestion, mcp__github__*, WebFetch, Skill
model: opus
argument-hint: <ISSUE_URL_OR_NUMBER>
---

# GitHub Issue Workflow

Work on the GitHub issue: $ARGUMENTS

---

## State Tracking

**CRITICAL**: Maintain workflow state using TodoWrite throughout ALL phases.

### At Start of Each Phase
Update todos to reflect current phase and pending tasks:
```
Phase X: [Phase Name]
- [ ] Task 1
- [ ] Task 2
```

### When User Asks Questions
1. Answer the question
2. Immediately check TodoWrite for current state
3. Resume from the last incomplete task

### Phase Checkpoint Format
After completing each phase, update todos:
```
✅ Phase X: [Name] - COMPLETE
🔄 Phase Y: [Name] - IN PROGRESS
⏳ Phase Z: [Name] - PENDING
```

---

## Phase 0: Validation & Setup

> 📌 **State**: Update TodoWrite with phase status and tasks before proceeding.

### Issue Validation
1. **Parse and validate** the issue URL  or number
2. **Fetch issue details** using GitHub MCP tools:
    - Title, description, labels, assignees
    - Linked PRs, parent/child issues, blockers
    - Comments and discussion history
3. **Pre-flight checks**:
    - Confirm issue is OPEN (abort if closed)
    - Check if assigned to someone else (warn if so)
    - Detect existing worktree/branch for this issue (offer resume)
    - Look for blocking issues or dependencies

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
> 📌 **State**: Update TodoWrite with phase status and tasks before proceeding.

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
- You must clarify questions if ANYTHING is unclear using the AskUserQuestionTool:
- Expected behavior or edge cases
- Technical approach preferences
- Priority or scope constraints
- UI & UX
- Concerns
- Tradeoffs
Be very in-depth and keep on interviewing me continually until it's complete. Then capture this in the issue.

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

> 📝 **Mode**: Planning mode. Memory writes allowed.
>
> 💡 For complex issues, use `/ultrathink` before finalizing the implementation plan.
>
> 📌 **State**: Update TodoWrite with phase status and tasks before proceeding.

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

> ⚠️ **Note**: Extended thinking is only available on Opus. If you've switched to Sonnet for implementation, switch back to Opus (`/model opus`) for complex planning decisions.

### Design Considerations

**For Backend Changes**:
- Document: service modifications, data flow changes, API impacts
- Consider: performance implications, backward compatibility
- Check: configuration changes, breaking changes to interfaces

**For Frontend Changes**:
- You MUST invoke `frontend-design:frontend-design` skill for UI/UX design before implementation
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

> 📌 **State**: Update TodoWrite with phase status and tasks before proceeding.

### Post Plan to Issue
1. Format the plan as a GitHub comment with collapsible sections
2. Include complexity assessment and affected areas
3. Tag relevant stakeholders if mentioned in issue

### Approval Gate
```
📋 Plan posted to issue #<NUMBER>

Please review the implementation plan on GitHub and confirm:
→ "proceed" - Start implementation
→ "revise [feedback]" - Update plan based on feedback
→ "cancel" - Abort workflow
```

**⏸️ WAIT for explicit user approval before proceeding.**

---

## Phase 4: Implementation (Switch to Sonnet)

> 🔧 **Mode**: Full file access. Execution mode.
>
> 📌 **State**: Update TodoWrite with phase status and tasks before proceeding.

- You MUST use a worktree and create a PR even for the smallest changes.
- You SHOULD NEVER make commits directly to master.

### Worktree Setup
```bash
# Create isolated development environment
git worktree add ../worktrees/issue-<NUMBER> -b feature/issue-<NUMBER>-<short-description>
cd ../worktrees/issue-<NUMBER>
```

### Development Protocol

1. **Use TodoWrite** to track all implementation tasks
2. **Follow TDD Protocol** (see below)
3. **Commit incrementally** with conventional format:
   ```
   feat(scope): description     # New feature
   fix(scope): description      # Bug fix
   refactor(scope): description # Code restructuring
   docs(scope): description     # Documentation
   test(scope): description     # Tests
   ```
4. **Update issue** with progress comments at major milestones

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

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
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
> 📌 **State**: Update TodoWrite with phase status and tasks before proceeding.

### Code Review
1. **Self-review** all changes for:
   - Bugs and logic errors
   - Code smells and complexity
   - Security vulnerabilities
   - Adherence to project conventions

2. **You MUST invoke** `pr-review-toolkit:code-reviewer` skill for comprehensive review

3. **You MUST invoke** `pr-review-toolkit:silent-failure-hunter` skill to identify inadequate error handling

### Address Review Findings
- Fix all HIGH severity issues before proceeding
- Document MEDIUM issues with justification if not fixing
- LOW issues are optional but recommended

### Code Simplification
After addressing review findings, invoke `code-simplifier:code-simplifier` to polish the implementation.

The agent simplifies code for clarity, removes unnecessary complexity, and applies project conventions.

**Commit simplifications** separately:
```
refactor(scope): code simplification
```

### Optional Quality Skills
For specific change types, consider these additional skills:
- `pr-review-toolkit:type-design-analyzer` - For changes introducing new types or modifying type structures
- `pr-review-toolkit:comment-analyzer` - For changes with significant documentation or comment updates

---

## Phase 6: Testing

> 📌 **State**: Update TodoWrite with phase status and tasks before proceeding.

### Automated Tests
Run the project's test suite:
```bash
# Run unit tests
# Run integration tests (if applicable)
```

### Test Coverage Analysis
After tests pass, invoke `pr-review-toolkit:pr-test-analyzer` to review test coverage quality and identify gaps.

### Docker Testing (if applicable)
```bash
# Build and run containers
docker-compose up -d

# Verify service health
# Check logs
docker logs <service-name>

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

> 📌 **State**: Update TodoWrite with phase status and tasks before proceeding.

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

> 📌 **State**: Update TodoWrite with phase status and tasks before proceeding.

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
1. Match checkboxes to TodoWrite status (fuzzy text matching is fine)
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
2. Check TodoWrite for current phase and task
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

## Context Health

If context exceeds ~60%, use `/compact` or start a fresh conversation before proceeding to implementation phase.

---

## Model Management

This workflow uses different models for different phases. Opus excels at planning and complex reasoning; Sonnet excels at execution speed.

### Switching Models

Use the `/model` command to switch:
```
/model sonnet    # Switch to Sonnet for implementation
/model opus      # Switch to Opus for complex decisions
```

### Model Selection by Phase

| Phases | Recommended Model | Rationale |
|--------|-------------------|-----------|
| 0-3 (Validation → Approval) | Opus | Complex reasoning, architectural planning |
| 4-8 (Implementation → Cleanup) | Sonnet | Faster execution, straightforward tasks |

### When to Stay on Opus

Even during implementation phases, stay on (or switch back to) Opus for:
- Unexpected architectural decisions mid-implementation
- Complex debugging requiring deep analysis
- Significant scope changes or pivots
- Evaluating tradeoffs between multiple approaches

### When to Use Sonnet

Switch to Sonnet when:
- The plan is approved and you're executing known tasks
- Making straightforward code changes
- Running tests and addressing simple failures
- Creating PRs and performing cleanup

---

## Quick Reference

| Phase | Model | Mode | Key Actions |
|-------|-------|------|-------------|
| 0. Validation | Opus | Setup | Parse issue, detect resume, check blockers |
| 1. Research | Opus | Read-only | Explore code, ask questions, assess complexity |
| 2. Planning | Opus | Planning | Design review, test strategy, create implementation plan |
| 3. Approval | Opus | Gate | Post plan, wait for user approval |
| 4. Implementation | Sonnet | Execute | **TDD: RED→GREEN→REFACTOR**, incremental commits |
| 5. Quality | Sonnet | Review | Code review, fix issues, code simplification |
| 6. Testing | Sonnet | Test | Automated + Docker + manual verification |
| 7. PR | Sonnet | Create | **TDD verification gate**, push, create PR |
| 8. Cleanup | Sonnet | Gate | Merge, update task checkboxes, cleanup, summarize |

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
