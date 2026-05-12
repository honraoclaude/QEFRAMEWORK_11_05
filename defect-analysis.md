# Workflow: Defect Analysis

Use when the user shares a Jira bug ticket or asks for defect analysis, root cause, or escape analysis.

## Step 1: Triage the report

Score the bug report itself before analyzing the bug. A poorly-written bug doubles cycle time.

**Bug report quality checklist:**
- [ ] One-line summary explains the impact, not just the symptom
- [ ] Environment specified (env, browser/OS, build/commit, user role)
- [ ] Preconditions stated
- [ ] Reproduction steps are numbered and unambiguous
- [ ] Actual vs Expected behavior are both stated
- [ ] Frequency stated (always / intermittent / once)
- [ ] Evidence attached (screenshots, video, logs, har file, request/response)
- [ ] First seen / last seen / build introduced
- [ ] Severity AND Priority set separately (not conflated)

If multiple items are missing, list them as "Report Quality Gaps" before analysis.

## Step 2: Severity vs Priority

These are different and often conflated. Always validate both:

**Severity** = technical impact if it happens
- **S1 Critical** — Data loss, security breach, total outage, no workaround
- **S2 High** — Core feature broken, painful workaround
- **S3 Medium** — Non-core feature broken or core feature with easy workaround
- **S4 Low** — Cosmetic, typo, minor inconvenience

**Priority** = business urgency to fix
- **P0** — Fix now, all hands
- **P1** — Fix this sprint
- **P2** — Fix soon, schedule
- **P3** — Backlog, fix if convenient

A typo on the legal page can be S4 / P0 (urgent for compliance). A rare crash can be S1 / P2 (severe but happens once a year to one user).

## Step 3: Root cause analysis (5 Whys)

Don't stop at the first cause. Drill down at least 5 levels.

Example:
1. Why did the payment fail? → API returned 500
2. Why did the API return 500? → Null pointer in payment service
3. Why was the value null? → User profile had no default currency
4. Why was there no default currency? → Signup flow skips currency for guest checkout
5. Why did payment service not handle null? → No validation on this code path; tests only covered logged-in users

**Real root cause:** Missing input validation + test gap on guest checkout, not "null pointer."

## Step 4: Categorize the root cause

Pick one primary category (and optionally a secondary):

| Category | Indicates a gap in |
|---|---|
| **Requirements** | Story missing AC, ambiguity, hidden assumption |
| **Design** | Architecture, contract, or data model flaw |
| **Code** | Logic error, off-by-one, null handling |
| **Test** | Scenario not covered by any test layer |
| **Test data** | Tests use unrealistic data; bug needs prod-like data |
| **Environment** | Config drift, infra mismatch, dependency version |
| **Integration** | Contract mismatch between services |
| **UX** | Confusing flow led user into invalid state |
| **Documentation** | User followed docs that were wrong/outdated |
| **Process** | Change slipped past review, no feature flag, no canary |

The category tells you **what to change**, not just **what to fix**.

## Step 5: Escape analysis

For every defect found in prod (or late stages), ask:

1. **Where in the SDLC was this defect introduced?** (requirements / design / code)
2. **Where could it have been caught earlier?** (refinement / code review / unit / API / E2E / UAT)
3. **Why wasn't it caught?** (no test / test was wrong / test was skipped / not in scope)
4. **What test do we add now?** (specific test case at the correct layer)
5. **Where does the test belong?** (which suite, who owns it)

Output the new test case in Given/When/Then so it's ready to implement.

## Step 6: Regression risk assessment

Before the fix ships:

- **Blast radius** — What other features use this code path?
- **Data risk** — Does the fix migrate or backfill data? Reversible?
- **Performance risk** — Does the fix add latency or DB load?
- **Rollback plan** — Feature flag? Canary? Can it be reverted in one click?
- **Targeted regression suite** — Which existing tests must pass before release?

List 3-10 specific regression test IDs to run, not "run full regression."

## Step 7: Prevention recommendations

The most valuable section. For each defect, suggest one shift-left change:

- "Add a Definition of Ready check: every story touching payment must specify guest-vs-logged-in behavior."
- "Add a unit test for null currency in payment service."
- "Add a contract test between Checkout and Payment services for the currency field."
- "Add a linting rule for missing null checks on external inputs."
- "Add an exploratory charter for guest checkout flows each sprint."

Prevention is the deliverable that distinguishes quality engineering from QA.
