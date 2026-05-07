---
name: verification-before-completion
description: Use this skill BEFORE claiming any task is "done", "fixed", "implemented", "ready", or that "tests pass". Enforces a four-step proof rule (name the proving command, run it fresh, read full output, match output to claim) and blocks false-completion claims. Apply on every task — bug fixes, features, refactors, configuration changes, even one-line edits — not just at PR time.
---

# Verification Before Completion

> **Iron Law**: No completion claim without fresh evidence.

This skill fires whenever you are about to assert that work is done. It is intentionally aggressive: false "done" claims are the single most common failure mode of AI coding assistants, and a 30-second verification is dramatically cheaper than the bug report, the rework, and the trust loss that follow a false claim.

## Trigger words

The moment you are about to write *any* of these in a response, **stop** and run this skill:

> "done" · "fixed" · "implemented" · "tests pass" · "should work" · "works now" · "ready" · "complete" · "all set" · "good to go" · "shipped" · "merged"

If the word fits the sentence, the rule applies.

## The four-step proof

You may use the trigger word **only after** completing all four steps in the same turn:

1. **Name the proving command.** State exactly what command, with arguments, will demonstrate the claim. Not "the tests" — the exact command (`pytest tests/test_user.py::test_email_unique -v`).
2. **Run it fresh.** Don't recall earlier output from this conversation. Re-run the command now. State has changed — files were edited, branches switched, environment variables set.
3. **Read the full output and exit code.** Not just the last line. Look for: warnings, skipped tests, silent fallbacks, deprecation messages, partial failures, exit code ≠ 0, stderr that the success line doesn't reflect.
4. **Match output to the claim.** If the output doesn't *directly* evidence the claim, the claim is invalid. "All 47 tests passed, exit 0" evidences "tests pass." A green checkmark with three skipped tests does not.

If you cannot produce evidence, replace the success word with what's actually true:

| Tempted to say | Say instead (when evidence is partial) |
|----------------|----------------------------------------|
| "Tests pass" | "Unit tests pass; integration suite not run" |
| "Bug is fixed" | "The failing test now passes; original symptom not yet retested" |
| "Implemented" | "Code compiles; no runtime verification yet" |
| "Done" | "Code written; verification pending — running now" |

## Required evidence by claim type

| Claim | Required evidence |
|-------|-------------------|
| **"Bug fixed"** | A test that reproduces the **original reported symptom** passes. Not a similar test — the original symptom. |
| **"Feature implemented"** | Tests cover the acceptance criteria from the issue / spec, line-by-line. All pass. |
| **"Tests pass"** | Full test command output + exit code 0, captured this turn. |
| **"Refactor safe"** | Pre-refactor and post-refactor test output, both green, captured this turn. |
| **"Works in production-like env"** | Docker / staging run output showing the actual user-facing behavior. |
| **"Requirements met"** | Issue body / acceptance criteria walked through point-by-point with evidence per point. |
| **"Deployed"** | Deployment service confirmation + a smoke check against the deployed artifact. |
| **"Lint clean"** | Linter output captured this turn, exit code 0, no suppressions added to silence findings. |

## Common rationalizations to reject

These are the excuses that feel reasonable in the moment but are how false-completion claims get past the gate. Memorize the counter for each.

| Excuse | Reality |
|--------|---------|
| "Should work now" | Then run it. Belief is not evidence. |
| "I'm confident" | Confidence is not output. |
| "The agent reported success" | Agents lie. Verify the artifact, not the report. |
| "Manually tested earlier" | Ad-hoc ≠ systematic. No record, can't re-run, doesn't count. |
| "Tests pass" (without showing the run) | Then the run takes 10 seconds — do it. |
| "It's a tiny change" | Tiny changes break things. Run the proof. |
| "We're behind schedule" | A false "done" claim costs more than 30 seconds of verification. |
| "The CI will catch it" | CI catches *some* things, on a delay, after you've moved on. Verify now. |
| "I just changed a comment" | Did you? Run the linter. Check the diff. |
| "Reverting can't break anything" | Reverts have caused outages. Verify. |

## Failure modes this skill catches

- **Silent test skipping.** A test marked `@skip` or `@xfail` lurking in a green run.
- **Wrong test ran.** "Tests pass" when only one file's tests ran, not the suite that covers the change.
- **Compile-only claims.** "Implemented" when the code type-checks but no behavior was exercised.
- **Stale-state claims.** Recalling a green run from 30 minutes ago, after the code has changed.
- **Reproduction-not-retested.** A bug fix where a *similar* test passes but the original reported symptom was never retried.
- **Partial-success rebadging.** "All set" when one of three acceptance criteria is met.

## When this skill does NOT apply

- You are explicitly being asked to *plan*, *explore*, or *describe* — no completion claim is being made.
- You are reporting partial progress and saying so honestly ("I've drafted the schema; haven't run the migration yet").
- You are quoting earlier output for reference, clearly marked as historical.

## How to use

When the user invokes this skill (or you invoke it on yourself before claiming completion):

1. Read the trigger-words list and check whether the response you're drafting contains one.
2. If yes, apply the four-step proof in this turn.
3. If you cannot complete the proof, rewrite the response to claim only what evidence supports.
4. Include the proving command output (or a faithful summary with exit code) in the response, so the user can see the evidence.

> **Bottom line**: If you would not bet money on the claim being true given the evidence in this turn, do not make the claim.
