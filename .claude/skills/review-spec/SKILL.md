---
name: review-spec
description: Review an implementation against its design spec for gaps, missing features, and improvements. Usage - /review-spec <plan-name> (matches files in docs/superpowers/plans/)
user-invocable: true
---

# Review Implementation Against Spec

You are reviewing an implementation to verify it matches its design spec. Find gaps, missing features, spec violations, and improvements.

## 1. Resolve the Plan

The user provided a plan name or partial name as arguments: `$ARGUMENTS`

Find the matching plan file in `docs/superpowers/plans/`. If the argument is:

- A full filename: use it directly
- A partial match: glob for `docs/superpowers/plans/*$ARGUMENTS*` and use the best match
- Empty: list available plans in `docs/superpowers/plans/` and ask the user to pick one

If no match is found, tell the user and list available plans.

## 2. Read the Plan and Spec

1. Read the full plan file
2. Find the `**Spec:**` line in the plan header — it points to the design spec
3. Read the full spec file
4. Note every file listed in the plan's **File Map** table — these are the implementation files to review

## 3. Read All Implementation Files

Read every file listed in the plan's File Map. These are the files that were created during plan execution.

## 4. Check Git Diff

Run `git diff` against the origin of the current branch to see all changes:

```bash
# Get the current branch and its merge base
git diff $(git merge-base HEAD origin/$(git branch --show-current) 2>/dev/null || echo HEAD~1)..HEAD --stat
```

If there's no remote or no prior commits, fall back to:

```bash
git log --oneline --all
git diff --stat
```

Review the diff output to understand the full scope of changes made.

## 5. Review Against Spec

Now systematically compare implementation against spec. For each section of the spec, check:

### Completeness

- Is every feature described in the spec implemented?
- Are all files from the File Map present and populated?
- Does the implementation cover all the spec's requirements?

### Correctness

- Does the implementation match what the spec describes?
- Are there deviations from the spec's design decisions?
- Do patterns match the spec's conventions?

### Gaps

- Are there spec requirements with no corresponding implementation?
- Are there edge cases the spec describes that aren't handled?
- Are there integration points described but not wired up?

### Quality & Improvements

- Are there things the spec doesn't cover that would make the implementation better?
- Are there inconsistencies between files?
- Could any patterns be improved while staying within the spec's intent?

## 6. Report

Present findings in this format:

### Spec Compliance Report

**Plan:** `<plan file path>`
**Spec:** `<spec file path>`
**Files reviewed:** N

#### Missing from Implementation

- [ ] List each spec requirement that is NOT implemented

#### Deviations from Spec

- [ ] List each place where implementation differs from spec

#### Improvements

- [ ] List suggestions that would make the implementation better

#### Passing

- [x] List spec requirements that ARE correctly implemented

If everything passes with no issues, say so clearly. If there are issues, prioritize them by severity.
