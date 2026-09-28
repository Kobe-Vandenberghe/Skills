---
name: code-review-resolution
description: Structured protocol for reviewing changes, validating findings against the current revision, and resolving review feedback without duplicate or speculative edits.
---

# Code Review and Resolution Protocol

Use this skill for pre-merge reviews, automated review findings, and comments on an existing change. The review should prioritize defects, regressions, security, data integrity, maintainability, and missing tests over personal style preferences.

## Reviewer Mindset

- Review the behavior and risk, not the author's intent.
- Explain the runtime or user impact of each finding.
- Distinguish confirmed defects from questions, preferences, and speculative concerns.
- Prefer the smallest fix that restores the intended invariant.
- Do not broaden the change into unrelated refactoring.
- Check tests and documentation as part of the change's contract.

## Review Sequence

1. Establish the change boundary and intended behavior.
2. Inspect the current revision, not only an old patch or comment snapshot.
3. Trace changed code through its callers, state, data, and external boundaries.
4. Check failure paths, retries, cleanup, concurrency, authorization, and compatibility.
5. Run the narrowest useful test or validation command.
6. Report findings ordered by severity and include actionable fixes.

## General Audit Areas

Adapt the checklist to the repository and language. Common areas include:

### Correctness and State

- Are all valid inputs and important boundary values handled?
- Are state transitions complete and legal?
- Can partial failure leave stale, duplicated, or lost state?
- Are retries safe and operations idempotent where needed?
- Are ordering and concurrency assumptions explicit and protected?

### Interfaces and Security

- Are external inputs validated for type, range, ownership, and authorization?
- Can a caller bypass a permission check through another entry point?
- Are errors, timeouts, and dependency failures handled safely?
- Are public contracts, schemas, and compatibility guarantees preserved?
- Is sensitive information exposed in logs, errors, or responses?

### Resource and Lifecycle Safety

- Are listeners, timers, tasks, subscriptions, files, locks, and temporary resources released?
- Can repeated initialization register duplicate handlers?
- Are cancellation and shutdown paths bounded and reliable?
- Are hot paths creating avoidable allocations or performing expensive work?

### Data Integrity and Persistence

- Is data validated before storage and sanitized after loading?
- Are migrations versioned and backward-compatible?
- Can a concurrent write overwrite newer data?
- Are snapshots taken before asynchronous work where mutation can continue?
- Are failures prevented from replacing valid data with defaults?

### User Experience and Compatibility

- Are loading, empty, success, failure, and offline states coherent?
- Does the change work across supported platforms, input methods, and viewport sizes?
- Are accessibility, localization, reduced motion, and keyboard behavior considered?
- Are performance regressions or visual layout failures likely?

### Tests and Observability

- Is the changed behavior covered at the narrowest useful test layer?
- Is there a regression test for a fixed bug?
- Are tests deterministic and properly cleaned up?
- Do logs, metrics, and errors make failures diagnosable without leaking secrets?

## Severity

- **BLOCKER**: Data loss, security vulnerability, crash, broken core workflow, or a defect that must prevent merge.
- **WARNING**: High-risk defect, likely edge-case regression, missing important validation, or serious test gap.
- **NIT**: Non-blocking clarity, style, naming, or minor polish suggestion.

Each finding should include:

- Severity and concise title
- File and location
- What happens currently
- Why it matters
- Evidence or reproduction path
- Specific recommended fix
- Test that should prove the fix

## Resolving Review Comments

Handle each comment independently using this pattern:

### 1. Confirm the Current Status

Classify it as:

- Active issue
- Already resolved in the current revision
- False positive or misunderstanding
- Partially addressed
- Out of scope, with rationale

Check the current branch, commit history, and file contents before deciding. Review comments often refer to an earlier revision.

### 2. Explain the Problem

Describe the underlying behavior and consequence in plain language. Identify the violated contract or invariant, and state whether the problem is reproducible or evidenced by inspection.

### 3. Choose the Action

- For an active issue, make the smallest focused fix and add or update a regression test.
- For an already-resolved issue, reply with the current revision or commit that addresses it.
- For a false positive, explain the architectural guarantee or evidence that makes the concern inapplicable.
- For an out-of-scope issue, record it separately rather than quietly expanding the change.

After editing, rerun the focused validation before moving to another review comment.

## Review Report Format

```markdown
## Review Summary

### Blockers
- **[path:location] Title**
  Impact, evidence, and required fix.

### Warnings
- **[path:location] Title**
  Risk, evidence, and recommended fix.

### Nits
- **[path:location] Title**
  Optional improvement.

### Validation
- Commands run and results
- Remaining test gaps or environmental limitations

### Verdict
APPROVED or CHANGES REQUESTED
```

If a review finds no issues, say so clearly and still mention meaningful residual risk or untested areas.

## Knowledge Capture

When a review catches a recurring defect pattern, decide whether it belongs in a reusable engineering guideline, repository instructions, or a regression test. Capture the rule at the narrowest level that will prevent the same mistake elsewhere.
